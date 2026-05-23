# ForgeOne Code Review Roadmap

> Review date: 2026-05-01
> Pipeline: Auth -> Session -> run_to_breakpoint -> Worker -> LangGraph -> State/Artifacts/Export -> Frontend WS/Polling

---

## CRITICAL (Must fix immediately)

### C1. CORS Wildcard + Credentials

**File:** `backend/app/main.py:96-102`

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    ...
)
```

`allow_origins=["*"]` + `allow_credentials=True` = any origin can make authenticated cross-origin requests.

**Fix:** Replace `["*"]` with explicit frontend origin allowlist from config. Separate dev/prod values.

---

### C2. No Rate Limiting on Auth Endpoints

**File:** `backend/app/main.py:285-308`

`/api/auth/login` and `/api/auth/register` have zero rate limiting. Unlimited brute-force / mass registration.

**Fix:** Add `slowapi` or `fastapi-limiter`. Login: 5 attempts/email/minute. Register: 10/IP/hour.

---

### C3. Published Pages Serve Unsanitized LLM-Generated HTML

**File:** `backend/app/main.py:696-702`

LLM-generated HTML returned with `media_type="text/html"` and no sanitization. An LLM can inject `<script>`, `<iframe>`, etc. Since pages are served from the same domain, XSS has full access to tokens and API.

**Fix:** Sanitize with `bleach` or `nh3` before storage. Or serve from a sandbox subdomain.

---

### C4. Default Admin Credentials admin/admin

**File:** `backend/app/config.py:18-21`

```python
DEFAULT_ADMIN_EMAIL = "admin"
DEFAULT_ADMIN_PASSWORD = "admin"
```

Non-production environments auto-enable bootstrap. Any staging/testing server is wide open.

**Fix:** Remove default credentials. Require explicit env config. Extend startup validator to warn in all non-CI environments.

---

### C5. No Job Crash Recovery

**File:** `backend/app/task_queue.py:67-70`

Process crash leaves jobs in `status="running"` forever. `has_active_execution_job` then blocks rollback, session becomes stuck.

**Fix:** Add startup sweep: transition all `status="running"` jobs from previous process to `"failed"` or `"queued"`. Add `max_attempts` column with exponential backoff.

---

### C6. state_payload Unbounded Growth

**File:** `backend/app/repository.py:844-854`

JSON column has no size limit. Deep research rounds, execution traces, and control plane audit logs are continuously appended. Full `copy.deepcopy` on every update. Single session can reach MBs.

**Fix:** Add size guard in `update_state_payload`. Extract hot-path data (execution_trace, deep_research rounds, control_plane) into separate tables or object storage references.

---

### C7. SQL Injection Risk via f-string Interpolation

**File:** `backend/app/repository.py:346`

```python
f"PRAGMA table_info({table_name})"
f"ALTER TABLE {table_name} ADD COLUMN {ddl}"
```

Currently called with hardcoded table names, but the inner function accepts any string. Future caller passing user input = direct SQL injection.

**Fix:** Add table name allowlist validation inside `table_columns` and `add_column_if_missing`.

---

## HIGH (Must fix before release)

### Security

#### H1. JWT TTL 12h, No Refresh/Revoke/Logout

**File:** `backend/app/config.py:189` (`43200` seconds = 12 hours)

No token revocation, no blacklist, no refresh mechanism. Leaked token = 12h unrestricted access.

**Fix:** Reduce to 15min access token + refresh token rotation. Add JTI claim + blacklist table for logout.

---

#### H2. Token Stored in localStorage

**File:** `frontend/lib/auth.ts:56-60`

XSS (including via C3) can steal token with one line: `localStorage.getItem("forgeone.auth.session")`.

**Fix:** Move to httpOnly, Secure, SameSite cookies. If SPA requires localStorage, reduce TTL drastically (see H1).

---

#### H3. Webhook Endpoint Unauthenticated + Stores Full Headers

**File:** `backend/app/main.py:631-643`

No auth, no rate limit. Full request headers (Cookie, Authorization) are persisted into state_payload.

**Fix:** Filter headers to only those needed for signature verification. Add rate limiting on public endpoints.

---

#### H4. No Security Headers

Missing globally: `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`, `Content-Security-Policy`, `X-XSS-Protection`, `Referrer-Policy`.

**Fix:** Add security headers middleware.

---

#### H5. No Max Password Length -> PBKDF2 DoS

**File:** `backend/app/schemas.py:230` (`min_length=8`, no `max_length`)

Multi-MB password string consumes CPU on PBKDF2 310k iterations.

**Fix:** Add `max_length=128`.

---

### Concurrency / Architecture

#### H6. Lock Dictionaries Grow Without Bound

**Files:** `backend/app/langgraph_runtime.py:956` (`_locks`), `backend/app/repository.py:294` (`_state_payload_locks`)

`defaultdict(asyncio.Lock)` / `defaultdict(threading.Lock)` creates a lock per session_id, never cleaned up.

**Fix:** Use bounded cache (`lru_cache` with max size) or clean up locks on session completion.

---

#### H7. append_jsonl Non-Atomic Read-Modify-Write

**File:** `backend/app/object_storage.py:44-53`

Reads entire file, appends line, writes whole file back. O(n) per append + concurrent writes overwrite each other.

**Fix:** Use `file.open("a")` for atomic append. Read separately when needed.

---

#### H8. _call_async + .result() Potential Deadlock

**File:** `backend/app/langgraph_runtime.py:3082-3083`

`asyncio.run_coroutine_threadsafe(...).result()` blocks thread. If the coroutine needs the event loop and the loop is blocked waiting for thread pool, deadlock.

**Fix:** Add timeout (`.result(timeout=30)`). Document threading model explicitly.

---

#### H9. Single-Process Architecture Ceiling

Entire system (web server, task worker, LangGraph runtime) runs in one uvicorn process. One worker, one thread pool. Cannot scale horizontally.

**Fix:** Document single-process constraint for SQLite mode. PostgreSQL path supports multi-worker via `SKIP LOCKED`. Long-term: externalize queue (Redis/RabbitMQ), WS fan-out (Redis pub/sub), object storage (S3).

---

#### H10. WebSocket Stale-Read Race Condition

**File:** `frontend/components/session-dashboard.tsx:293-302`

WS message triggers `getSession()` separately. Multiple in-flight requests can resolve out of order, overwriting fresh state with stale data.

**Fix:** Include full snapshot in WS message, or add monotonic version to session and only apply higher-version updates, or serialize getSession calls.

---

### Code Quality

#### H11. Oversized Core Files

| File | Lines | Guideline |
|------|-------|-----------|
| `backend/app/langgraph_runtime.py` | 3180 | 800 max |
| `backend/app/repository.py` | 2986 | 800 max |
| `frontend/components/session-result-page-client.tsx` | 1793 | 800 max |
| `frontend/components/session-dashboard.tsx` | 1295 | 800 max |

**Fix:** Split into focused modules (see Recommendations section below).

---

#### H12. Functions Exceeding 50 Lines

| Function | Lines | File |
|----------|-------|------|
| `_run_roundtable_critic` | ~175 | `langgraph_runtime.py` |
| `_run_deep_research_evidence_critic` | ~145 | `langgraph_runtime.py` |
| `_run_deep_research_search` | ~135 | `langgraph_runtime.py` |
| `_run_deep_research_extract` | ~120 | `langgraph_runtime.py` |
| `_build_snapshot` | ~125 | `repository.py` |
| `_create_session` | ~110 | `repository.py` |
| `process_external_dispatch` | ~100 | `repository.py` |

**Fix:** Extract sub-functions with clear names and single responsibility.

---

#### H13. Pervasive `Any` Type Usage

**File:** `backend/app/langgraph_runtime.py:852, 898, 1189, 1230` etc.

Many methods accept `session: Any` instead of `SessionView`. Typos like `session.state_paylod` not caught at lint time.

**Fix:** Replace `Any` with `SessionView` or define a `Protocol`.

---

#### H14. Zero Database Indexes

**File:** `backend/app/db_models.py`

High-frequency query columns have no indexes:

| Column | Query Context |
|--------|---------------|
| `execution_jobs.status` | Polled every 200ms |
| `execution_jobs.session_id` | Active job checks |
| `sessions.user_id` | List user sessions |
| `sessions.tenant_id` | Tenant scoping |
| `sessions.status` | Admin session lists |
| `decision_cards.session_id + status` | Pending decision lookup |
| `agent_runs.session_id` | Session view loading |
| `cost_ledgers.session_id` | Budget aggregation |

**Fix:** Add `Index()` definitions via `__table_args__`. High-impact, low-risk change.

---

#### H15. N+1 Query in count_execution_jobs_by_status

**File:** `backend/app/repository.py:2273-2284`

Loads full ORM objects into memory just to count them.

**Fix:** Use `session.scalar(select(func.count()).where(...))`.

---

#### H16. setInterval Polling Without Concurrency Guard

**File:** `frontend/components/session-dashboard.tsx:334-345`

If `getSession` takes >2s, next interval fires before previous completes. Overlapping requests cause state thrashing.

**Fix:** Switch to recursive `setTimeout` where next poll starts only after previous finishes.

---

#### H17. WebSocket + Polling Run Simultaneously

**File:** `frontend/components/session-dashboard.tsx:266-357`

Both pathways call `getSession` + `setSession`. Polling overwrites WS-triggered state and vice versa.

**Fix:** Pause polling when WS is connected. Resume on WS disconnect.

---

### Database

#### H18. No Alembic Migration Files

`backend/alembic/versions/` is empty. PostgreSQL path (`alembic upgrade head`) will fail.

**Fix:** Generate migration files from current models.

---

#### H19. Float for Monetary Values

`cost_so_far`, `budget_cap`, `cost_estimate`, etc. all use `Float`. Floating-point rounding errors for money.

**Fix:** Use `Numeric(10, 2)` (DECIMAL) for PostgreSQL path.

---

#### H20. datetime.utcnow() Deprecated + Inconsistent

Used throughout `repository.py` and `langgraph_runtime.py`, while `db_models.py` uses `datetime.now(timezone.utc)`. Mixed naive/aware datetimes.

**Fix:** Standardize on `datetime.now(timezone.utc)` everywhere.

---

## MEDIUM (Should fix)

### Security

| ID | Issue | File | Line |
|----|-------|------|------|
| M1 | Admin bootstrap overwrites password every startup | `repository.py` | 516-523 |
| M2 | User enumeration via registration error message | `main.py` | 297 |
| M3 | Login KeyError leaks email in exception message | `repository.py` | 539 |
| M4 | No CSRF protection on public form endpoints | `main.py` | 604-693 |
| M5 | WS auth only on connect, no re-auth | `main.py` | 851-879 |
| M6 | JWT missing `iss`/`aud` claims | `auth.py` | 44-52 |
| M7 | Token via query param leaks into logs | `main.py` | 124 |
| M8 | Default webhook secret `"mock-signature"` | `config.py` | 21 |

### Frontend

| ID | Issue | File |
|----|-------|------|
| M9 | `as Record<string, unknown>` pervasive, untyped payload access | `session-dashboard.tsx:165` |
| M10 | `scenario_type` cast without validation | `create-session-form.tsx:91` |
| M11 | `runtimeSignature` JSON.stringify in render, not `useMemo` | `workflow-editor.tsx:380-388` |
| M12 | `onNodesChange`/`onEdgesChange` recreated every render, missing `useCallback` | `workflow-editor.tsx:455-475` |
| M13 | `deepResearchProviderCost` computed every render without memo | `session-dashboard.tsx:376` |
| M14 | `buildDecisionBrief` ~80 lines runs every render, no `useMemo` | `session-result-page-client.tsx:1197` |
| M15 | Event list key uses index | `session-dashboard.tsx:701` |
| M16 | `MVP_DIMENSIONS` array recreated every render, defeats `useMemo` | `deep-research-entry.tsx:40-47` |
| M17 | `downloadExportFile` revokes URL before browser finishes download | `api.ts:777-787` |
| M18 | No list virtualization for execution steps | `session-dashboard.tsx:608` |

### Accessibility

| ID | Issue | File |
|----|-------|------|
| M19 | Form inputs lack explicit `id`/`htmlFor` | `auth-gate.tsx:147-155` |
| M20 | Tab nav missing `role="tablist"`/`role="tab"` | `session-dashboard.tsx:1268-1281` |
| M21 | Status indicators rely on color only | `session-dashboard.tsx:870-877` |

### Backend Patterns

| ID | Issue | File |
|----|-------|------|
| M22 | String status values instead of Enum | `repository.py`, `langgraph_runtime.py`, `orchestrator.py` |
| M23 | Deprecated `on_event("startup"/"shutdown")` | `main.py:221, 251` |
| M24 | Module-level singletons at import time | `main.py:75-93` |
| M25 | Metrics middleware high-cardinality paths (includes session_id) | `main.py:214-217` |
| M26 | PDF hardcoded Chinese font STSong-Light, no fallback | `export_renderer.py:619-633` |
| M27 | HTML export via f-strings instead of template engine | `export_renderer.py:19-186` |
| M28 | `build_task_graph` returns untyped `dict[str, Any]` | `task_graph.py:179-229` |
| M29 | `_trace_step` has 18 parameters | `langgraph_runtime.py:972-1022` |
| M30 | Mutable default `default=dict` in SQLAlchemy columns | `db_models.py:59, 87, 104, 126` |

### WebSocket

| ID | Issue | File |
|----|-------|------|
| M31 | No heartbeat/ping mechanism | `websocket_manager.py` |
| M32 | Server discards all client messages silently | `main.py:877` |

### Caching

| ID | Issue | File |
|----|-------|------|
| M33 | 1.5s TTL cache staleness window after writes | `repository.py:715-729` |
| M34 | Redundant `deepcopy` calls in tight loops | `repository.py` throughout |

---

## Positive Observations

- PBKDF2-SHA256 with 310k iterations + `hmac.compare_digest` (timing-safe)
- Pydantic validation on all endpoints
- SQLAlchemy ORM parameterized queries on main paths
- Production startup validation enforces strong secrets
- Webhook signature uses `compare_digest`
- Frontend `tsc --noEmit` zero errors with `strict: true`
- Proper WebSocket close code handling (4401/4403/4404)
- Consistent `active` flag pattern in React `useEffect` cleanup
- `revokeObjectURL` in `finally` block for download cleanup

---

## Priority Remediation Roadmap

### Phase 1: Immediate (Block release)

- [ ] C1: Fix CORS to explicit origin allowlist
- [ ] C2: Add rate limiting on auth endpoints
- [ ] C3: Sanitize LLM-generated HTML (bleach/nh3)
- [ ] C4: Remove default admin credentials
- [ ] C5: Add job crash recovery startup sweep
- [ ] C7: Add table name allowlist in SQLite compat functions

### Phase 2: Short-term (Next sprint)

- [ ] H1: Reduce JWT TTL + implement refresh tokens
- [ ] H14: Add database indexes on hottest query paths
- [ ] H16: Frontend setInterval -> recursive setTimeout
- [ ] H17: Pause polling when WS connected
- [ ] H18: Generate Alembic migration files
- [ ] H20: Replace datetime.utcnow() globally
- [ ] H5: Add max password length
- [ ] H4: Add security headers middleware
- [ ] C6: Add state_payload size guard

### Phase 3: Medium-term (Next quarter)

- [ ] H11: Split langgraph_runtime.py into modules:
  - `langgraph_nodes.py` (node implementations)
  - `langgraph_budget.py` (budget guard)
  - `langgraph_artifacts.py` (artifact blueprints)
  - `langgraph_research.py` (deep research)
  - `langgraph_runtime.py` (graph construction)
- [ ] H11: Split repository.py into:
  - `session_repository.py`
  - `execution_job_repository.py`
  - `export_repository.py`
  - `snapshot_repository.py`
- [ ] H11: Split frontend large components:
  - `session-result-page-client.tsx` -> per-artifact sub-components
  - `session-dashboard.tsx` -> per-tab components
- [ ] H10: Add monotonic version to session for stale-read prevention
- [ ] H6: Bounded lock cache (evict on session completion)
- [ ] H19: Monetary columns -> Numeric(10,2)
- [ ] M22: Status strings -> Enum
- [ ] M31: WebSocket heartbeat

### Phase 4: Long-term (Scaling)

- [ ] H9: Externalize task queue (PostgreSQL SKIP LOCKED / Redis)
- [ ] H9: Redis pub/sub for WebSocket fan-out
- [ ] H9: S3 object storage backend
- [ ] Evaluate LangGraph `ainvoke` vs `asyncio.to_thread`
- [ ] Export rendering async with completion notification

---

## Severity Summary

| Severity | Count | Top Areas |
|----------|-------|-----------|
| CRITICAL | 7 | CORS, rate limiting, XSS, default creds, crash recovery, unbounded state, SQL injection |
| HIGH | 21 | JWT design, indexes, concurrency races, file sizes, type safety, frontend state management |
| MEDIUM | 34 | Enum usage, accessibility, caching, WebSocket hardening, code organization |
