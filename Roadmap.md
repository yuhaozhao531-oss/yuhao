# ForgeOne Roadmap

更新时间：2026-04-29

## 定位

ForgeOne 是面向普通个人创业者和 0-3 人小团队的 `one-stop validation system`。

核心能力不是通用聊天，而是围绕数字产品、内容型业务、预售型项目提供可控执行、成本约束、风险暴露、人工确认和可交付验证产物。

## 当前状态

### 已具备

- [x] 后端基础架构：FastAPI、持久化、WebSocket、对象存储、任务队列 worker
- [x] 会话执行主链路：intake、classify、decompose、parallel plan、decision gates、artifact generation、complete
- [x] LangGraph 编排运行时接入
- [x] 动态任务图生成、节点依赖、分支条件、失败策略、恢复策略
- [x] 多 Agent 并行执行：DesignAgent、PackagingAgent、CostAgent、LaunchPageAgent、ResearchAgent、PricingAgent、ValidationAgent
- [x] 人工确认门：target audience、product packaging、launch page copy
- [x] 成本控制：总预算守卫、阶段预算守卫、Agent 限额守卫
- [x] 检查点、暂停、恢复、回滚、执行日志、决策记录、导出文件
- [x] 三类首发场景：digital product、content business、presale project
- [x] 结果导出：HTML、PDF、JSON bundle
- [x] 用户系统、管理员视图、多租户隔离
- [x] 基于商业原子动作的知识模型：找需求、定价、包装、内容生产、获客、转化、交付、复购
- [x] RAG 已按商业原子动作建模，支持 approved / legacy_unverified 区分
- [x] token 使用、风险、执行透明化、恢复面板
- [x] waitlist page、presale page、content asset pack、digital product starter bundle、go / no-go validation report
- [x] 落地页发布能力
- [x] 内部外部执行栈与 mock provider 闭环：form、waitlist、preorder、payment、email、webhook、CRM、analytics
- [x] 最小真实第三方闭环：published page -> form/waitlist -> email -> CRM -> analytics

### 部分具备

- [~] idea 到输出结果的主闭环已存在，但仍需补齐更多真实 provider 和生产门槛
- [~] waitlist、preorder、payment 已具备内部公共接口与 mock provider，真实第三方接入未完成
- [~] 外部执行器已有 adapter 接口，仍需补齐配置、密钥、幂等、重试、审计和运行治理

### 未具备

- [ ] 独立 DeepResearch 功能
- [ ] Tavily + Firecrawl DeepResearch provider 接入
- [ ] waitlist、preorder、payment 从 mock provider 切换到真实 provider
- [ ] 系统级风险聚合、风险阈值、风险拦截、风险恢复策略
- [ ] 订阅套餐、配额、用量计费、账单、发票
- [ ] 团队工作区、成员邀请、角色权限、项目级访问控制
- [ ] 跨项目复用、历史复盘、模板复用、最佳实践推荐
- [ ] 完整生产安全、观测、发布治理、灾难恢复能力

## 下一阶段优先级

### P0：ForgeOne = Agent Control Plane + Startup Validation Domain Engine

目标：将现有 `idea -> classify -> DeepResearch -> Critic -> decompose -> 多 Agent -> gate -> artifact/export` 从单条 execution path 升级为完整生产级系统：`ForgeOne = Agent Control Plane + Startup Validation Domain Engine`。Control Plane 负责 workflow、agent template、tool registry、policy、RBAC、secret、audit、observability、replay；Domain Engine 负责 DeepResearch、Evidence Binding、Decision Ledger、Artifact Contract、Trust Score、go/no-go。

#### Agent Control Plane

- [x] 移除 `backend/app/config.py` 中硬编码 LLM / Tavily / Firecrawl key，改为环境变量、secret store 或部署时注入
- [x] 启动阶段增加 secret / provider / environment 校验：缺失真实 provider key 时 fail-fast，开发环境允许显式 fake provider
- [x] 建立 Control Plane 数据模型：workflow、workflow_version、agent_template、tool_definition、policy_definition、approval_request、audit_event、replay_record
- [x] 建立 Governance Plane：RBAC、tenant scope、project/session scope、secret scope、tool permission、policy decision
- [x] 建立 Observability Plane：trace、structured log、metric、cost ledger、provider latency、node latency、failure taxonomy、replay record
- [x] 将主流程抽象为版本化 `workflow definition`：nodes、edges、conditions、tools、budgets、policies、artifacts、recovery_strategy
- [x] 建立 workflow definition schema 与 migration 策略，支持 session 绑定固定 workflow version
- [x] 建立 Tool Permission Model：每个 Agent 只能调用声明在 allowlist 中的工具
- [x] 工具调用必须绑定 tenant_id、session_id、agent_id、workflow_version、trace_id、budget_scope
- [x] 建立外部写入工具分级：read_only、draft、sandbox_write、live_write、payment、publish
- [x] 建立 Policy Gate：预算、风险、外部写入、发布、付款、邮件发送必须经过规则校验或人工确认
- [x] 建立 OPA-compatible policy 接口，首版可用本地 Python policy evaluator 实现
- [x] 建立 Human-in-the-loop approval record：approver、payload_hash、policy_result、decision、timestamp
- [x] 建立 Agent Template Versioning：agent role、prompt、tools、budget、timeout、retry、output_schema 均可版本化
- [x] 建立执行审计日志：workflow run、node run、agent run、tool call、policy decision、artifact mutation
- [x] 建立 provider governance：sandbox / live 模式、幂等键、重试策略、降级策略、错误分类、成本归因
- [x] 建立 production observability：structured logs、metrics、trace_id、node latency、provider latency、cost ledger、failure rate
- [x] 建立恢复与补救机制：failed node resume、manual retry、provider fallback、artifact rollback、policy unblock request
- [x] 建立 Eval / Replay：固定输入、固定 workflow version、固定 agent template version，可复现实验与回归比较
- [x] 建立 Quota / Billing 基础模型：tenant quota、provider cost、agent run cost、tool call cost、artifact storage cost
- [x] 前端流程图展示 workflow version、policy gate、tool permissions、approval status、audit trail
- [x] 管理端展示 control plane 对象：workflow version、agent template、tool registry、policy registry、approval queue、audit trail

#### Startup Validation Domain Engine

- [x] 建立 Domain Taxonomy：验证维度、信号类型、证据等级、来源等级、风险类型、决策类型、artifact 类型
- [x] Domain Taxonomy 覆盖首版维度：demand_signal、competitors、pricing、acquisition、compliance_cost_risk、delivery_feasibility、launch_readiness
- [x] 建立 Verified Evidence Binding：Agent 输出必须提供 `claim -> citation_id / assumption` 映射
- [x] 建立 Evidence Binding Gate：无 citation 的事实性结论自动降级为 assumption，assumption 不允许支撑 go/no-go
- [x] 关键 go/no-go 结论必须绑定 citation；缺 citation 时阻断并进入 `decision_blockers`
- [x] 建立 DeepResearch Evidence Quality Threshold：默认 5 个验证维度，每个维度至少 3 个独立来源，关键结论至少 2 个 citation
- [x] DeepResearch 停止条件改为 evidence sufficiency driven：覆盖率、citation 数、来源独立性、freshness、冲突状态达标后停止，不固定轮数
- [x] 高风险主题启用 expanded research：合规、付款、平台政策、医疗/金融/法律、强竞争市场默认扩展到 30+ 有效来源
- [x] assumption 只能进入 `next_validation_actions`，作为待验证假设和后续实验输入
- [x] 建立 Artifact Contract：每种 artifact 定义必填字段、evidence binding 规则、blocked 状态、导出字段和展示字段
- [x] 建立 Decision Ledger：所有 go/no-go、pricing、channel、risk、launch readiness 判断单独落账，绑定 citation / assumption / decision_blocker
- [x] 建立 Trust Score：按 citation 覆盖率、来源权重、freshness、来源独立性、冲突数量、unsupported_claim 数计算结果可信度
- [x] Agent artifact schema 增加 `evidence_bindings`、`assumptions`、`unsupported_claims`、`decision_blockers`、`assumption_to_test`
- [x] export / result page 展示每个关键判断的 citation、assumption、unsupported_claim、assumption_to_test 状态

#### P0 验收标准

- [x] Control Plane 验收：任何 Agent/tool 外部动作都必须经过权限、策略、审批、审计、成本归因、失败恢复路径
- [x] Domain Engine 验收：任何关键创业判断必须绑定 citation；assumption 只能进入待验证行动，不能支撑 go/no-go
- [x] 测试覆盖：schema、policy、tool permission、evidence binding、artifact contract、decision ledger、trust score、audit、resume、approval、eval/replay

### P0：DeepResearch MVP

- [x] 新增独立 DeepResearch 功能入口
- [x] 建立 DeepResearch 工作流：Planner -> Search -> Extract -> Critic -> Synthesize
- [x] 将 DeepResearch 接入主验证流程：idea -> classify -> DeepResearch -> Critic -> decompose -> parallel agent execution -> decision gates -> artifacts -> result
- [x] 新增 runtime 节点：`deep_research_plan`、`deep_research_search`、`deep_research_extract`、`deep_research_evidence_critic`、`deep_research_synthesize`
- [x] 在 LangGraph 中将执行顺序调整为 `intake -> classify -> deep_research_plan/search/extract/evidence_critic/synthesize -> decompose`
- [x] 更新 `NODE_COSTS`、`NODE_PROGRESS`、`STAGE_NODES`、`execution_order`，纳入 DeepResearch 节点
- [x] 更新 workflow preview / 前端流程图，展示 `deep_research_*` 节点
- [x] DeepResearch runtime 状态写入 `state_payload.deep_research`
- [x] `state_payload.deep_research` 包含 `plan`、`search_rounds`、`facts`、`evidence_table`、`critic_verdict`、`open_questions`、`citations`、`cost_snapshot`
- [x] 拆分 DeepResearch provider 接口：`SearchProvider` 与 `ExtractionProvider`
- [x] 新增 Tavily `SearchProvider` adapter：`search(query, max_results, topic, search_depth, include_answer, include_raw_content)`
- [x] 新增 Firecrawl `ExtractionProvider` adapter：`extract(urls, formats, only_main_content, timeout, schema)`
- [x] Tavily 配置项：`FORGEONE_SEARCH_PROVIDER=tavily`、`FORGEONE_TAVILY_API_KEY`、`FORGEONE_TAVILY_MAX_RESULTS`、`FORGEONE_TAVILY_TIMEOUT_SECONDS`
- [x] Firecrawl 配置项：`FORGEONE_EXTRACTION_PROVIDER=firecrawl`、`FORGEONE_FIRECRAWL_API_KEY`、`FORGEONE_FIRECRAWL_MAX_URLS`、`FORGEONE_FIRECRAWL_TIMEOUT_SECONDS`
- [x] 本地 API key 填写位置已标注：`backend/app/config.py` 的 `DeepResearch providers` 配置块；真实 key 可用同名环境变量覆盖
- [x] 启动阶段校验 Tavily / Firecrawl 配置；真实 provider 模式缺少 API key 时 fail-fast
- [x] Tavily 负责 source discovery：query、URL、snippet、初步 relevance、source metadata
- [x] Firecrawl 负责 source extraction：markdown、main content、metadata、links、price table、FAQ、structured scrape
- [x] Evidence Critic 决定 Firecrawl 深挖 URL：默认 Top 3-5，或 snippet 证据不足、pricing/competitor/compliance 维度触发
- [x] Tavily / Firecrawl 调用必须支持 timeout、错误记录、预算扣减、trace_id、原始响应摘要落盘
- [x] 首版人工验收使用真实 Tavily + Firecrawl API；不使用 mock 作为人工验收路径
- [x] 保留 fake search/extraction provider 仅用于自动化回归和无网络 CI
- [x] DeepResearch/Tavily/Firecrawl 成本写入现有 cost ledger 和 token / cost 面板
- [x] 搜索中断后支持从最近 search round 恢复；抽取中断后支持从最近 extraction round 恢复，避免整条 session 重跑
- [x] 建立 DeepResearch 数据保留规则：raw content、markdown、URL、引用文本、citation ledger 分级保存
- [x] 引用合规：报告只保存短摘录、结构化事实和 URL，不长期保存完整网页正文或完整 markdown
- [x] 新增 DeepResearch Critic 节点，输出 `sufficient / insufficient / conflicted / stale`
- [x] Critic 判定 `insufficient / conflicted / stale` 时触发补充搜索循环，达到边界后输出 open questions
- [x] Evidence Critic 输入为 `facts + evidence_table + citations + search_rounds`
- [x] Evidence Critic 输出结构：`verdict`、`confidence`、`insufficient_dimensions`、`conflict_groups`、`stale_sources`、`required_followup_search`、`reasoning_summary`
- [x] `decompose` 阶段读取 DeepResearch 的 evidence table 和 critic verdict 生成任务图
- [x] 为 `decompose` 提供读取接口：`get_deep_research_context(session.state_payload)`
- [x] `get_deep_research_context` 返回 `validated_facts`、`risk_flags`、`market_signals`、`pricing_signals`、`channel_signals`、`compliance_flags`、`open_questions`
- [x] `PricingAgent / ValidationAgent / LaunchPageAgent` 复用 DeepResearch 证据
- [x] 接入 Tavily + Firecrawl 组合 provider
- [x] 首版覆盖 5 个验证维度：需求信号、竞品、定价、获客、合规/成本风险
- [x] 输出 `deep_research_report` artifact
- [x] 注册 artifact 类型：`deep_research_report`
- [x] `deep_research_report.payload` 包含 `executive_summary`、`decision`、`dimensions`、`search_rounds`、`extraction_rounds`、`evidence_table`、`citations`、`critic_verdict`、`open_questions`、`next_validation_actions`
- [x] 更新 delivery template，将 `deep_research_report` 加入各首发场景产物顺序
- [x] 更新 export renderer，支持 `deep_research_report` 的 HTML / PDF / JSON bundle 输出
- [x] 更新结果页，展示 DeepResearch summary、evidence table、citations、critic verdict
- [x] WebSocket 发送 DeepResearch 进度事件：plan/search/extract/critic/synthesize
- [x] 租户隔离：DeepResearch 状态、artifact、搜索日志、citation ledger 必须写入当前 tenant/session 路径
- [x] 报告包含 go / no-go、evidence table、citations、open questions、next validation actions
- [x] 每条关键事实保留 source URL、引用编号、confidence、freshness、conflict status、extraction status
- [x] 设置循环与成本边界：max_depth、max_queries_per_round、max_sources_per_query、max_extracted_urls_per_round、预算上限、超时上限
- [x] 建立 DeepResearch 单元测试与集成测试覆盖（2026-04-28）

### P0 测试覆盖明细

新增 `backend/tests/test_deep_research.py`（148 个单元测试）+ 已有 `test_api.py`（10 个集成测试）+ 已有 `test_config.py`（12 个配置测试），合计 170 个测试用例通过。

#### 单元测试（test_deep_research.py）

- [x] `_truncate_text` / `_short_excerpt`：截断边界、None/空串/非字符串输入
- [x] `build_research_limits`：session budget cap 覆盖全局 cap、timeout 取三者最小值、无 session cap 回退
- [x] `apply_research_limits_to_plan`：dimensions 数量限制、max_results 截断、空列表/非列表输入
- [x] `apply_extraction_limits_to_round`：results 按 min(sources, extracted) 截断、非列表输入
- [x] `_sanitize_search_result`：移除 raw_content、保留 excerpt、retention 标记
- [x] `sanitize_search_rounds`：非列表/非字典输入过滤、逐结果清洗
- [x] `_sanitize_extracted_source`：移除 markdown/main_content、保留 excerpt、retention 标记
- [x] `sanitize_extraction_rounds`：非列表输入过滤
- [x] `build_citation_ledger`：从 citations + evidence_table 构建、空输入处理、dimension_id 关联
- [x] `_freshness_is_stale`：近期日期/过期日期/边界日期/特殊值（unknown/current_session/stale/expired/outdated）/非法日期
- [x] `_conflict_groups`：按 dimension 分组冲突行、无冲突返回空
- [x] `_stale_sources`：识别无新鲜替代的过期维度、有新鲜替代则不标记
- [x] `normalize_dimensions`：空列表/非列表回退默认值、strip/filter
- [x] `build_research_plan`：plan 结构、dimension 必需字段、label/question 解析、未知 dimension fallback
- [x] `run_local_search`：每维度一轮、provider/status/result_count/freshness
- [x] `run_local_extraction`：每 search round 一轮、空 results 处理
- [x] `extract_evidence`：facts/evidence_table/citations 提取、extraction 数据集成、空输入、fact/evidence 必需 key
- [x] `critique_evidence`：四种 verdict（sufficient/insufficient/conflicted/stale）、输出结构完整性
- [x] `build_followup_search_plan`：去重、空 followup、verdict 传递
- [x] `build_open_questions`：从 followups 构建问题、空 verdict
- [x] `get_deep_research_context`：validated_facts/risk_flags/market_signals/pricing_signals/channel_signals/compliance_flags 分类、缺失/非字典输入
- [x] `_context_rows_for_dimensions`：按维度集合过滤
- [x] `_report_key_facts` / `_report_search_rounds` / `_report_evidence_table`：claim 截断、snippet 截断
- [x] `synthesize_report`：go + continue_validation 路径、no-go + collect_more_evidence 路径、report 完整字段
- [x] `deep_research_data_retention_policy`：6 tier 全覆盖、raw_content=transient_only、markdown=excerpt_only、full_content_persisted=false、tier_order
- [x] `FakeSearchProvider` / `FakeExtractionProvider`：string/dict 输入、provider name
- [x] `TavilySearchProvider._normalize_relevance`：high/medium/low/unknown/None/string score
- [x] `TavilySearchProvider.build_single_query_plan`：string 转 plan 结构
- [x] `FirecrawlExtractionProvider._normalize_input`：string 列表转 round、dict 列表透传
- [x] `FirecrawlExtractionProvider._deep_dive_reasons`：direct_url/default_top_source/dimension_trigger/snippet_insufficient
- [x] 常量验证：DEFAULT_DIMENSIONS 5 维、FIRECRAWL_DEEP_DIVE_DIMENSIONS、CRITIC_FOLLOWUP_MAX_ROUNDS、excerpt 限制、FIRECRAWL_DEFAULT_TOP_URL_LIMIT、FIRECRAWL_SNIPPET_MIN_CHARS

#### 集成测试（test_api.py）

- [x] 独立入口创建 marked session（`test_deep_research_independent_entry_creates_marked_session`）
- [x] 完整工作流 plan/search/extract/critic/synthesize（`test_deep_research_workflow_runs_plan_search_extract_critic_synthesize`）
- [x] research_limits 包含全部 6 个 key、plan.limits 与 research_limits 一致
- [x] facts 包含 source_url/citation_id/confidence/freshness/conflict_status/extraction_status
- [x] search_rounds/extraction_rounds/evidence_table/report.dimensions 覆盖全部 5 维度
- [x] data_retention_policy 版本、citation_ledger、critic_verdict=sufficient、report.decision=continue_validation
- [x] storage_keys 按 tenant/session 隔离、search_log/extraction_log/citation_ledger 落盘
- [x] report 不含 raw_content/markdown/main_content、含 snippet_excerpt、quoted_excerpt ≤ 240、claim ≤ 800
- [x] task_graph.deep_research_context 传递到 decompose、PricingAgent/ValidationAgent/LaunchPageAgent 复用证据
- [x] cost_ledgers 包含 deep_research_search/extract、cost_snapshot 与 ledger 一致
- [x] execution_steps 包含全部 5 个 DR 节点 + 3 个 Agent 节点，status=completed
- [x] round resume helpers 跳过 completed rounds（`test_deep_research_round_resume_helpers_skip_completed_rounds`）
- [x] WebSocket progress event payload（`test_deep_research_progress_event_payload`）
- [x] critic 输出四种 verdict + 完整输出结构 + followup plan + open questions（`test_deep_research_critic_outputs_required_verdicts`）
- [x] Tavily provider 标准化 + trace_id + timeout + latency + cost + response summary + relevance + source metadata（`test_tavily_search_provider_normalizes_results`）
- [x] Tavily+Firecrawl combined provider 选择正确 adapter（`test_tavily_firecrawl_combined_provider_selects_real_adapters`）
- [x] Firecrawl provider 标准化 + scrape + deep_dive + price_table/faq/structured_fields + retention（`test_firecrawl_extraction_provider_normalizes_scrape_results`）
- [x] provider 错误记录 + response summary（`test_deep_research_providers_record_errors_and_response_summaries`）
- [x] Firecrawl deep dive URL selection 按 evidence critic 规则（`test_firecrawl_deep_dive_url_selection_uses_evidence_critic_rules`）

#### 配置测试（test_config.py）

- [x] Tavily 环境变量配置 + 数值校验
- [x] Firecrawl 环境变量配置 + 数值校验
- [x] DeepResearch 边界配置 + 数值校验
- [x] 真实 provider 启动校验：缺 API key fail-fast
- [x] Tavily+Firecrawl 组合 provider：缺任一 key fail-fast
- [x] Fake provider 需要 CI flag 或 fake_providers_allowed
- [x] FORGEONE_CI_NO_NETWORK=true 允许 fake provider

### P1：Roundtable Critic

- [x] 新增 Roundtable Critic 作为 DeepResearch 之后的压力测试层
- [x] 保留 Evidence Critic 作为事实充分性检查层
- [x] 建立两层 Critic 流程：DeepResearch -> Evidence Critic -> Roundtable Critic -> decompose
- [x] 新增 runtime 节点：`roundtable_critic`
- [x] 更新 `NODE_COSTS`、`NODE_PROGRESS`、`STAGE_NODES`、`execution_order`，纳入 `roundtable_critic`
- [x] 更新 workflow preview / 前端流程图，展示 `roundtable_critic`
- [x] Roundtable Critic 输入为 `deep_research_report + evidence_table + evidence_critic_verdict`
- [x] Roundtable Critic 状态写入 `state_payload.deep_research.roundtable_critic`
- [x] Roundtable Critic 不模仿具体公众人物，只抽象投资人与经营者思维模型
- [x] 内置 critic agents：Skeptical VC、Operator COO、CFO、Growth Lead、Legal/Platform Risk
- [x] 输出结构化压力测试结果：objections、counter_evidence、stress_questions、kill_criteria、required_followup_search、verdict
- [x] Roundtable Critic 每个 agent 输出结构：`role`、`objections`、`counter_evidence`、`stress_questions`、`kill_criteria`、`confidence`
- [x] Roundtable Critic 汇总输出结构：`verdict`、`top_objections`、`kill_criteria`、`required_followup_search`、`recommended_pivots`、`reasoning_summary`
- [x] 当 Roundtable Critic 产生 required_followup_search 时，回流到 DeepResearch 补充搜索
- [x] `decompose` 阶段读取 Roundtable Critic 的 objections 与 kill criteria 生成后续任务图
- [x] risk panel 读取 Evidence Critic / Roundtable Critic 的风险信号
- [x] `go_no_go_validation_report` 读取 DeepResearch / Critic 的 decision、gaps、kill criteria、open questions
- [x] WebSocket 发送 Roundtable Critic 进度事件
- [x] 建立 Roundtable Critic 单元测试覆盖（2026-04-28）

### P1 测试覆盖明细

新增 33 个 Roundtable Critic 单元测试到 `backend/tests/test_deep_research.py`（总计 127 个）。

#### 单元测试（test_deep_research.py - Roundtable Critic 部分）

- [x] `ROUNDTABLE_CRITIC_ROLES` 常量验证：5 个抽象模型、不含真实人物名、每个 role 映射到维度
- [x] `run_roundtable_critic` 三种 verdict 路径：
  - [x] `pressure_test_passed`：evidence_critic=sufficient + 全维度有证据
  - [x] `needs_followup`：evidence_critic=insufficient / conflicted / stale
  - [x] `pressure_test_warning`：evidence_critic=sufficient + agent confidence=low（无证据）
- [x] `run_roundtable_critic` 输出结构验证：verdict、critic_agents、top_objections、kill_criteria、required_followup_search、recommended_pivots、input_summary、reasoning_summary、created_at
- [x] 每个 critic agent 输出结构：role、objections（非空）、counter_evidence、stress_questions（非空）、kill_criteria（非空）、confidence（medium/low）
- [x] critic_agents 数量 = ROUNDTABLE_CRITIC_ROLES 数量（5）
- [x] top_objections 从所有 agent 收集
- [x] kill_criteria 从所有 agent 收集
- [x] required_followup_search 继承自 evidence_critic_verdict
- [x] recommended_pivots 非空
- [x] input_summary 捕获 report_decision、evidence_row_count、evidence_critic_verdict
- [x] `_roundtable_role_assessment` 角色评估：
  - [x] sufficient evidence → medium confidence
  - [x] insufficient evidence → low confidence
  - [x] no evidence → low confidence
  - [x] 非 sufficient verdict 时 objection 包含 evidence critic status
  - [x] counter_evidence 有 claim 时填充、无证据时为空
- [x] Roundtable Critic WebSocket 进度事件：stage/node/verdict/critic_agent_count/top_objection_count/followup_round_count
- [x] 四种 evidence_critic verdict 对应 Roundtable verdict 映射验证
- [x] `get_deep_research_context` 读取 roundtable_critic：
  - [x] 从 report.roundtable_critic 读取
  - [x] 从 deep_research.roundtable_critic 兜底
  - [x] 缺失时返回空字典
- [x] NODE_COSTS 包含 roundtable_critic 且 > 0
- [x] NODE_PROGRESS[roundtable_critic] > NODE_PROGRESS[deep_research_synthesize]
- [x] execution_order：roundtable_critic 在 deep_research_synthesize 之后、decompose 之前

### P2 Deepresearch/Critic测试 
- [x] 建立 DeepResearch 独立入口 smoke test（2026-04-28）
- [x] 建立主流程接入测试：idea -> classify -> DeepResearch -> Evidence Critic -> Roundtable Critic -> decompose（2026-04-28）
- [x] 验证 `decompose` 能读取 evidence table、critic verdict、objections、kill criteria（2026-04-28）
- [x] 验证后续 `PricingAgent / ValidationAgent / LaunchPageAgent` 能读取 DeepResearch 证据（2026-04-28）
- [x] 验证 `insufficient / conflicted / stale` 会触发补充搜索或输出 open questions（2026-04-28）
- [x] 验证 `required_followup_search` 会从 Roundtable Critic 回流到 DeepResearch（2026-04-28）
- [x] 验证 `deep_research_report` artifact、evidence table、citations、confidence、freshness、conflict status 均落盘（2026-04-28）

#### P2 artifact 落盘测试明细（test_deep_research.py - DeepResearchReportArtifactTests）

新增 20 个单元测试，验证 `deep_research_report` 完整 payload 结构与字段合法性。

- [x] report 包含全部 15 个必需 payload key（executive_summary / decision / go_no_go / key_facts / dimensions / search_rounds / extraction_rounds / evidence_table / citations / citation_ledger / data_retention_policy / critic_verdict / roundtable_critic / open_questions / next_validation_actions）
- [x] executive_summary 提及 idea 与 fact 数量
- [x] decision 与 go_no_go 一致性（sufficient → go / 其他 → no_go）
- [x] key_facts 每条包含 fact_id / dimension_id / claim / source_url / source_title / source_type / citation_id / confidence / freshness / conflict_status / extraction_status
- [x] evidence_table 覆盖全部 5 个维度
- [x] evidence_table 每行包含 dimension_id / claim / source_url / citation_id / confidence / freshness / conflict_status / extraction_status
- [x] evidence_table claim 截断至 STRUCTURED_FACT_MAX_CHARS（800）
- [x] citations 每条包含 citation_id / url / title / quoted_excerpt / source_kind / retention_tier / full_content_persisted=false
- [x] quoted_excerpt 截断至 CITATION_EXCERPT_MAX_CHARS（240）
- [x] citation_ledger 与 citations 一一对应，ledger 行包含 citation_id / url / title / source_kind / retention_tier / excerpt_char_count / full_content_persisted=false
- [x] dimensions 覆盖 5 维，每条含 dimension_id / label / summary / evidence_count
- [x] search_rounds 每条含 round / dimension_id / query / provider / status / results，results 内不含 raw_content
- [x] extraction_rounds 每条含 round / dimension_id / provider / status / sources，sources 内不含 markdown / main_content
- [x] critic_verdict 包含 verdict / confidence / insufficient_dimensions / conflict_groups / stale_sources / required_followup_search / reasoning_summary
- [x] roundtable_critic 包含 verdict / critic_agents / top_objections / kill_criteria / required_followup_search / recommended_pivots / reasoning_summary
- [x] open_questions 为 list
- [x] next_validation_actions 非空
- [x] data_retention_policy 包含 policy_version=deep_research_retention_v1，6 tier 全覆盖，full_content_persisted=false
- [x] report 整体 JSON 可序列化/反序列化
- [x] 验证 workflow preview 包含 DeepResearch / Roundtable Critic 节点
- [x] 验证 WebSocket 会发送 DeepResearch / Roundtable Critic 进度事件
- [x] 验证 result page 和 export renderer 能展示 `deep_research_report`
- [x] 验证 risk panel 和 `go_no_go_validation_report` 会吸收 Critic 风险信号
- [x] 验证 DeepResearch artifact、搜索日志、citation ledger 按 tenant/session 隔离
- [x] 建立真实 Tavily API smoke test，要求配置 `FORGEONE_TAVILY_API_KEY`
- [x] 验证 Tavily 超时、失败、API key 缺失、预算触顶路径
- [x] 验证 DeepResearch/Tavily 成本进入 cost ledger 和 token / cost 面板
- [x] 验证搜索中断后可从最近 search round 恢复
- [x] 验证 raw content、短摘录、URL、citation ledger 按数据保留规则落盘
- [x] 验证报告不保存完整网页正文
- [x] 建立 fake search provider 自动化回归，仅用于 CI 和无网络环境
- [x] 建立端到端回归用例，覆盖 completed、open questions、search loop capped 三种路径

### P3：真实外部执行闭环

- [ ] 将 waitlist、preorder、payment 接入真实 provider
- [ ] 补齐 provider config、secret 管理、环境隔离、sandbox / live 模式
- [ ] 补齐 idempotency key、重试、超时、dead-letter queue、失败补偿
- [ ] 建立外部事件时间线：webhook、email、CRM、analytics、payment
- [ ] 支持失败重试、手动重放、人工补偿、trace_id 关联

### P4：Webhook 与支付上线门槛

- [ ] Webhook signature verification
- [ ] Event deduplication
- [ ] Replay protection
- [ ] Raw payload audit log
- [ ] 支付订单状态机：checkout -> authorized -> captured / refunded / canceled
- [ ] 支付对账、退款、撤销、金额篡改防护、回调最终一致性处理

### P5：公网入口与滥用防护

- [ ] 公开 form、waitlist、preorder、payment 接口增加 rate limit
- [ ] 增加 captcha / honeypot
- [ ] 增加 IP / UA 基础风控
- [ ] 增加重复提交检测、黑名单、隔离队列
- [ ] 提交链路补齐 submission-level idempotency key、去重规则、trace_id、审计关联

### P6：生产安全与配置治理

- [ ] 建立租户级 provider registry 和凭据管理台
- [ ] 支持 sandbox / live 切换、健康检查、配额、熔断、禁用、到期告警
- [ ] 禁止默认 admin/admin、默认 FORGEONE_AUTH_SECRET_KEY、默认开放配置上线
- [ ] 启动阶段增加 fail-fast 环境完整性校验
- [ ] CORS 从全开放收敛为环境级 allowlist
- [ ] 区分后台 API、公开页面、管理后台访问域名策略

### P7：落地页发布治理

- [ ] 支持真实域名、证书状态、CDN、缓存策略
- [ ] 支持页面版本历史、发布回滚、CDN purge
- [ ] 支持 SEO 元数据、robots、sitemap
- [ ] 补齐公开访问安全基线和可用性监控

### P8：观测与测试

- [ ] 外部接入审计日志、监控、告警、限流、滥用防护
- [ ] Adapter contract tests
- [ ] Fake provider integration tests
- [ ] External dispatch smoke tests
- [ ] 浏览器 E2E
- [ ] Synthetic probe
- [ ] 发布后 smoke test
- [ ] 备份恢复演练和灾难恢复演练

### P9：订阅价值与平台化

- [ ] 建立跨项目复用的验证模板和知识沉淀
- [ ] 建立项目对比、历史复盘、复用建议
- [ ] 建立团队协作、评论、审计日志、角色权限
- [ ] 评估下层知识图谱与 Agent 网络的平台化边界

## DeepResearch 未来能力扩展

- [ ] 维度执行策略：MVP 默认执行 5 个维度；生产级 go/no-go 必须覆盖 7 个维度；更多维度只作为行业模板或高风险扩展启用
- [ ] 扩展目标用户与真实痛点维度
- [ ] 扩展交付成本与毛利结构维度
- [ ] 扩展失败案例与反证信号维度
- [ ] demand_signal 扩展研究不按固定轮数增加；按独立来源数、意图强度、社区原文、搜索趋势、付费/预售信号、反证饱和度决定是否继续
- [ ] demand_signal 停止条件：新增来源连续低信息增益、重复同源内容增多、正反证已覆盖、关键结论达到 citation 阈值后停止
- [ ] 支持 source whitelist、source quality score、citation ledger、extraction quality score
- [ ] 支持 evidence conflict graph
- [ ] 支持按行业模板选择 research dimensions
- [ ] 支持跨项目证据复用与历史证据 freshness 复查
- [ ] 建立 DeepResearch quota / rate limit，控制普通用户触发真实 Tavily / Firecrawl 成本
- [ ] 建立 DeepResearch observability：query、source count、extracted URL count、latency、failure reason、critic verdict
- [ ] 建立 Planner、Extractor、Critic、Synthesizer prompt versioning，并在报告中记录版本号
- [ ] 增加 `extraction_rounds` 历史结构，记录 Firecrawl URL、状态、失败原因、成本、latency
- [ ] 建立 DeepResearch provider policy：触发条件、降级策略、URL 去重、质量评分、预算边界
- [ ] 支持 canonical URL 归并、同源限流、重复内容去重
- [ ] 建立 robots / ToS / crawl allowlist 策略
- [ ] 支持 Firecrawl 失败降级：保留 Tavily snippet，不阻断整轮研究
- [ ] citation ledger 区分 `search_snippet`、`extracted_markdown`、`structured_field`
- [ ] 建立 provider budget policy：每轮 Tavily query 数、Firecrawl URL 数、单 session 最大美元成本

## 结果页与用户体验扩展

- [x] 结果页重构为 founder decision brief
- [x] 首屏展示 go / no-go、核心原因、预算消耗、关键风险、3-5 个建议动作
- [x] 增加验证信号面板：目标用户、核心卖点、定价建议、CTA、go / no-go 条件、缺口
- [x] 增加真实执行状态面板：published page、form、waitlist、preorder、payment、email、CRM、analytics
- [x] 增加用户视角下一步：继续验证、修正文案、重新定价、补齐支付、发布落地页
- [x] 首页增加 history sessions 和 recent sessions
- [ ] 补齐 session detail discoverability：最近会话、最近结果页、待处理决策入口

## 运营与商业化

- [ ] 订阅套餐
- [ ] 配额管理
- [ ] 用量计费
- [ ] 账单与发票
- [ ] 客服与运营后台
- [ ] 会话检索、执行重放、人工补偿、导出、封禁

## 数据与合规

- [ ] 数据保留策略
- [ ] 租户级备份恢复
- [ ] 对象存储清理与归档
- [ ] 数据导出
- [ ] 删除请求
- [ ] 敏感字段脱敏
- [ ] 操作审计
- [ ] 密钥轮换
- [ ] 第三方凭据健康检查与失效告警

## 工程与运维

- [ ] staging / production 环境分层
- [ ] 发布审批
- [ ] 灰度发布与回滚编排
- [ ] 错误追踪
- [ ] 性能基线
- [ ] SLO / SLI
- [ ] 值班告警
- [ ] 事故复盘机制
- [ ] 环境初始化脚本
- [ ] 容器化部署
- [ ] CI/CD
- [ ] 端到端自动化回归
- [ ] 将 FastAPI `@app.on_event(startup/shutdown)` 替换为 `lifespan`

## 历史未完成补充池

### 协作与权限

- [ ] 团队协作
- [ ] 评论与批注
- [ ] 共享决策流
- [ ] 审计日志
- [ ] 角色权限控制：owner / operator / viewer

### 外部商业闭环

- [ ] 接入真实支付平台
- [ ] 接入真实发布平台或 waitlist 工具
- [ ] 接入邮件、消息通知、webhook
- [ ] 接入第三方表单、CRM、分析工具
- [ ] 建立真实落地页发布与反馈回收链路

### 保留规则

- [ ] 历史条目保留，不视为废弃
- [ ] 当前优先级以 P0 -> P1 -> P2 -> P3 为主
- [ ] 主控编排、动态任务图、商业原子动作模型稳定后，再决定历史条目的回收顺序

## 近期执行建议

1. 完成 DeepResearch MVP 独立入口。
2. 实现 Planner -> Search -> Extract -> Critic -> Synthesize 工作流。
3. 将 DeepResearch 接入 `classify` 与 `decompose` 之间。
4. 实现 Evidence Critic 与 Roundtable Critic。
5. 建立 DeepResearch / Critic 集成测试。
6. 首版覆盖需求信号、竞品、定价、获客、合规/成本风险 5 个维度。
7. 输出 `deep_research_report` 和 evidence table。
8. 补齐 citations、confidence、freshness、conflict status。
9. 设置 max_depth、搜索数量、来源数量、预算和超时边界。
