# Implementation Status

Last updated: 2026-09-26

Canonical design: [docs/ddd-development-design.md](ddd-development-design.md)

本文件记录 **Current Reality**，不替代设计或任务计划。检查基线为 `90bae21`（`架构及ddd开发文档`）及本次治理文档工作树；读取和更新时都应重新核对实际代码。新增治理文档不代表应用工作包已开始实现。

## Work Package Status

| Package | Status | Verification | Notes |
|---|---|---|---|
| P1-01 | NOT_STARTED | - | 工程与账号；已建立任务说明，无工程代码 |
| P1-02 | NOT_STARTED | - | 文件与版本；依赖 P1-01 |
| P1-03 | NOT_STARTED | - | 解析与发布；依赖 P1-02 |
| P1-04 | NOT_STARTED | - | 阅读领域与页面；依赖 P1-03 |
| P2-01 | NOT_STARTED | - | Runtime 与模型端口；依赖 P1-01、P1-04 |
| P2-02 | NOT_STARTED | - | 教学与上下文；依赖 P2-01 |
| P2-03 | NOT_STARTED | - | 缓存与阅读体验；依赖 P2-02 |
| P3-01 | NOT_STARTED | - | 学习积累；依赖 P2-03 |
| P3-02 | NOT_STARTED | - | 复习材料与出题；依赖 P3-01 |
| P3-03 | NOT_STARTED | - | 作答与讲评；依赖 P3-02、P2-01 |
| P4-01 | NOT_STARTED | - | 部署与运行检查；依赖 P1–P3 |
| P4-02 | NOT_STARTED | - | 备份与恢复；依赖 P4-01 |
| P4-03 | NOT_STARTED | - | 完整验收；依赖 P4-02 |

允许的状态只有以下五个：

| Status | 含义 |
|---|---|
| NOT_STARTED | 尚未开始代码实现；已有设计、任务拆分或治理文档仍属于此状态 |
| IN_PROGRESS | 部分代码已经实现，但尚未满足 Definition of Done |
| BLOCKED | 存在阻止下一步所需实现／验收的明确问题；必须记录原因、影响任务及解除条件 |
| IMPLEMENTED | 范围内代码已实现完成，但完整验收尚未通过或证据尚不完整 |
| VERIFIED | 对应工作包所需验证全部通过，满足 Definition of Done，且有可复查证据 |

状态不得只凭自述推进。只完成部分 Txx 时，Verification 必须标注子范围及剩余场景；包的 VERIFIED 也不等于整个应用或所有 T01–T25 已通过。

## Current Repository Reality

本次检查前，仓库只包含 `prompt.txt`、`docs/ddd-development-design.md`、`docs/agent-architecture-plan.md` 三个受版本控制的文件，Git 工作树干净，没有现存实现需要迁移或保留。

本次新增治理层后，除 `.git/` 元数据外的文件结构为：

```text
AGENTS.md
README.md
prompt.txt
docs/
  agent-architecture-plan.md
  ddd-development-design.md
  implementation-status.md
  work-packages/
    P1-01.md
  adr/
    README.md
```

- 已存在的是设计、产品需求及治理文档；ADR 目录仅有规范，尚无决策记录。
- `apps/`、`packages/`、`tests/`、`prompts/`、`deploy/` 均未建立。
- 没有 package manifest、pnpm workspace／锁文件、TypeScript／ESLint 配置、构建脚本、CI、测试工具配置。
- 没有 Fastify 服务、React 页面、contracts 实现、shared-kernel 实现、SQLite 数据库、Drizzle schema 或迁移。
- Identity、Library、Reading、Tutoring、Learning、Review 及技术支撑 Runtime 均不存在；没有模型调用或 AgentRun 实现。
- DDD 中的类型示例、SQL、目录、配置及“profile 已实现”等阶段前提不是当前代码事实。

## Active Work Package

- Package: [P1-01 Engineering Foundation & Identity](work-packages/P1-01.md)
- Status: `NOT_STARTED`
- 当前活动是治理文档交付，不是 P1-01 Coding。
- 下一步：按 P1-01-T01 开始工程基础；在身份契约定稿和 HTTP 实现前处理下文 G-01。
- P1-01 的工作包依赖：无（DDD §15.1）。G-01 不阻止 workspace、边界、日志与数据库基础工作，但不能被身份接口实现静默绕过。
- 不创建 P1-02 ～ P4-03 的详细任务文档；其名称、顺序和依赖继续以 DDD §15 为准。

## Verification Evidence

| 检查 | 当前证据／结果 | 范围限制 |
|---|---|---|
| Repository baseline | 2026-09-26：`git ls-tree -r --name-only HEAD`、`git status --short --branch`、文件树检查 | 确认初始仅三份输入，基线工作树干净 |
| Design reading | 完整阅读 DDD、架构背景与 prompt | 只证明分析输入完整，不证明实现 |
| Governance consistency review | 2026-09-26：PASS。一次性 `python3` 静态检查通过 41 个本地链接／锚点、112 处 DDD 章节引用、13 个包的完整清单、12 个任务与 12 个 AC；代码围栏及逐文件 `git diff --no-index --check` 通过 | 仅文档静态验证；人工同时复核设计、范围、术语、测试追踪和 Agent 入口，无应用通过结论 |
| Original design preservation | PASS：三份原始输入 SHA-256 与修改前一致，`git diff --exit-code -- docs/ddd-development-design.md docs/agent-architecture-plan.md prompt.txt` 返回 0 | 本次只有五个治理 Markdown 文件新增，未改变原设计与产品需求 |
| Build / typecheck / lint / boundary checks | NOT RUN：工程及命令尚未建立 | 无通过结论 |
| Domain / SQLite / HTTP / model / E2E tests | NOT RUN：没有实现和测试 | T01–T25 均未通过运行验收；T20 当前也没有局部通过记录 |

后续每次更新 Verification 时记录：日期、提交或明确工作树范围、实际命令、退出结果、测试文件／证据路径、关联 AC／INV／Txx、未运行或失败原因。不要将敏感数据、凭据或原始请求载荷放入证据。

## Known Design / Implementation Gaps

以下均来自现有文档与当前仓库的核对；本次未修改 canonical design，也未替这些问题作出新的业务决定。尚未实现的计划模块本身不列为 technical debt。

| ID | 发现与依据 | 影响与处理边界 |
|---|---|---|
| G-01 | **Identity 的公开契约尚未完全明确。** DDD §9.2 的 login／me 只指定“用户公开信息”，没有字段白名单；§9.4 未明确错误密码／未知账号的错误码映射。§4 同时禁止 credentials types 进入 contracts，而 §9.2 登录请求包含 password，登录请求 schema 的共享范围尚需明确。 | 不阻止工程基础开工。P1-01-T10 定稿前须澄清公开字段、登录失败语义和请求 schema 所在边界，按 AGENTS 的 Level 2 同步设计。不得先自行定义宽泛 UserDto、向前端暴露数据库 User 或放宽凭据隔离；重要授权／隐私边界变化需 ADR。 |
| G-02 | **T20 覆盖跨越多个阶段。** DDD §15.1 的 P1-01 要求登录与资源归属验证；§14.2 T20 还包括书籍、源文件、任务及复习，而这些在 P1-02 及之后才出现。 | P1-01 只记录真实 Identity／Session 的 HTTP 隔离与身份安全覆盖。后续资源逐个实现归属测试，最终在 P4-03 汇总完整 T20；不能把本阶段局部通过、空路由 404 或模拟资源当作完整 T20 通过。此为验收追踪边界，不是删减 T20。 |
| G-03 | **Reading 到 Review 的阶段前提需要明确。** DDD §6.2 要求首次跨越八片段阈值时，通过 Review 应用接口在同一事务创建提醒；但 §15.1 的 P1-04 在 §15.3 的 P3-02（复习提醒）之前，依赖表未说明该最小能力如何先到位。 | 不影响 P1-01。展开 P1-04／相关依赖之前须确认阶段安排或最小依赖，并同步设计／任务。不得因此在 P1-01 实现 Review，也不得在 P1-04 用 no-op 掩盖提醒的事务要求。 |

## Status Maintenance Rules

- 当前包切换时同时更新 Active Work Package、状态表和验证证据；新包依赖必须先核对实际结果。
- 包内部分完成、阻塞解除和验收失败也更新本文件；包内详细任务仍放在对应 work-package。
- 缺口解决后保留决定及设计／ADR／验证位置，并注明已解决；不要以删除条目掩盖未处理问题。
- 代码与设计的差异先分类，不能只更新本文件就视为架构变更已获接受。
