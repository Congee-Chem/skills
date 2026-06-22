# Vibecoding 全栈开发流程（阅读版）

> 这是 `/vibecoding` skill（[../vibecoding/SKILL.md](../vibecoding/SKILL.md)）的人类阅读版，把同一套流程写成可通读的规范文档。
> 核心理念：**用最短的可验证步长前进**——每一步都产出能跑、能看、能回滚的东西。AI 是结对的全栈工程师，不是甩手掌柜：关键架构与安全决策由人确认。

---

## 阶段 0 · 启动：确定模式与技术栈

写任何代码前，先一次性对齐三件事。

### 1）协作模式
- **个人版**：一个人 + AI 结对，目标是快、可回滚、够用。轻流程。
- **团队版**：多人协作，目标是可协作、可评审、可追溯。加分支 / PR / CI / 约定。

### 2）技术栈（4 个维度直接问：协作模式 / 后端语言 / 前端 / 范围；其余带默认值快速确认）

> 启动时一批问清 **协作模式、主力后端语言、前端、本次范围** 4 项；**数据库 / ORM / 鉴权 / 部署** 用「推荐」默认值逐层快速确认。

| 层 | 选项 | 何时选 |
|---|---|---|
| **主力后端语言** | Python (FastAPI / Django) · Go (Gin·Echo) · Java (Spring Boot) · .NET (ASP.NET Core) | 见下方「语言选型速查」 |
| **前端** | React (Next.js·推荐) · Vue (Nuxt) · SvelteKit · Astro(内容站) · 无前端(纯 API) | 中后台/通用 Web→Next；轻量→SvelteKit；内容站→Astro；后端非 JS 时前端独立工程、独立部署 |
| 数据库 | **PostgreSQL（推荐）** · MySQL · SQLite(本地/原型) · MongoDB | 默认 Postgres；原型期可先 SQLite，重关系/事务别上 Mongo |
| ORM/数据层 | 跟随语言：Python→SQLAlchemy / Django ORM · Go→GORM / sqlc · Java→Spring Data JPA / MyBatis · .NET→EF Core | 别原生拼 SQL 字符串 |
| 鉴权 | 框架原生优先：Java→Spring Security · .NET→ASP.NET Identity · Python→Authlib/fastapi-users · Go→JWT 库 ／ 或托管(Clerk/Auth0) | 别从零写密码体系 |
| 部署 | **Docker + 容器平台（推荐，重后端）** · 云厂商(AWS·阿里云·Azure) · Railway·Render · Serverless(仅轻量) | 常驻服务走容器；Serverless 只适合轻量/突发 |

**语言选型速查**
- **Python** — 原型/迭代最快，AI·数据·脚本生态最强；弱在 CPU 密集与高并发（靠 async 缓解）。适合 MVP、数据类、AI 应用。
- **Go** — 高并发网络服务、云原生首选，单二进制部署最省心；生态克制、样板少。适合高吞吐 API、基础设施、微服务。
- **Java (Spring Boot)** — 大型企业系统、强类型、生态与中间件最全；启动重、样板多、上手成本高。适合复杂业务、长期演进的团队系统。
- **.NET (ASP.NET Core)** — 性能优秀、C# 体验好、Azure/Windows 生态友好；跨平台已成熟但社区偏微软栈。适合企业应用、对性能与类型安全都有要求的团队。

### 3）本次范围
明确这次交付的是「一个完整 MVP」还是「在已有项目上加一个功能」；用一句话写清目标用户与核心用户故事。

### 🔎 搭配合理性检查（选型后必过一遍）

收齐选择后**先别脚手架**，命中下表即说明问题并给替代方案，让人拍板后再继续：

| 不合理组合 | 问题 | 建议 |
|---|---|---|
| Java / .NET + 「就想快速验证个想法」 | 启动重、样板多，原型期拖慢迭代 | 原型期改用 Python(FastAPI) 或 Go，验证成功再考虑重平台重写 |
| Go / Java / .NET 常驻服务 + Serverless/Vercel | 冷启动、长连接、常驻进程不友好 | 走 Docker + 容器平台 / 云主机；Serverless 只留给轻量函数 |
| 生产多人高并发 + SQLite | 单写锁、并发写易瓶颈 | 上 PostgreSQL / MySQL；SQLite 仅限本地/原型/单机小工具 |
| 强关系、强事务业务 + MongoDB | 跨文档事务与关联查询吃力 | 用 PostgreSQL；确有文档/弹性 schema 场景再局部用 Mongo |
| 纯内容/展示站 + 重后端语言 + 关系库 | 杀鸡用牛刀，运维成本高 | 用 Astro/Next 静态 + 无后端 / Headless CMS |
| 后端 Java/Go/.NET + 前端 Next「同构后端」 | 同构是 JS 专属，跨语言用不了 | 明确前后端分离：前端只做 UI + 调 REST/GraphQL |
| 单人极简项目 + 微服务拆分 | 运维/联调成本远超收益 | 先单体，有明确扩展压力再拆 |
| 选了 ORM 又大量手拼 SQL 字符串 | 注入风险 + 维护割裂 | 统一走 ORM/查询构建器或参数化查询 |

> ⚠️ **优化点**：跳过选型直接开干是最常见的翻车根因。强制先选型、过合理性检查、再对齐范围；最终选型与「为什么」写进 `DECISIONS.md`。

---

## 阶段 1 · 需求澄清与切片

- 把想法拆成**用户故事**：「作为 X，我要 Y，以便 Z」。
- 按价值排序，划出 **MVP 边界**，并显式写下这次**不做**什么（防范围蔓延）。
- 把 MVP 切成**垂直切片**：每片贯穿前端→后端→数据、能独立演示；而不是按「先写完所有后端再写前端」的水平分层。
- 用任务清单登记切片，作为进度的唯一事实来源。

> ⚠️ **优化点**：vibecoding 最容易「一直加功能、永不收口」。MVP 边界 + 显式「不做清单」是刹车。

---

## 阶段 2 · 架构与脚手架

1. **方案速记**：仓库根写 `DECISIONS.md`，一行一条 ADR 风格记录选型、目录约定、关键取舍。
2. **脚手架**：用官方脚手架起项目，别手搓配置。

   **后端**
   - Python (FastAPI)：`uv init <api>` → `uv add fastapi "uvicorn[standard]" sqlalchemy alembic`；运行 `uv run uvicorn app.main:app --reload`
   - Python (Django)：`uv init <api>` → `uv add django djangorestframework` → `django-admin startproject config .`
   - Go (Gin)：`go mod init <module>` → `go get github.com/gin-gonic/gin gorm.io/gorm`；入口 `main.go`
   - Java (Spring Boot)：`spring init --dependencies=web,data-jpa,postgresql,security --build=gradle <api>`（或用 start.spring.io 生成）
   - .NET (ASP.NET Core)：`dotnet new webapi -o <Api>` → `dotnet add package Microsoft.EntityFrameworkCore.Design`

   **前端**（后端非 JS 时独立工程）
   - React：`npx create-next-app@latest <web>`　·　Vue：`npm create nuxt@latest <web>`
   - SvelteKit：`npx sv create <web>`　·　Astro(内容站)：`npm create astro@latest <web>`

   **数据库**（本地用容器免装）
   - Postgres：`docker run -d --name pg -e POSTGRES_PASSWORD=dev -p 5432:5432 postgres:16`
   - MySQL：`docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=dev -p 3306:3306 mysql:8`

3. **第一根主线先打通**：建一个最小的「前端按钮 → 调接口 → 读数据库 → 显示结果」端到端骨架，先让它在本地跑起来，再谈业务。
4. 配好 `.env.example`、`.gitignore`、格式化/lint（前端 Prettier+ESLint；Python Ruff；Go gofmt；Java Spotless；.NET dotnet format）。

> ⚠️ **优化点**：先搭骨架跑通端到端，比先精雕某一层更能暴露集成问题（CORS、鉴权、环境变量、构建）。集成问题越晚发现越贵。

---

## 阶段 3 · 增量实现（核心循环）

对每个垂直切片跑这个小循环：

1. **取一片** → 标记进行中。
2. **实现**：写最直接能跑的代码；遵循 `DECISIONS.md` 约定，复用已有模块，风格对齐周围代码。
3. **自测**：本地真的点一遍 / 调一次接口，确认行为符合预期。
4. **提交**：一个切片一次提交，message 说清「做了什么、为什么」。
5. 标记完成 → 取下一片。**一次只推进一片，不并行铺开半成品。**

> ⚠️ **优化点**：AI 容易「一次生成一大坨、整体跑不起来」。强制小步：每片都能跑、能提交、能回滚，出问题回退成本最低。

---

## 阶段 4 · 验证与硬化

- **跑通核心用户故事**：按阶段 1 的故事逐条手动走查。
- **测试**：给核心业务逻辑补关键单测/集成测试，覆盖「改了会出事」的路径（不追求覆盖率数字）。
- **边界与错误态**：空数据、超长输入、网络失败、未登录、权限不足——至少不崩。
- **安全自查**：密钥不进仓库；输入校验；用 ORM/参数化防注入；鉴权在服务端而非仅前端隐藏。
- **性能体检**：N+1 查询、未加索引的高频查询、首屏体积。

> ⚠️ **优化点**：vibecoding 常把「能跑」当「做完」。错误态与安全是最常被跳过、上线后最贵的两块。

---

## 阶段 5 · 评审与交付（两条轨道）

### 🧑 个人版
- 自审当前 diff（可用 `code-review`），按建议修一遍。
- 更新 `README`：怎么装、怎么跑、环境变量说明。
- 部署到所选平台，并**冒烟测试线上环境**（别只在本地验证就宣布完成）。
- 把「还没做/已知问题」记进 `DECISIONS.md` 或 issue，给未来的自己留线索。

### 👥 团队版
- **分支与 PR**：功能分支 → 开 PR；描述写清动机、改动、验证方式、风险。
- **CI**：PR 上自动跑 lint + test + build，绿了才可合。没有就先配一个最小 workflow。
- **代码评审**：先用工具过一遍降低 reviewer 负担；人类 review 关注架构与业务正确性。
- **约定对齐**：Conventional Commits、`CODEOWNERS`、PR 模板、`CONTRIBUTING.md`。
- **环境分离**：dev / staging / prod 分环境与密钥；迁移脚本（Alembic / GORM migrate / Flyway·Liquibase / EF Migrations）纳入流程，禁止手改生产库。
- **可观测性**：日志、错误上报（Sentry 等）、基础监控。

> ⚠️ **优化点（个人→团队最大落差）**：个人版可靠「脑子里记着」，团队版必须把约定**外显成文件和 CI**，否则协作在不一致中内耗。

---

## 贯穿全程的原则

- **可运行优先**：任何时刻工作区都应能跑起来。
- **小步可回滚**：宁可多次小提交，不要一次大爆炸。
- **决策留痕**：重要取舍写进 `DECISIONS.md`，而不是只活在对话里。
- **先对齐再动手**：每阶段进入下一步前，复述「现在在哪、下一步做什么」。
- **AI 是结对工程师**：关键架构与安全决策让人确认，不默默替人拍板。

---

## 使用提示
- **已有项目接入**：跳过脚手架部分，但仍快速过一遍选型现状并补 `DECISIONS.md`。
- **很急/只要原型**：可压缩为「个人版 + SQLite + 托管部署」，但「不做清单」与「冒烟测试」两道刹车不要省。
