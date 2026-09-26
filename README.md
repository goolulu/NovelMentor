# NovelMentor

NovelMentor 是面向个人跨设备使用的英文小说阅读导师：计划支持 EPUB、TXT、文字型 PDF 的连续阅读，并按阅读位置提供中文翻译、词句讲解、学习记录和复习。

**项目目前处于设计阶段，尚无可运行应用。** 当前仓库提供设计与 AI 开发治理文档；工程、依赖、迁移和测试将在 P1-01 开始建立。

设计采用模块化单体，一个应用部署单元和一个本地 SQLite 数据库。Library、Reading、Tutoring、Learning、Review、Identity 分离建模，Runtime 提供有界生成任务的技术支撑；一个 Tutor 按任务加载教学指令。

## Planned Stack

- 工程：pnpm workspace、TypeScript；后端：Node.js 24、Fastify。
- 前端：React、Vite、TanStack Query、React Router。
- 数据与边界：SQLite、better-sqlite3、Drizzle、服务器文件、Zod；模型接入使用服务端 AI SDK。
- 验证：Vitest、真实 SQLite／HTTP 集成测试、模型契约测试、Playwright。

上述均为设计选择，不能据此推断依赖已安装或对应能力已实现。

## Repository and Development Entry

| 入口 | 职责 |
|---|---|
| [AGENTS.md](AGENTS.md) | Coding Agent 的工作方式、工程边界及设计变更流程 |
| [DDD 开发设计](docs/ddd-development-design.md) | Canonical design：系统应该是什么 |
| [Agent 架构背景](docs/agent-architecture-plan.md) | 架构意图与依据 |
| [Tutor 产品需求](prompt.txt) | 教学行为输入 |
| [Implementation Status](docs/implementation-status.md) | 当前真实文件树、实现程度、验证证据与缺口 |
| [P1-01](docs/work-packages/P1-01.md) | 当前唯一展开的工作包：工程与账号 |
| [ADR 规范](docs/adr/README.md) | 重要设计变化的原因与决策记录 |

开始开发前读取 AGENTS、当前工作包、相关 DDD 章节和 implementation-status，并检查代码现状。尚无可执行的安装／构建／测试脚本；P1-01 将建立命令并记录实测结果。不要按设计中的最终目录提前初始化所有业务模块。
