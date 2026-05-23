# ForgeOne Instruction

## 1. 文档用途

本文件用于后续会话快速建立 ForgeOne 当前状态、可用启动方式、基础使用路径、mock 实验路径和已知上线缺口。

## 2. 当前现状

### 2.1 产品定位

ForgeOne 当前定位是 `one-stop validation system`。
核心不是通用聊天，而是把 `idea -> 拆解 -> 多 Agent 执行 -> 决策门 -> artifact -> 外部执行模拟` 串成可控流程。

### 2.2 当前已具备

- 用户注册、登录、Bearer Token 鉴权
- 自动创建个人租户，多租户数据隔离
- 会话创建、异步任务队列、WebSocket 实时刷新
- 基于 idea 的场景分类与预算建议
- 动态任务图与动态决策门
- 多 Agent 并行执行
- artifact 导出：`HTML`、`PDF`、`JSON bundle`
- workflow 可视化预览和租户级 workflow definition 编辑
- RAG 知识库、审核工作台、原子动作知识层
- token / 风险 / 恢复面板
- 落地页发布
- `form / waitlist / preorder / payment` 的 mock 外部执行链路

### 2.3 当前不是生产可上线状态

以下能力仍未达到生产门槛：

- 外部执行器仍以 mock provider 为主，不是真实第三方接入
- webhook 安全、支付状态机、对账退款、幂等、限流、滥用防护未补齐
- 发布页真实域名、CDN、回滚治理未补齐
- 默认开发配置仍可启动，生产 fail-fast 校验未补齐
- 监控、审计、E2E、灾备演练未补齐

上线缺口统一以 [Roadmap.md](./Roadmap.md) 为准，后续必须优先参考该文档。

## 3. 已验证结果

当前仓库已验证：

- 后端 API 测试：`31 passed`
- 前端生产构建：`next build` 通过

当前明确存在的 warning：

- FastAPI `@app.on_event("startup") / @app.on_event("shutdown")` 已弃用
- 需要替换为 `lifespan`

## 4. 一键启动

仓库根目录执行：

```powershell
.\install-deps.bat
.\start.bat
```

停止：

```powershell
.\stop.bat
```

首次迁移到新电脑时，先运行 `.\install-deps.bat`。

### 4.1 `install-deps.bat` 当前行为

`install-deps.bat` 会调用 `scripts/forgeone-install-deps.ps1`，完成以下动作：

- 自动检测可用 Python
- 如果是 `py.exe -3.11` 或用户 Python，则把后端依赖安装到当前用户 Python 环境
- 如果只能回退到 Blender Python，则重建 `.pydeps`
- 执行 `frontend\npm install`

### 4.2 `start.bat` 当前行为

`start.bat` 会调用 [`scripts/forgeone-start.ps1`](./scripts/forgeone-start.ps1)，完成以下动作：

- 检查当前可用 Python
- 如果回退到 Blender Python，检查 `.pydeps`
- 检查 `frontend\node_modules`
- 优先使用 `py.exe -3.11`
- 如果 `py.exe -3.11` 不可用，再回退到兼容的 Blender Python `>= 3.10`
- 使用 `User Python / py.exe -3.11` 时，不再注入 `.pydeps`
- 只有回退到 Blender Python 时，才依赖 `.pydeps`
- 自动选择可用后端端口，默认从 `8010` 开始
- 自动选择可用前端端口，默认从 `3000` 开始
- 自动设置 `FORGEONE_LLM_MOCK_MODE=false`
- 启动后端和前端
- 默认打开浏览器

运行状态文件：

- `.runtime\backend.log`
- `.runtime\frontend.log`
- `.runtime\services.json`

### 4.3 默认访问地址

实际端口以 `.runtime\services.json` 为准。

- 前端：`http://localhost:3000` 起始自动选端口
- 后端：`http://localhost:8010` 起始自动选端口

## 5. 手动启动

### 5.1 后端

```powershell
$env:PYTHONPATH='F:\Code\Discuss\ForgeOne\.pydeps;F:\Code\Discuss\ForgeOne'
$env:FORGEONE_LLM_MOCK_MODE='false'
C:\WINDOWS\py.exe -3.11 -m uvicorn backend.app.main:app --host 127.0.0.1 --port 8000
```

健康检查：

```powershell
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:8000/api/health
```

### 5.2 前端

```powershell
cd frontend
$env:NEXT_PUBLIC_API_BASE_URL='http://127.0.0.1:8000'
npm.cmd run dev
```

## 6. 关键环境变量

后端常用：

- `FORGEONE_DATABASE_URL`
- `FORGEONE_LLM_BASE_URL`
- `FORGEONE_LLM_API_KEY`
- `FORGEONE_LLM_MODEL`
- `FORGEONE_LLM_MOCK_MODE`
- `FORGEONE_OBJECT_STORAGE_ROOT`
- `FORGEONE_AUTH_SECRET_KEY`
- `FORGEONE_AUTH_ACCESS_TOKEN_TTL_SECONDS`
- `FORGEONE_ADMIN_BOOTSTRAP_ENABLED`
- `FORGEONE_ADMIN_EMAIL`
- `FORGEONE_ADMIN_PASSWORD`

前端常用：

- `NEXT_PUBLIC_API_BASE_URL`

当前默认行为：

- 默认数据库：`data/forgeone.db`
- 默认对象存储：`data/object-store`
- 默认真实 LLM：通过 `start.bat` 启动时启用；只有显式设置 `FORGEONE_LLM_MOCK_MODE=true` 才进入 mock
- 默认 admin：`admin / admin`

说明：

- 不要再用 Blender 2.93 自带 Python 3.9 启动后端
- 当前后端代码已经要求 Python 3.10+

## 7. 正常使用路径

### 7.1 UI 使用路径

1. 执行 `start.bat`
2. 打开前端首页
3. 注册普通账号，或使用 `admin / admin` 登录
4. 在首页输入 `idea`
5. 可选填写 `user_constraints`
6. 可直接接受系统预算建议，或展开高级设置手动覆盖
7. 提交后跳转到 `/sessions/{id}`
8. 在会话页处理决策卡
9. 会话完成后查看 artifact、导出文件和外部执行摘要

### 7.2 创建会话的最小输入

只输入 `idea` 即可。

系统会自动：

- 推断 `scenario_type`
- 给出预算建议
- 建立动态任务图

### 7.3 当前支持的场景

- `digital_product`
- `content_business`
- `presale_project`

## 8. 核心 API

### 8.1 认证

- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/me`

认证方式：

```text
Authorization: Bearer <token>
```

### 8.2 会话

- `POST /api/sessions`
- `GET /api/sessions/{session_id}`
- `POST /api/sessions/{session_id}/control`
- `POST /api/decisions/{decision_id}/submit`
- `GET /api/sessions/{session_id}/workflow-preview`

### 8.3 调试与导出

- `GET /api/health`
- `GET /api/metrics`
- `GET /api/exports/{export_id}/download`

### 8.4 公开 mock 工具入口

- `POST /api/public/forms/{session_id}/{form_id}/submit`
- `POST /api/public/waitlists/{session_id}/{tool_id}/join`
- `POST /api/public/preorders/{session_id}/{tool_id}/reserve`
- `POST /api/public/payments/{session_id}/{tool_id}/checkout`
- `GET /p/{page_id}`

## 9. mock 流程怎么实验功能

### 9.1 实验目标

mock 流程不是接真实支付、真实邮件、真实 CRM。
当前用途是验证：

- 会话是否能完成
- artifact 是否生成
- 发布页是否可访问
- 外部执行链路是否被写入会话状态
- `email / webhook / crm / analytics / payment` 摘要是否可被消费

### 9.2 最快实验路径：UI

建议用 `presale_project`，因为它覆盖最完整的 mock 外部执行链路。

推荐 idea：

```text
validate a hardware preorder with refundable deposit and waitlist conversion
```

推荐步骤：

1. `start.bat`
2. 登录
3. 创建 `presale_project` 会话
4. 连续处理决策卡，全部点 `approve`
5. 等会话状态变成 `completed`
6. 打开结果页，确认 artifact 中存在：
   - `external_execution_stack`
   - `waitlist_page` / `presale_page` 相关内容
   - `go_no_go_validation_report`
7. 如果存在发布页入口，访问 `/p/{page_id}` 检查静态发布结果

### 9.3 API mock 实验路径：从创建到完成

#### 1. 登录

```powershell
$login = Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/auth/login" -ContentType "application/json" -Body (@{
  email = "admin"
  password = "admin"
} | ConvertTo-Json)
$token = $login.access_token
$headers = @{ Authorization = "Bearer $token" }
```

#### 2. 创建会话

```powershell
$session = Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/sessions" -Headers $headers -ContentType "application/json" -Body (@{
  idea = "validate a hardware preorder with refundable deposit and waitlist conversion"
  scenario_type = "presale_project"
  budget_cap = 5
  user_constraints = "keep the first batch refundable and limited"
} | ConvertTo-Json)
$sessionId = $session.id
```

#### 3. 轮询会话

```powershell
$snapshot = Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/api/sessions/$sessionId" -Headers $headers
```

#### 4. 处理决策卡

当 `status=waiting_decision` 时，从 `decision_cards` 中找 `status=pending` 的卡，提交：

```powershell
$decisionId = "<pending_decision_id>"
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/decisions/$decisionId/submit" -Headers $headers -ContentType "application/json" -Body (@{
  action = "approve"
} | ConvertTo-Json)
```

重复直到会话 `status=completed`。

### 9.4 API mock 实验路径：外部执行链路

会话完成后，从 `artifacts` 中找到 `artifact_type = external_execution_stack`，读取其中的：

- `form.submit_path`
- `waitlist.submit_path`
- `preorder.reserve_path`
- `payment.checkout_path`

#### waitlist

```powershell
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/public/waitlists/<session_id>/<tool_id>/join" -ContentType "application/json" -Body (@{
  email = "waitlist@example.com"
  name = "Waitlist User"
  notes = "Interested in prototype milestones"
  metadata = @{ utm_source = "waitlist-demo" }
} | ConvertTo-Json -Depth 4)
```

#### preorder

```powershell
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/public/preorders/<session_id>/<tool_id>/reserve" -ContentType "application/json" -Body (@{
  email = "preorder@example.com"
  name = "Preorder User"
  quantity = 2
  reservation_tier = "Founding Batch"
  notes = "Need two units for pilot testing"
  metadata = @{ utm_source = "preorder-demo" }
} | ConvertTo-Json -Depth 4)
```

#### payment

```powershell
Invoke-RestMethod -Method Post -Uri "http://127.0.0.1:8000/api/public/payments/<session_id>/<tool_id>/checkout" -ContentType "application/json" -Body (@{
  email = "buyer@example.com"
  name = "Buyer User"
  quantity = 1
  amount = 99
  currency = "USD"
  notes = "Proceed with refundable deposit"
  metadata = @{ utm_source = "payment-demo" }
} | ConvertTo-Json -Depth 4)
```

### 9.5 mock 外部执行的观察点

再次调用：

```powershell
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/api/sessions/$sessionId" -Headers $headers
```

重点看：

- `state_payload.external_execution.summary`
- `state_payload.external_execution.waitlist_signups`
- `state_payload.external_execution.preorder_reservations`
- `state_payload.external_execution.payment_checkouts`
- `state_payload.external_execution.payment_charges`
- `state_payload.external_execution.email_outbox`
- `state_payload.external_execution.webhook_deliveries`
- `state_payload.external_execution.crm_contacts`
- `state_payload.external_execution.analytics_events`
- `execution_jobs` 中是否存在 `external_dispatch` 且 `status=completed`

### 9.6 mock 结果的真实含义

当前 mock 提交会产生：

- submission 记录
- mock email outbox 记录
- mock webhook delivery 记录
- mock CRM contact 记录
- mock analytics event 记录
- payment mock 下的 `checkout` 与 `authorized` 记录

当前不会产生：

- 真实扣款
- 真实邮件发送
- 真实 webhook 出站
- 真实 CRM upsert
- 真实 analytics 上报

## 10. 当前建议的实验组合

### 10.1 数字产品

目标：验证 idea 分类、预算建议、artifact 生成、导出。

推荐 idea：

```text
ship a Notion template bundle for freelance designers
```

### 10.2 内容业务

目标：验证 `content_business` 动态任务图和内容交付物。

推荐 idea：

```text
launch a paid newsletter with membership community and sponsorship slots
```

### 10.3 预售项目

目标：验证最完整 mock 外部链路。

推荐 idea：

```text
validate a hardware preorder with refundable deposit and waitlist conversion
```

## 11. 当前限制

- workflow 编辑器当前修改的是可视化定义和预览层，不直接改后端执行图代码
- prompt template 仍偏全局，不是完整租户级治理
- 真实 provider 未接通
- 当前 CORS 默认全开放，属于开发状态
- 默认 admin 和默认 auth secret 只适合本地实验
- 公开接口尚未具备生产级风控

## 12. 调试入口

### 12.1 健康检查

```powershell
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:8000/api/health
```

### 12.2 指标

```powershell
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/api/metrics" -Headers $headers
```

### 12.3 日志

- `.runtime\backend.log`
- `.runtime\frontend.log`

### 12.4 运行时元数据

- `.runtime\services.json`

### 12.5 对象存储

当前对象存储内容会写入：

```text
data/object-store/tenants/{tenant_id}/sessions/{session_id}/...
data/object-store/tenants/{tenant_id}/workflow-definitions/main.json
```

## 13. Python 测试执行方式

当前仓库推荐使用：

```powershell
C:\WINDOWS\py.exe -3.11 -m pytest backend/tests/test_api.py -q
```

说明：

- 沙箱内 `python` 可能不可见
- 沙箱外 `C:\WINDOWS\py.exe -3.11` 已验证可用

## 14. 后续会话约束

后续任何涉及上线评估、生产缺口、是否可上线、应补哪些功能的问题，优先参考：

1. [Roadmap.md](./Roadmap.md)
2. 本文件

如果两者冲突，以 `Roadmap.md` 中“未来上线必完成”为最高优先级。
