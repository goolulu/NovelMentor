# Coding Agent Development Rules

本文件规定 Coding Agent 在本仓库中如何工作，适用于 Codex、Claude Code、Gemini CLI 等工具。开始任务时 MUST 显式读取本文件及当前工作包，不依赖某个客户端是否自动加载指令。`MUST` 表示必须，`MUST NOT` 表示禁止。

## Source of Truth

| 职责 | 文件 | 使用规则 |
|---|---|---|
| Canonical development specification | [docs/ddd-development-design.md](docs/ddd-development-design.md) | 系统应该是什么；领域、架构、契约、事务、验收和工作包顺序的权威依据 |
| Architecture rationale | [docs/agent-architecture-plan.md](docs/agent-architecture-plan.md) | 解释架构背景与选择；与 DDD 不一致时以 DDD 为准 |
| Tutor / product behavior | [prompt.txt](prompt.txt) | 产品与教学需求输入；工程落实及首版边界以 DDD 为准 |
| Implementation reality | [docs/implementation-status.md](docs/implementation-status.md) | 已存在的代码、当前工作包、验证证据与已发现缺口；使用前仍须核对代码 |
| Current task specification | `docs/work-packages/<work-package>.md` | 当前增量的目标、范围、任务与验收；不得覆盖 canonical design |
| Design decision history | [docs/adr/README.md](docs/adr/README.md) 与其目录中的 ADR | 记录为什么改变设计；被接受的决策必须同步进 canonical design |

当前任务从 implementation-status 的 Active Work Package 读取，不能从文件名排序、最近提交或对话猜测。第一个工作包是 [P1-01](docs/work-packages/P1-01.md)。后续包在接近实施时才按 DDD §15 细化。

代码是实现现状的证据，MUST NOT 反向成为设计的事实来源。发现代码、任务或文档冲突时，MUST 先记录设计章节、代码位置、实际行为、受影响契约／不变量及验证证据，并分类：

| 分类 | 处理 |
|---|---|
| implementation bug | 在当前授权范围内修复实现，补足行为验证；不通过改设计使错误合法化 |
| documentation drift | 根据已确认设计和决策核对陈旧文档；不能仅因代码不同就判定 canonical design 过期 |
| contract change | 按 Level 2 处理，识别调用方及兼容性影响 |
| domain model change | 按 Level 3 处理 |
| architecture change | 按 Level 4 处理 |

无法分类或涉及未确定的契约时，MUST 报告缺口，暂停依赖该决定的实现；可继续不受影响的任务。MUST NOT 静默选择一种解释、加入临时旁路或修改设计后继续。

## Coding Agent Workflow

每次收到实现任务，MUST 按以下顺序完成 Design → Task → Code → Test → Design Sync：

1. **确认 Work Package**：核对用户任务与 Active Work Package，确认其依赖和授权范围。没有对应任务说明时，先从 DDD 派生该包的说明，不自动展开所有后续包。
2. **阅读上下文**：本文件、对应 work-package、DDD 相关章节及 implementation-status；产品语义涉及教学时再读取相关 prompt 与架构依据。
3. **检查代码事实**：查看 repository tree、Git 状态、现有实现、脚本、迁移及测试。MUST NOT 根据设计目录推断代码已存在；保留用户已有改动。
4. **列出影响面**：明确 relevant invariant IDs、bounded contexts、aggregates、application use cases、public contracts、database schema、APIs、required tests；不涉及的项写明原因。
5. **制定最小实现计划**：列出本次子任务、文件范围、依赖、验证命令与验收映射；标出已知缺口。新增依赖须说明用途和现有能力为何不足。
6. **实现当前增量**：遵守边界与变更协议，同步补充日志、注释、契约映射及必要迁移，不扩展到未来工作包。
7. **运行验证**：按当前工作包与 DDD §14 执行要求的层次；记录实际命令、结果、覆盖项与未运行原因。失败、跳过、环境阻塞均不能描述为通过。
8. **同步文档与状态**：核对 Design → Task → Code → Test 的一致性；必要时按协议同步 canonical design／ADR、contracts、任务与运行说明；最后更新 implementation-status 的日期、状态、代码及验证证据、剩余缺口和下一项任务。

未完成的任务也 MUST 更新真实进展。交接信息至少应能定位到包内任务 ID、实现文件、测试、最近结果及下一步，不能只写“已完成大部分”。

## Architecture Constraints

以下规则从 DDD §3–§4、§6–§10 提炼；完整业务语义仍以对应设计章节为准。

### Modules and layers

- MUST 保持模块化单体：一个应用部署单元、一个本地 SQLite 数据库；MUST NOT 自行拆分微服务、增加第二套持久化事实来源或消息交付架构。
- MUST 保持 Library、Reading、Tutoring、Learning、Review、Identity 的 bounded context 边界。Runtime 是技术支撑模块，MUST NOT 将其当成 Tutor 领域或把 Library 导入任务伪装成 AgentRun。
- MUST 按 `domain / application / infrastructure / interfaces` 分层。Domain 只依赖本域模型及必要的 `shared-kernel`；MUST NOT 依赖 Fastify、Drizzle、Zod、AI SDK、HTTP DTO、`contracts` 或其他域的聚合实现。
- Application MUST 定义仓储和外部能力 ports，由应用服务加载聚合、执行业务方法、保存结果；infrastructure MUST 实现这些 ports，MUST NOT 让应用用例依赖具体适配器。
- Interfaces MUST 负责 HTTP 输入校验、DTO 映射和调用应用用例；MUST NOT 把领域规则隐藏在 controller、路由 hook 或 ORM 代码中。
- 跨模块导入 MUST 仅通过目标模块的 `public.ts`；MUST NOT 深层导入其他模块的 infrastructure、Drizzle tables 或私有实现，也不能通过 public.ts 转导出这些内部类型来绕过限制。跨域查询使用公开查询 ports／快照，不共享可变聚合实例。
- 跨域 orchestration MUST 放在 `workflows`。跨域写入 MUST 调用相应域的公开应用操作，MUST NOT 直接写其他域的表。无实际跨域用例时，不创建 workflow 空壳。
- 模块路由 MUST 使用 Fastify 插件及局部 hook 注册；进程级数据库、日志与时钟 MUST 在 bootstrap 装配注入。框架插件封装不能替代代码依赖检查。
- MUST 使用 ESLint 导入限制和 TypeScript 工程边界检查上述方向；MUST NOT 通过扩大 allowlist 或关闭规则使越界代码通过。

### Contracts and frontend

- 前端对服务端的依赖 MUST 仅经 `packages/contracts` 的公开 DTO、枚举和 schema；MUST NOT 导入服务端 domain、application、infrastructure 或数据库模型。
- Domain、持久化和 HTTP 表示 MUST 在边界显式映射。领域枚举与公共枚举 MUST 分开并验证全部成员的映射覆盖；MUST NOT 靠强制类型断言代替映射。
- `contracts` MUST NOT 包含私有答案、评分依据、模型消息、任务输入快照或凭据类型（DDD §4）。认证请求的具体共享范围存在待澄清项时，按 implementation-status 中的缺口处理，不能自行放宽此规则。
- 对外序列化 MUST 使用公开字段白名单；MUST NOT 先序列化聚合／数据库行，再删除敏感字段。新增公开字段、枚举协议值及错误码须执行变更协议。
- 用户身份 MUST 来自服务端会话，所有用户资源读取和写入 MUST 校验归属；MUST NOT 将客户端 userId 作为可信身份。前端不得自行维护第二份持久化业务事实库。

### Transactions and persistence

- MUST 遵守 DDD §6.7 的原子提交清单。workflows 使用同一事务下各域的应用操作，各域操作内部使用自己的仓储；具体事务句柄类型 MUST NOT 泄漏到 domain。
- `better-sqlite3` UnitOfWork MUST 使用同步短事务回调，MUST NOT 跨越 `await`，也不能在事务中等待模型、文件解析或其他外部 I/O。外部工作完成后在事务内重新检查版本、状态和租约。
- SQLite MUST 使用本地磁盘、单写入连接及 DDD §7.1 的 PRAGMA：所有连接启用外键；写入连接使用 WAL、FULL、5000ms busy timeout。
- Schema、唯一约束、CHECK、外键和索引 MUST 由版本化迁移建立；MUST NOT 用生产自动 schema push 或应用层“先查后写”替代设计要求的数据库约束。
- UUID、UTC 毫秒、API ISO 8601 时间、稳定枚举值和带 schemaVersion 的 JSON MUST 遵守 DDD §7.1；JSON 读取也须校验。归属检查不能简化为“外键存在”。
- MUST 明确文件与数据库不能共用一个事务，按对应工作包执行 DDD 的文件落地、引用提交和恢复规则；不能伪造跨资源原子性。

### Runtime and external execution

- 实施 Runtime 时 MUST 以 DDD §7.3、§9.5、§10 为依据，持久化任务、租约、调用预算、事件和恢复状态；MUST NOT 用进程内通知承担唯一业务交付责任。
- MUST 保持 AgentRun 终态、执行批次、调用预扣、重试／修复预算和 deadline 的设计语义；AI SDK 的隐藏自动重试 MUST 关闭，恢复不能重置已用预算。
- 模型调用 MUST 在事务外执行；发布时 MUST 重新校验执行资格。SSE MUST 只发送公开状态及结果定位，MUST NOT 发送未经校验的模型 delta 或私有材料。
- 上述是未来实现的约束，MUST NOT 因为本文件提到 Runtime 就在 P1-01 创建它。

## Business Invariants

INV-01 ～ INV-13 的唯一完整定义在 [DDD §2.2](docs/ddd-development-design.md#22-业务不变量)。本文件不复制或重新解释这些定义。

任何触及不变量的实现 MUST 在计划、任务及测试中指出对应 INV ID，提供能验证相关行为的测试，并在交付证据中建立关联。MUST NOT 为简化代码、满足工期或让测试通过而弱化不变量。只验证了当前增量时，MUST 标注覆盖边界，不能声称整个不变量已被全系统验证。

## Scope Control

原则：**implement the smallest correct increment**。

- MUST 只实现当前 Work Package 及其明确必要、设计允许的依赖；必要性须写入最小计划。依赖未知时先核对设计，不能把“以后会用”当作当前依赖。
- MUST NOT 顺手实现未来包、创建未来领域的空壳或提前冻结其 schema／接口。
- P1-01 MUST NOT 实现 Library、Book 导入、EPUB／TXT／PDF parser、Reading、Tutor／Tutoring、Agent Runtime、Learning 或 Review。登录所需 Identity 与工程设施以 P1-01 文档为边界。
- 设计要求未来能力参与当前用例而阶段依赖不清时，MUST 记录缺口并澄清，MUST NOT 用返回空成功的 stub 掩盖业务规则缺失。

## Test Policy

遵循 [DDD §14](docs/ddd-development-design.md#14-测试与验收)，按行为所需层次验证：

| 类型 | 工具／真实边界 | 验证责任 |
|---|---|---|
| Domain unit tests | Vitest、可控时钟 | 领域规则、状态转换和边界情况；不依赖 Web 框架或真实模型 |
| SQLite integration tests | 真实临时 SQLite 文件、正式迁移 | 事务回滚、唯一性、归属、竞争及恢复；MUST NOT 用 repository mock 或仅内存实现替代验收 |
| HTTP integration tests | Fastify injection 验证普通 HTTP；SSE、提供商流式适配使用真实本地 HTTP 连接 | schema、鉴权、DTO、错误、响应头与流协议；MUST NOT 仅测试 application service，也不得以 injection 替代设计明确要求的真实连接 |
| Model contract tests | 可控提供商模拟服务；实际模型另做质量／能力验收 | 分块、429、超时、修复、工具与取消；模拟通过不等于真实提供商验收通过 |
| Playwright E2E | 两个独立浏览器上下文模拟同一用户的设备 | 浏览器 Cookie、登录恢复及对应包已经存在的业务流程；不提前要求尚未实现的全流程 |

每个包 MUST 将 Acceptance Criteria 映射到 DDD 章节、适用的 Txx 及实际测试。工程构建、边界或安全要求没有独立 Txx 时，引用原章节和包内 AC ID，MUST NOT 编造 T26 或改变 T01 ～ T25 的语义。跨阶段 Txx 必须列清已覆盖和待覆盖部分。

验证证据 MUST 包含日期、代码提交／工作树范围、实际命令、结果及未覆盖项。不能把未运行写成通过，不能用文档检查、健康接口或某个子测试替代完整验收。

## Definition of Done

任务标记为完成或工作包标记为 VERIFIED 前，MUST 满足：

- 当前范围的 design requirements 已实现，未解决的设计依赖不影响本次验收。
- 相关不变量得到保留，AC、INV 与测试证据可追踪。
- build、typecheck、lint、architecture boundary checks 全部通过。
- 当前工作包要求的单元、真实 SQLite、HTTP、模型契约及 E2E 验证按适用范围通过。
- 所需 migrations 已包含且验证；contracts、枚举映射、错误契约及调用方保持同步。
- 文档、工作包、必要的 canonical design／ADR 已同步，implementation-status 已更新。
- 没有把跳过、环境阻塞或部分 Txx 覆盖描述为完整通过。

不涉及 migration／某类测试时，MUST 写明“不适用”及设计依据；不能把缺失实现写成不适用。代码完成但验收未齐只能记录 IMPLEMENTED；“代码已经写完”不能算 Done。

## Design Change Protocol

不可行或冲突的设计 MUST 先说明证据与影响，不得 silent workaround。按影响最高的级别处理：

| Level | 变化 | 实施前要求 |
|---|---|---|
| 1 | Implementation detail：不改变公开契约、领域语义或架构 | 在当前授权内记录选择并测试，无需 ADR |
| 2 | Public contract change：公开 DTO、字段、枚举、API、错误或兼容性变化 | 先明确变化及调用方影响，同步 canonical design、contracts、任务和测试；影响兼容性、授权／隐私边界或重要协议语义时还 MUST 有 ADR |
| 3 | Domain model change：聚合边界、领域语义、业务不变量或事务语义变化 | MUST 先有被接受的 ADR 并修改 canonical design，确认决策后才实现受影响部分 |
| 4 | Architecture change：模块边界、依赖方向、部署／存储／执行模型变化 | MUST 先有被接受的 ADR 并修改 canonical design，确认决策后才继续受影响实现 |

ADR 位于 `docs/adr/`，格式与流程见 [ADR README](docs/adr/README.md)。任务授权不自动等于改变既有设计的授权；已有明确授权则按该决策继续，无需重复请求。提案、未决问题及代码中的 TODO 不能替代已接受的设计。

## Coding Principles

- MUST 遵循 YAGNI，避免 speculative abstraction；优先显式领域规则。
- MUST NOT 在真实重复和用例尚未证明需要前创建通用 BaseService、BaseRepository 或框架抽象；确有必要时在计划中给出依据。
- MUST 保持事务边界显式，防止 infrastructure 类型泄漏到 domain；不把领域决策藏在 controller 或 ORM 中。
- 新增库 MUST 有当前用例依据、边界说明及兼容验证；不能仅因便利而引入。版本与锁文件随工程提交，不机械复制文档中的外部示例版本。
- 复杂逻辑 MUST 注释意图、非显然的决定和关键边界；不得只复述语句。
- 数据类、接口、关键值对象 MUST 使用 TypeScript 的 JSDoc（对应数据说明要求），说明用途、单位、可空含义、版本及所有权；DTO、持久化实体与领域对象分别说明。
- 新代码中的有限状态、动作、类型和错误码 MUST 使用枚举；序列化和数据库边界 MUST 显式保存稳定的字符串协议值并测试映射。
- 关键处理阶段与失败分支 MUST 提供可操作的结构化日志，含适用的 requestId、实体 ID、状态、阶段、耗时和安全错误码。MUST 遵守 DDD §13.1 的过滤规则，禁止记录凭据、令牌、敏感载荷或直接展开外部异常；SQL 参数调试默认关闭。
