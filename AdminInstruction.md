# ForgeOne Admin Instruction

## 1. 这份文档给谁看

这份文档给不需要看代码的人。
用途是快速知道：

- ForgeOne 现在能看什么功能
- 应该怎么体验
- 哪些是 mock 演示，不是真实第三方
- 换到新电脑前需要准备什么

## 2. ForgeOne 现在是什么

ForgeOne 当前是一个 `AI 创业验证操作系统`。
它不是聊天机器人首页。
它更像一个把 `idea -> 分析 -> 多 Agent 执行 -> 人工确认 -> 产物 -> 外部执行模拟` 串起来的控制台。

## 3. 现在能看哪些功能

### 3.1 首页

可以做：

- 登录 / 注册
- 输入一个 `idea`
- 自动拿到场景建议和预算建议
- 创建一个验证会话

### 3.2 会话页

创建会话后，会进入 `/sessions/{id}`。

这里可以看：

- 当前执行到哪一步
- 多个 Agent 的执行状态
- 成本 / token / 风险 / 恢复信息
- 决策卡
- 实时事件

### 3.3 工作流编辑器

“类似 Dify”的可视化编排界面不在首页。
它在会话详情页底部。

打开方式：

1. 先创建一个 session
2. 进入 `/sessions/{id}`
3. 往页面下方滚动
4. 会看到 `工作流编辑器与执行预览`

当前这个编辑器可以：

- 看 workflow 图
- 看当前运行节点
- 调整节点标题、描述、位置
- 调整边的标签和条件
- 保存租户自己的可视化定义

当前它不能做的事：

- 不能直接像 Dify 那样独立新建完整应用
- 不能直接改后端真实执行逻辑
- 现在改的是“可视化 definition”和“preview”

### 3.4 结果页

会话完成后可以看：

- 生成的 artifact
- HTML / PDF / JSON 导出
- mock 外部执行摘要

### 3.5 admin 和普通用户看到什么

`admin` 登录后，首页比普通用户多两个管理区：

- `全局管理员视图`
- `知识审核工作台`

`admin` 可以看到：

- 所有租户的会话总数、运行中数量、待决策数量
- 跨租户会话列表
- 直接跳转到任意会话页和结果页
- 知识白名单配置
- 候选来源发现与导入
- 来源审核
- Fragment 审核
- Atom 创建、审核、发布

普通用户可以看到：

- 首页介绍区
- `idea` 输入和会话创建表单
- 自己的会话页
- 自己会话的结果页
- 自己会话里的 workflow 可视化编辑器与执行预览

普通用户看不到：

- 跨租户会话总览
- 其他人的 session / result 快捷入口
- 知识审核工作台
- 白名单、来源、Fragment、Atom 的审核与发布入口

## 4. 推荐演示路径

如果只是给朋友看功能，推荐只演示 3 条路径。

### 4.1 路径一：数字产品

推荐输入：

```text
ship a Notion template bundle for freelance designers
```

适合看：

- idea 输入
- 自动预算建议
- 会话控制台
- artifact 结果页

### 4.2 路径二：内容业务

推荐输入：

```text
launch a paid newsletter with membership community and sponsorship slots
```

适合看：

- 场景分类
- 动态 workflow
- 内容类交付物

### 4.3 路径三：预售项目

推荐输入：

```text
validate a hardware preorder with refundable deposit and waitlist conversion
```

适合看：

- 最完整的 mock 外部执行流程
- waitlist / preorder / payment 三段演示
- orchestration summary

## 5. 现在怎么体验

### 5.1 启动

仓库根目录执行：

```powershell
.\install-deps.bat
.\start.bat
```

停止：

```powershell
.\stop.bat
```

默认成功后：

- 前端：`http://localhost:3000`
- 后端：`http://localhost:8010`

实际端口以 `.runtime/services.json` 为准。

新电脑迁移时，先执行 `.\install-deps.bat`。
它会自动安装前端依赖，并按当前机器上的 Python 情况安装后端依赖。

### 5.2 登录

当前本地默认 admin：

- 账号：`admin`
- 密码：`admin`

说明：

- 这只适合本地演示
- 不适合生产环境

### 5.3 演示步骤

1. 启动项目
2. 用 `admin / admin` 登录
3. 输入一个 idea
4. 创建 session
5. 在会话页处理决策卡，全部点击 `approve`
6. 等待会话完成
7. 打开结果页
8. 回到会话页底部，看 workflow editor

## 6. 什么叫 mock 演示

当前很多“外部执行”是 mock。
意思是系统会模拟一条真实业务链路，但不会真的调用第三方。

### 6.1 当前 mock 的内容

- form
- waitlist
- preorder
- payment
- email
- webhook
- crm
- analytics

### 6.2 mock 会产生什么

会产生：

- submission 记录
- email outbox 记录
- webhook delivery 记录
- CRM contact 记录
- analytics 事件
- payment checkout / authorized 记录

### 6.3 mock 不会产生什么

不会产生：

- 真正扣款
- 真正发邮件
- 真正发 webhook
- 真正写入第三方 CRM
- 真正写入第三方 analytics

## 7. 最适合朋友看的功能点

如果朋友只看产品感受，建议重点看这些：

- 输入一个 idea 就能起 session
- 自动场景分类和预算建议
- 多 Agent 执行控制台
- 决策卡机制
- 结果页 artifact
- 会话页底部的 workflow editor
- presale 项目的 mock 外部执行链路

## 8. 当前不要误解的点

- 这不是生产环境
- 这不是完整的 Dify 替代品
- 这不是已经接好了真实支付和真实 CRM 的 SaaS
- 现在最强的是“验证流程演示”和“系统编排能力演示”

## 9. 移植到新电脑前要准备什么

建议新电脑至少准备以下内容：

### 9.1 系统环境

- Windows
- PowerShell
- Python 3.11
- Node.js 20+
- npm

### 9.2 仓库目录需要具备

- 项目代码
- `backend/requirements.txt`
- `frontend/package.json`
- `frontend/package-lock.json`
- `start.bat`
- `install-deps.bat`
- `stop.bat`
- `scripts/forgeone-start.ps1`
- `scripts/forgeone-install-deps.ps1`
- `scripts/forgeone-stop.ps1`

### 9.3 本地依赖目录

启动脚本当前至少会检查：

- `frontend/node_modules`

如果只能回退到 Blender Python，还会检查：

- `.pydeps`

如果缺少这些依赖，`start.bat` 会直接失败。

## 10. 新电脑推荐准备步骤

### 10.1 安装 Python 3.11

确认以下命令可用：

```powershell
C:\WINDOWS\py.exe -3.11 --version
```

### 10.2 一键安装依赖

```powershell
.\install-deps.bat
```

### 10.3 脚本会自动处理

- 前端 `node_modules`
- 后端 Python 依赖
- Blender Python 模式下的 `.pydeps`

### 10.4 启动验证

回到仓库根目录执行：

```powershell
.\start.bat
```

成功标准：

- `.runtime/services.json` 里能看到前后端端口
- 能访问前端首页
- `http://localhost:8010/api/health` 返回 `status: ok`

## 11. 新电脑迁移时最容易出问题的地方

### 11.1 Python 版本不对

当前后端要求 Python 3.10+，推荐 Python 3.11。
如果用过旧的 Blender Python 或 Python 3.9，会直接启动失败。

### 11.2 `.pydeps` 没装

如果 `.pydeps` 不存在，后端起不来。

### 11.3 `frontend/node_modules` 没装

如果前端依赖没装，前端起不来。

### 11.4 端口冲突

如果 `3000` 或 `8010` 被占用，脚本会自动找新端口。
所以新电脑上不要只盯着固定端口，要看 `.runtime/services.json`。

## 12. 给朋友演示时怎么讲最省事

可以直接这么说：

1. ForgeOne 不是聊天工具，是创业验证控制台
2. 先输入一个 idea
3. 系统会自动判断场景、预算和执行路线
4. 多个 Agent 会并行产出结果
5. 中间关键节点可以人工确认
6. 最后可以看到页面、定价、验证报告和 mock 业务链路
7. 底部还能看到 workflow 图和可视化编排界面

## 13. 更详细资料

如果需要更完整的技术说明，看：

- [instruction.md](./instruction.md)
- [Roadmap.md](./Roadmap.md)

如果需要判断是否能上线，只看：

- [Roadmap.md](./Roadmap.md)
