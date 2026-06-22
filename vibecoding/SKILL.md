---
name: vibecoding
description: >-
  全栈 Vibecoding 开发流程：从需求澄清 → 技术栈选型 → 架构设计 → 脚手架 → 垂直切片增量实现 →
  自测验证 → 评审 → 部署交付，驱动人与 AI 结对快速且可控地交付一个全栈功能/项目。
  调用时让使用者选择技术栈与协作模式（个人 / 团队），然后按对应轨道推进。
  当用户想"用 vibecoding 方式从零做一个全栈项目/功能"、"按规范流程结对开发"、
  或"帮我把一个想法落地成可运行可部署的代码"时使用。
---

# Vibecoding 全栈开发流程

你（Claude）是结对的全栈工程师。目标不是一口气把代码写完，而是**用最短的可验证步长前进**：每一步都产出能跑、能看、能回滚的东西。下面是驱动整个流程的剧本。**严格按阶段推进，每个阶段结束前先和用户对齐再进入下一阶段。**

## 阶段 0 · 启动：确定模式与技术栈

在写任何代码前，用 **一批 `AskUserQuestion`（4 个问题）** 一次性问清下面 4 个维度（不要逐条挤牙膏）：**① 协作模式 · ② 主力后端语言 · ③ 前端 · ④ 本次范围**。剩下的 **数据库 / ORM / 鉴权 / 部署** 在这批之后，用带「推荐」默认值**逐层快速确认**（用户没特别要求就用推荐，不必再开一轮选择题）。

1. **协作模式**（第 1 问） — 决定走哪条轨道：
   - `个人版`：一个人 + AI 结对，目标是快、可回滚、够用。轻流程。
   - `团队版`：多人协作，目标是可协作、可评审、可追溯。加分支/PR/CI/约定。

2. **主力后端语言**（第 2 问） — 驱动后面 ORM/鉴权/部署的连带选择，选项：
   `Python (FastAPI/Django)` / `Go (Gin·Echo)` / `Java (Spring Boot)` / `.NET (ASP.NET Core)`。用户犹豫时用下方「语言选型速查」给建议。

3. **前端**（第 3 问） — 选项：
   `React (Next.js)` / `Vue (Nuxt)` / `SvelteKit` / `Astro (内容/展示站)` / `无前端 (纯 API)`。
   - 没主见、要中后台/通用 Web → 推荐 **React (Next.js)**：生态最大、组件库最全。
   - 偏好渐进式/模板语法 → **Vue (Nuxt)**；要极致轻量/性能 → **SvelteKit**；纯内容/营销站 → **Astro**；只做后端服务 → **无前端**。
   - 后端非 JS（Go/Java/.NET）时，前端一律**独立工程、独立部署**，通过 REST/GraphQL 对接，不用 Next 的同构后端能力。

4. **本次范围**（第 4 问） — 见下方第 3 节。

   收齐这 4 项后，再用「推荐」默认值快速过一遍剩余层（用户可随时改）：

   | 层 | 选项 | 何时选 |
   |---|---|---|
   | 数据库 | **PostgreSQL (推荐)** / MySQL / SQLite(本地/原型) / MongoDB | 默认 Postgres；原型期可先 SQLite，重关系/事务别上 Mongo |
   | ORM/数据层 | **跟随语言**：Python→SQLAlchemy / Django ORM · Go→GORM / sqlc · Java→Spring Data JPA / MyBatis · .NET→EF Core | 别原生拼 SQL 字符串 |
   | 鉴权 | **框架原生优先**：Java→Spring Security · .NET→ASP.NET Identity · Python→Authlib/fastapi-users · Go→JWT 库 ／ 或托管(Clerk/Auth0) | 别从零写密码体系 |
   | 部署 | **Docker + 容器平台 (推荐, 重后端)** / 云厂商(AWS·阿里云·Azure) / Railway·Render / Serverless(仅轻量) | 常驻服务走容器；Serverless 只适合轻量/突发 |

   **语言选型速查**（用户犹豫时用这个给建议）：
   - **Python** — 原型/迭代最快，AI·数据·脚本生态最强；弱在 CPU 密集与高并发（靠 async 缓解）。适合：MVP、数据类、AI 应用。
   - **Go** — 高并发网络服务、云原生首选，单二进制部署最省心；生态克制、样板少。适合：高吞吐 API、基础设施、微服务。
   - **Java (Spring Boot)** — 大型企业系统、强类型、生态与中间件最全；启动重、样板多、上手成本高。适合：复杂业务、长期演进的团队系统。
   - **.NET (ASP.NET Core)** — 性能优秀、C# 体验好、Azure/Windows 生态友好；跨平台已成熟但社区偏微软栈。适合：企业应用、对性能与类型安全都有要求的团队。

**本次范围（第 4 问，详情）** — 这次要交付的是「一个完整 MVP」还是「在已有项目上加一个功能」（或「只想快速出原型」）？目标用户/核心用户故事一句话是什么？

### 🔎 搭配合理性检查（选型后必须过一遍，不合理就主动提建议）

收齐选择后，**先别急着脚手架**，用下面的规则核对一遍；命中即向用户说明问题并给替代方案，让用户拍板后再继续：

| 不合理组合 | 问题 | 建议 |
|---|---|---|
| Java / .NET + 「我就想快速验证个想法」 | 启动重、样板多，原型期拖慢迭代 | 原型阶段改用 **Python(FastAPI)** 或 **Go**，验证成功再考虑重平台重写 |
| Go / Java / .NET 常驻服务 + **Serverless/Vercel** 部署 | 冷启动、长连接、常驻进程不友好 | 走 **Docker + 容器平台 / 云主机**；Serverless 只留给轻量函数 |
| 生产多人高并发 + **SQLite** | 单写锁、并发写易瓶颈 | 上 **PostgreSQL / MySQL**；SQLite 仅限本地/原型/单机小工具 |
| 强关系、强事务业务 + **MongoDB** | 跨文档事务与关联查询吃力 | 用 **PostgreSQL**；确有文档/弹性 schema 场景再局部用 Mongo |
| 纯内容/展示站 + 重后端语言 + 关系库 | 杀鸡用牛刀，运维成本高 | 用 **Astro/Next 静态** + 无后端 / Headless CMS |
| 后端 Java/Go/.NET + 前端 Next 的「Server Actions/同构后端」 | 同构能力是 JS 专属，跨语言用不了 | 明确**前后端分离**：前端只做 UI + 调 REST/GraphQL，后端独立工程 |
| 单人极简项目 + 微服务拆分 | 运维/联调成本远超收益 | 先**单体**，有明确扩展压力再拆 |
| 选了 ORM 又大量手拼 SQL 字符串 | 注入风险 + 维护割裂 | 统一走 ORM/查询构建器或参数化查询 |

> ⚠️ **优化点（很多 vibecoding 翻车的根因）**：跳过选型直接开干，写到一半才发现栈不合适。这里强制先选型、过合理性检查、再对齐范围。最终选型与「为什么这么选」要**写进项目里**（见阶段 2 的 `DECISIONS.md`），后续不再反复纠结。

## 阶段 1 · 需求澄清与切片

- 把"想法"拆成**用户故事**，每条形如「作为 X，我要 Y，以便 Z」。
- 按价值排序，划出 **MVP 边界**：明确这次**不做**什么（写下来，防止范围蔓延 ⚠️）。
- 把 MVP 切成**垂直切片**（每片都贯穿前端→后端→数据，能独立演示），而不是按"先写完所有后端再写前端"的水平分层。
- 用 `TodoWrite` 把切片登记成任务清单，作为后续进度的唯一事实来源。

> ⚠️ **优化点**：vibecoding 最容易陷入"一直加功能、永远不收口"。MVP 边界 + 显式的"不做清单"是刹车。

## 阶段 2 · 架构与脚手架

1. **方案速记**：在仓库根写一份 `DECISIONS.md`，记录技术选型、目录结构约定、关键取舍（一行一条，ADR 风格）。这是给未来的你/队友看的。
2. **脚手架**：用官方脚手架起项目，别手搓配置。按所选栈取对应命令：

   **后端**
   - Python (FastAPI)：`uv init <api>` → `uv add fastapi "uvicorn[standard]" sqlalchemy alembic`；运行 `uv run uvicorn app.main:app --reload`
   - Python (Django)：`uv init <api>` → `uv add django djangorestframework` → `django-admin startproject config .`
   - Go (Gin)：`go mod init <module>` → `go get github.com/gin-gonic/gin gorm.io/gorm`；入口 `main.go`
   - Java (Spring Boot)：`spring init --dependencies=web,data-jpa,postgresql,security --build=gradle <api>`（或用 start.spring.io 生成）
   - .NET (ASP.NET Core)：`dotnet new webapi -o <Api>` → `dotnet add package Microsoft.EntityFrameworkCore.Design`

   **前端**（后端非 JS 时独立工程）
   - React：`npx create-next-app@latest <web>`　·　Vue：`npm create nuxt@latest <web>`
   - SvelteKit：`npx sv create <web>`　·　Astro(内容站)：`npm create astro@latest <web>`

   **数据库（本地起服务，统一用容器免装）**
   - Postgres：`docker run -d --name pg -e POSTGRES_PASSWORD=dev -p 5432:5432 postgres:16`
   - MySQL：`docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=dev -p 3306:3306 mysql:8`
3. **第一根主线先打通**：建一个最小的"前端按钮 → 调接口 → 读数据库 → 显示结果"的端到端骨架，**先让它在本地跑起来**，再谈业务。
4. 配好 `.env.example`、`.gitignore`、格式化/lint（Prettier + ESLint / Ruff 等）。

> ⚠️ **优化点**：先搭骨架跑通端到端，比先精雕某一层更能暴露集成问题（CORS、鉴权、环境变量、构建）。集成问题越晚发现越贵。

## 阶段 3 · 增量实现（核心循环）

对任务清单里的**每一个垂直切片**，跑这个小循环：

1. **取一片** → 在 `TodoWrite` 标记 in-progress。
2. **实现**：写最直接能跑的代码；遵循 `DECISIONS.md` 的约定，复用已有模块，命名/风格对齐周围代码。
3. **自测**：本地跑起来真的点一遍 / 调一次接口，确认行为符合预期（必要时用 `verify` skill）。
4. **提交**：一个切片一次提交，commit message 说清"做了什么、为什么"。
5. 标记 done → 取下一片。**一次只推进一片，不要并行铺开半成品。**

> ⚠️ **优化点**：AI 容易"一次生成一大坨然后整体跑不起来"。强制小步：每片都必须能跑、能提交、能回滚。出问题时回退成本最低。

## 阶段 4 · 验证与硬化

- **跑通核心用户故事**：按阶段 1 的故事逐条手动走查。
- **测试**：给核心业务逻辑补关键单测/集成测试（不追求覆盖率数字，覆盖"改了会出事"的路径）。
- **边界与错误态**：空数据、超长输入、网络失败、未登录、权限不足——至少处理不崩。
- **安全自查**：密钥不进仓库、输入校验、SQL 注入（用 ORM/参数化）、鉴权在服务端而非仅前端隐藏。
- **性能体检**：明显的 N+1 查询、未加索引的高频查询、首屏体积。

> ⚠️ **优化点**：vibecoding 常把"能跑"当成"做完"。错误态和安全是最常被跳过、上线后最贵的两块，这里显式拦一道。

## 阶段 5 · 评审与交付（两条轨道分叉）

### 🧑 个人版
- 用 `code-review` skill 自审当前 diff，按建议修一遍。
- 更新 `README`：怎么装、怎么跑、环境变量说明。
- 部署到所选平台，**冒烟测试线上环境**（别只在本地验证就宣布完成 ⚠️）。
- 把"还没做/已知问题"记进 `DECISIONS.md` 或 issue，给未来的自己留线索。

### 👥 团队版
- **分支与 PR**：功能分支 → 开 PR；PR 描述写清动机、改动、验证方式、风险。
- **CI**：PR 上自动跑 lint + test + build，绿了才可合。没有 CI 先帮用户配一个最小 workflow。
- **代码评审**：可先用 `code-review` skill 过一遍降低 reviewer 负担；人类 review 关注架构与业务正确性。
- **约定对齐**：commit 规范（Conventional Commits）、`CODEOWNERS`、PR 模板、`CONTRIBUTING.md`。
- **环境分离**：dev / staging / prod 分环境与密钥；迁移脚本（Prisma migrate 等）纳入流程，禁止手改生产库。
- **可观测性**：日志、错误上报（Sentry 等）、基础监控，便于多人协作时定位问题。

> ⚠️ **优化点（个人→团队最大落差）**：个人版可以靠"脑子里记着"，团队版必须把约定**外显成文件和 CI**，否则协作会在不一致中内耗。

## 贯穿全程的原则

- **可运行优先**：任何时刻 `main`/工作区都应是能跑起来的状态。
- **小步可回滚**：宁可多次小提交，不要一次大爆炸。
- **决策留痕**：重要取舍写进 `DECISIONS.md`，而不是只活在对话里。
- **先对齐再动手**：每个阶段进入下一阶段前，向用户复述"我们现在在哪、下一步做什么"。
- **AI 是结对工程师不是甩手掌柜**：关键架构与安全决策让用户确认，不要默默替用户拍板。

## 使用提示

- 已有项目接入：跳过阶段 0 的脚手架部分，但仍要快速过一遍选型现状并补 `DECISIONS.md`。
- 用户很急/只想要原型：可压缩到 `个人版` + SQLite + 托管部署，但**不做清单和冒烟测试这两道刹车不要省**。
