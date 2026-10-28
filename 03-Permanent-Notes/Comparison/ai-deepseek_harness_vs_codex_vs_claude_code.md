---
type: permanent
subtype: comparison
status: 🟢成熟
created: "2026-10-19"
updated: "2026-10-19"
verified: "2026-10-19"
tags: [AI, agent, harness, coding-agent, software-engineering]
domain: [artificial-intelligence, software-engineering]
compared-entities: ["[[DeepSeek Harness]]", "[[Codex]]", "[[Claude Code]]"]
comparison-type: mixed
source-literature:
  - "https://blog.csdn.net/aidoudoulong/article/details/163759887"
  - "https://deepseek.com/harness/"
  - "https://developers.openai.com/blog/codex-as-a-platform"
  - "https://code.claude.com/docs/en/overview"
---

# DeepSeek Harness vs Codex vs Claude Code：AI 编程 Harness 对比

> **比较问题**：选择现成编程助手，还是构建可定制的 Agent 运行时？
> **比较维度**：架构与扩展、模型兼容、安全边界、会话管理、成本与适用场景。
> **资料范围**：用户提供的博客正文作为比较线索，以 **2026-10-19** 访问的官方文档核查、修正；不将当前资料回推为博客标注的 2026-08-14 历史状态。
> **证据边界**：本文是文档比较，未进行同任务实测。官方文档可证明功能设计与支持范围，不能单独证明性能领先或绝对安全。

## 1. 比较概览

**Harness 是围绕模型的执行系统**，负责组织上下文、调用工具、推进任务、处理状态与权限。“Model + Harness = Agent”是理解分工的工程简写，并非性能公式。实际效果还受模型、工具、运行环境与任务约束共同影响。[^codex-platform]

**一句话结论**：DeepSeek Harness 突出运行时能力的插件化重组；Codex 将可复用的开放 Harness 与多端开发工作流结合；Claude Code 提供围绕 Claude 的多端编程环境、生命周期钩子与 Agent SDK。三者的差异是扩展接口、生态适配与部署边界的不同，不能简化为“开源框架 / 安全工具 / 闭源产品”三个互斥类别。[^dsh-architecture][^codex-platform][^claude-overview][^claude-sdk]

## 2. 多维度对比

| 维度 | DeepSeek Harness | Codex | Claude Code |
|---|---|---|---|
| 核心组织方式 | Cordis 插件树；模型、工具、会话、循环和 UI 等能力可替换 | 可复用的 Agent 执行循环；提供 CLI、SDK 和 app-server 集成入口 | 多端共享 Claude Code 引擎；以 hooks、skills、MCP 和 SDK 扩展 |
| 开放性与许可 | MIT 开源；仍为预览软件 | Harness、CLI、app-server、官方 SDK 开源；模型与托管服务另计 | 商业产品；Agent SDK 使用受商业条款约束，部分组件有独立许可 |
| 使用入口 | 当前已有 macOS、Windows 桌面端；可用命令启动本地 Web UI | 桌面端、CLI、IDE 扩展、Cloud 等入口 | 终端、桌面端、Web、VS Code、JetBrains 等入口 |
| 模型接入 | 提供多厂商目录与适配机制，支持自定义提供方 | 可配置模型提供方；需满足对应协议和能力要求 | 官方支持以 Claude 模型为中心；支持云平台与网关接入 |
| 定制深度 | 可重组运行时内部能力，包括 Agent 循环与会话服务 | 通过 SDK、app-server、MCP 等集成现有 Harness | 通过事件钩子、插件和 SDK 定制工作流程 |
| 会话与状态 | 追加式事件日志是会话事实源，可由事件重建模型消息历史 | SDK 支持继续与恢复线程；app-server 提供历史、事件和审批交互 | SDK 支持维持、恢复和分叉会话 |
| 主要代价 | 运行时与插件维护成本；API 可能发生不兼容变更 | 自定义提供方存在兼容性要求；托管能力受服务与账户条件约束 | 非 Claude 模型不在官方支持范围；复杂钩子需要维护 |
| 更适合的起点¹ | 运行时研发、插件实验、多模型接入 | 开发任务执行、代码审查、将现有 Harness 嵌入内部工具 | Claude 生态开发、事件驱动流程、团队规范自动化 |

表中依据：DeepSeek 官方主页、仓库与架构文档；Codex 平台、SDK、配置文档；Claude Code 总览、SDK、网关文档。[^dsh-home][^dsh-repo][^dsh-architecture][^dsh-models][^codex-platform][^codex-sdk][^codex-config][^claude-overview][^claude-sdk][^claude-gateway]

¹ “更适合”是根据接口与维护成本作出的选型推论，不代表经同条件测试得到的优劣排名。

### 安全控制：分清三个层次

| 控制层 | 解决的问题 | 比较要点 |
|---|---|---|
| 流程与 hooks | 某个生命周期节点应执行什么检查？ | Claude Code 的 `PreToolUse` 等事件可介入工具调用；钩子数量不等于安全强度。[^claude-hooks] |
| 权限与审批 | 哪些操作自动执行，哪些需要授权？ | 与操作系统沙箱相互配合；审批放宽会改变实际风险 |
| 执行隔离 | 子进程实际能访问哪些文件、网络与系统资源？ | 比较生效范围、默认策略、例外和平台实现，不能仅看产品名称 |

具体差异：

- **Codex**：本地命令使用平台原生隔离；当前文档列出 macOS Seatbelt、Linux/WSL2 的 bubblewrap，以及原生 Windows 沙箱。沙箱规定技术边界，审批策略决定何时请求越界授权。[^codex-sandbox]
- **Claude Code**：同样提供操作系统级命令沙箱，macOS 使用 Seatbelt，Linux/WSL2 使用 bubblewrap；当前原生 Windows 不提供该命令沙箱。其覆盖对象主要是 shell 命令及其子进程，文件工具、hooks、MCP 服务等不在同一沙箱范围内。[^claude-sandbox]
- **DeepSeek Harness**：沙箱与审批属于可组合的运行时能力。官方安全声明明确：项目仍属实验性预览，未经安全审计，不应视为安全或生产就绪，也不能作为不可信负载的唯一安全控制。[^dsh-safety]

**判断原则**：操作系统隔离能约束被覆盖进程的资源访问，但无法保护已被允许修改的文件，也不能据此宣称“不可绕过”或“绝对安全”。对不可信仓库，应核对实际启用的隔离、网络、凭据、插件及越界授权配置；必要时使用独立容器或虚拟机。此处是部署建议，不是三者的安全认证排名。[^codex-sandbox][^claude-sandbox][^dsh-safety]

## 3. 差异本质与容易混淆之处

### 3.1 扩展工作流程，与重组运行时

DeepSeek Harness 的 Cordis 插件可贡献服务、事件与可撤销的注册效果，模型适配器、工具注册表、会话日志及 Agent 循环均可在配置层替换。其区别在于**替换内部能力的组合方式**。[^dsh-architecture]

Codex 并非不可二次开发：可以使用 `codex exec` 执行非交互任务，通过 SDK 控制 Agent，或使用 app-server 构建自有客户端。Claude Code 也通过 Agent SDK 提供工具、权限、hooks、子 Agent 与会话能力；将 SDK 名称差异写成“没有编程接口”会误导选型。[^codex-platform][^codex-sdk][^claude-sdk]

### 3.2 支持多模型，不等于任意端点即插即用

DeepSeek Harness 的模型目录包含 OpenAI、Anthropic、Kimi、GLM 等提供方，接入仍需相应凭据和适配。Codex 的自定义提供方需符合接口要求，当前配置参考中 `wire_api` 仅支持 `responses`；仅提供 Chat Completions 的“OpenAI 兼容”端点不自动等价。Claude Code 官方明确不支持通过网关路由到非 Claude 模型。[^dsh-models][^codex-config][^claude-gateway]

因此，模型选择应依次确认：**官方支持范围 → 协议兼容 → 工具调用与上下文能力 → 实际任务质量**。统一网关可以简化端点管理，但不会自动免除上游授权、计费或适配要求。

### 3.3 日志可回放，不等于重新执行结果一致

DeepSeek Harness 采用事件溯源：模型消息由追加式日志推导，回放可重建历史上下文。这有利于轨迹检查和调试；Codex 与 Claude 的 SDK 也提供线程或会话恢复，因此不能把后二者概括为“没有状态记录”。[^dsh-session][^codex-sdk][^claude-sdk]

需区分 **查看历史轨迹、重建模型输入、重新执行任务**。最后一项还依赖模型版本、采样、代码与依赖、外部服务及工具副作用；单靠会话日志不能保证确定性复现。这是可复现性的工程边界，而非某款产品独有的缺陷。

### 3.4 Harness 成本，与完成任务的总成本

| 成本口径 | DeepSeek Harness | Codex | Claude Code |
|---|---|---|---|
| 软件与服务 | 开源软件不收许可证费；推理与部署另计 | 开源组件与付费模型、托管服务需分开计算 | 按所选订阅、API 或云平台路径计费 |
| 推理支出 | 由所选模型、token、缓存和轮次决定 | 订阅额度或 API 用量；超额与附加功能规则依账户而定 | 订阅包含用量或按 token 计费；可另购用量额度 |
| 工程支出 | 插件开发、兼容升级、部署维护 | 接入、任务审查、自动化与环境维护 | hooks、规范配置、审查与流程维护 |

计费结构依据官方成本与价格说明；这里只比较口径，不将不同产品的额度直接换算为等量任务。[^dsh-repo][^codex-pricing][^claude-costs]

选型更有意义的指标是：**每个验收通过任务的成本 =（分摊订阅或 API 支出 + 执行环境成本 + 人工审查与返工成本）÷ 验收通过任务数**。这是评估口径，不是厂商计费公式。低 token 单价不必然意味着较低的任务总成本。

## 4. 对原博客的必要修正

以下比较针对用户粘贴的正文；原网页此前未能完整抓取。[^blog]

| 博客表述 | 本笔记的处理 |
|---|---|
| Codex“核心代码不开源” | 修正：官方明确 Harness、CLI、app-server、SDK 为开放组件；不据此推断模型权重和托管服务开源。[^codex-platform] |
| Claude Code“仅应用层沙箱”，Codex“内核沙箱不可绕过” | 修正：两者均有 OS 级隔离机制；应比较覆盖范围、启用条件与授权例外，删除绝对安全判断。[^codex-sandbox][^claude-sandbox] |
| DeepSeek Harness“没有桌面端” | 时效更新：当前官方已提供 macOS、Windows 桌面下载；不据此判定其最初发布时也已具备。[^dsh-home] |
| Claude Code 没有 Python SDK、Codex 没有 SDK | 当前 Claude Agent SDK 与 Codex SDK 均有 Python、TypeScript 接口；更应比较接口能力。[^claude-sdk][^codex-sdk] |
| Codex 可直接调用“任意 OpenAI 兼容模型” | 收窄：需核对协议、认证、工具与能力兼容；当前配置要求 Responses 协议。[^codex-config] |
| “¥0.2/任务”“便宜 57 倍”“Opus 5 接近 96%” | 不采作比较结论：原文未给出足以复核的任务、token、模型快照、评测设置与原始证据。 |
| “26 个钩子”“6 条并行线程”“40 家厂商”“300 个插件” | 不作为固定产品属性：计数可能受版本、设置与口径影响；选型以所需事件、并发和提供方是否受支持为准。 |
| 固定默认模型、上下文上限、月费及发布时间表 | 不继承为当前事实：上下文与价格取决于模型、认证路径、方案和日期。当前文档不能完整验证原文的历史时间线。 |

**证据不足不等于已证伪**：未获充分核实的数值予以排除；已与当前官方资料不符的内容标明修正或时效更新。

## 5. 决策框架

下表是基于文档的条件性建议；实际采用前宜在自有仓库验证。

| 需求或约束 | 优先评估 | 理由与边界 |
|---|---|---|
| 研究 Agent 循环、会话机制，或重组运行时 | DeepSeek Harness | 插件化替换范围明确；须接受预览 API 与维护负担 |
| 将现有编码 Agent 嵌入内部应用或 CI | Codex / Claude Agent SDK | 依据模型生态、会话接口、审批需求和许可条件选择 |
| 在 Claude 生态落实工具前检查与团队流程 | Claude Code | hooks 与 SDK 可承载流程规则；规则仍需测试和维护 |
| 多模型路由、国产模型接入是核心需求 | DeepSeek Harness；亦检查 Codex 提供方兼容性 | 多厂商目录较直接；不能仅按适配器数量选择 |
| 审查不可信 PR 或执行无人值守任务 | 先确定隔离部署，再选工具 | Codex 与 Claude 都有 OS 级沙箱；没有本文证据支持“某款绝对最安全” |
| 降低批处理预算 | 在同一批任务上比较模型与 Harness 组合 | 记录通过率、成本、耗时、人工返工；低价模型只是候选 |
| 重视图形界面、并行任务与审查体验 | 实测三者当前桌面入口 | DeepSeek 也已有桌面版；不能沿用旧形态表排除 |

**组合使用**：可在同一仓库使用多款工具，但应将不同任务放在独立分支或 worktree，避免同时改写同一文件。指令文件可以并存，仍需统一项目约束并检查各工具的读取规则；文件名不同不代表语义、权限或执行过程天然无冲突。

**最小评估方法**：选取修复缺陷、实现功能、重构与审查等代表任务；固定仓库提交、验收测试、权限和预算，记录具体模型与 Harness 版本，进行重复运行。跨模型比较测得的是“模型 + Harness + 配置”的整体效果；若要归因于 Harness，应尽可能固定模型并披露无法保持一致的条件。

## 6. 知识网络与资料出处

- [[DeepSeek Harness]]、[[Codex]]、[[Claude Code]]：可拆分为独立产品笔记。
- [[ai-agent_vs_script|Agent 代理执行 vs 脚本自动化执行]]：解释 Agent 如何组织底层脚本；本笔记继续比较 Agent 执行系统的实现方式。
- 可延伸概念：[[Harness Engineering]]、[[操作系统沙箱]]、[[事件溯源]]、[[AI Agent 可复现性]]。

以下官方资料均访问于 **2026-10-19**。动态文档与 `master` 分支可能继续变化；用于正式研究或复现实验时，应另存文档快照及 commit。博客是二手比较材料，不作为争议技术结论的最终依据。

[^blog]: 用户提供正文：[DeepSeek Harness vs Codex vs Claude Code：三款 AI 编程 Harness 深度对比](https://blog.csdn.net/aidoudoulong/article/details/163759887)，文内标注信息截至 2026-08-14。本文依据粘贴文本分析，未宣称已完整读取原网页。
[^dsh-home]: DeepSeek：[DeepSeek Harness 官方主页](https://deepseek.com/harness/)。用于当前桌面入口与预览状态；功能展示不等于独立性能评测。
[^dsh-repo]: DeepSeek-AI：[deepseek-harness 官方仓库](https://github.com/deepseek-ai/deepseek-harness)。用于 MIT 许可、Web 启动方式及兼容性变更声明。
[^dsh-architecture]: DeepSeek-AI：[DeepSeek Harness Architecture](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)。用于 Cordis 服务、事件、插件树、profiles 与 bundles。
[^dsh-models]: DeepSeek-AI：[Configure models](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/providers.md)。用于提供方目录、模型配置和凭据要求。
[^dsh-session]: DeepSeek Harness：[Sessions](https://deepseek-harness.github.io/deepseek-harness/en/reference/subsystems/session)。用于追加式事件日志、模型消息推导与回放语义。
[^dsh-safety]: DeepSeek-AI：[Safety](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)。用于实验状态、未经安全审计及隔离边界声明。
[^codex-platform]: OpenAI：[Codex as a platform: build on the open agent harness](https://developers.openai.com/blog/codex-as-a-platform)，2026-08-19。用于 Harness 定义、开源范围和集成层次。
[^codex-sdk]: OpenAI：[Codex SDK](https://learn.chatgpt.com/docs/codex-sdk)。用于 Python / TypeScript 接口、线程恢复及 app-server 关系。
[^codex-config]: OpenAI：[Configuration Reference](https://learn.chatgpt.com/docs/config-file/config-reference)。用于 TOML、自定义提供方及 `wire_api = responses` 的协议约束。
[^codex-sandbox]: OpenAI：[Sandbox](https://learn.chatgpt.com/docs/sandboxing)。用于跨平台命令隔离、默认边界与审批的关系；不提供绝对安全保证。
[^codex-pricing]: OpenAI：[Pricing](https://learn.chatgpt.com/docs/pricing)。用于订阅用量、附加 credits 与账户条件；本文不复刻动态价格表。
[^claude-overview]: Anthropic：[Claude Code Overview](https://code.claude.com/docs/en/overview)。用于多端入口、共享引擎、并行 Agent 与扩展能力。
[^claude-sdk]: Anthropic：[Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)。用于 Python / TypeScript SDK、会话、权限与商业条款。
[^claude-hooks]: Anthropic：[Hooks reference](https://code.claude.com/docs/en/hooks)。用于生命周期事件和工具调用前的控制能力。
[^claude-sandbox]: Anthropic：[Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing)。用于 OS 级隔离、平台差异、覆盖范围及沙箱外运行的例外。
[^claude-gateway]: Anthropic：[Other LLM gateways](https://code.claude.com/docs/en/llm-gateway)。用于网关接入和非 Claude 模型不获官方支持的边界。
[^claude-costs]: Anthropic：[Manage costs effectively](https://code.claude.com/docs/en/costs)。用于订阅、token 计费、用量额度和成本影响因素。

*创建：2026-10-19 | 修改与核查：2026-10-19*
