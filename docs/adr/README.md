# Architecture Decision Records

ADR 记录重要设计决定的背景、选择及后果，回答“为什么改变设计”。[DDD](../ddd-development-design.md) 仍是当前 canonical development specification；ADR 不取代设计正文，也不是实现状态、任务清单或普通代码提交日志。

目前没有实际 ADR。本次只建立记录规范。

## When an ADR is Required

按 [AGENTS.md 的 Design Change Protocol](../../AGENTS.md#design-change-protocol) 执行：

| 变化级别 | 要求 |
|---|---|
| Level 1 — Implementation detail | 不改变契约、领域或架构的普通实现选择通常无需 ADR，例如局部函数拆分或等价实现 |
| Level 2 — Public contract change | 必须同步 canonical design、contracts、调用方及测试；兼容性、授权／隐私边界或重要协议语义变化必须有 ADR，普通兼容性补充可只同步设计 |
| Level 3 — Domain model change | 聚合边界、领域语义、不变量、事务语义等变化必须有 ADR，并先更新 canonical design |
| Level 4 — Architecture change | bounded context、依赖方向、存储、部署、运行执行机制等变化必须有 ADR，并先更新 canonical design |

同时影响多类时按最高级别处理。不能将行为变化命名为“重构”以规避记录，也不必为每个函数、测试或依赖补丁版本建立 ADR。重要依赖选择若改变上述边界，则仍按实际影响分类。

## File Naming and Lifecycle

- 文件名使用四位递增编号及简短英文标题，例如 `0001-short-title.md`；标题使用 `ADR-0001`。编号不得复用。
- Status 使用 `PROPOSED`、`ACCEPTED`、`REJECTED`、`SUPERSEDED`，Date 使用 `YYYY-MM-DD`。
- 提案需说明原设计章节、发现的证据、替代方案、契约／不变量／迁移与测试影响；只提出问题而未决定时保持 PROPOSED。
- Level 3／4 以及需要 ADR 的 Level 2，在项目负责人已确认决定、ADR 为 ACCEPTED 且 canonical design 已同步后，才能实现受影响部分。已有明确决定无需重复确认。
- 接受时同步 work-package、相关 contracts／测试与 implementation-status；后者记录实际实现和验证进度，不能因 ADR 被接受就写 VERIFIED。
- 被替代的记录保留，标为 SUPERSEDED 并链接新 ADR；不改写历史使旧决定看起来从未发生。

## Template

```markdown
# ADR-XXXX: Title

Status: PROPOSED
Date: YYYY-MM-DD

## Context

当前设计、发现的问题与证据；受影响工作包、领域、契约和不变量。

## Decision

选择什么、适用边界、兼容性及迁移方式；接受后记录确认依据。

## Alternatives Considered

实际考虑的替代方案，包括保持原设计，以及未选择的原因。

## Consequences

收益、代价、风险、运维影响、回退约束和需要补充的验证。

## Affected Design Sections

canonical design 的准确章节／链接，以及需同步的工作包、契约和测试。
```
