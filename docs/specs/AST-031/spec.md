---
schema_version: 4
template_version: 1
kind: system-spec
id: spec:AST-031
authority: draft
archive_reason: null
superseded_by: null
approved_by: null
approved_at: null
phase: proposed
owners: [josephfarina, cixzhang]
affects_architecture: [architecture:cli-surface]
affects_families: []
affects_contributing: []
affects_consumer_docs: [authoring]
---

# 运行时集成功能组合系统规范

## 意图

为运行时集成功能定义可复用的组合规则——集成可在其默认清单之外贡献的 CLI 能力。每个功能都是集成模块的一个具名导出，并可在 `astryx.config` 中有可选的对应字段。多个处理器以叠加方式组合：每个已配置的合格处理器独立运行，失败按处理器分别上报，功能定义一个聚合的命令结果，且不允许某个处理器压制其他处理器。

本规范先记录通用组合契约，再记录基于它构建的首个功能 `gapReport` 的特定要求。

## 非目标

- 替换默认导出清单或其贡献模型（组件、模板、codemod、文档）。这些由 `Project` 接缝加载，不在本范围内。
- 为任何处理器定义传输协议、网络目标或持久化格式。
- 提供带生命周期 Hook、依赖排序或处理器间通信的插件系统。

---

## 要求

### 通用组合规则

以下规则适用于遵循本规范的每个运行时集成功能。功能由 `astryx.integration.{ts,mjs,js}` 的具名导出标识，并可在 `astryx.config` 中有可选的对应字段。

#### 具名导出 + 匹配的项目配置字段

功能声明为集成模块的**具名导出**。同一功能也可作为项目级处理器出现在使用方的 `astryx.config` 字段中。不认识该具名导出的旧版 CLI 会忽略它，继续加载默认清单。

#### 叠加组合

有效集合中的每个处理器都会收到一个有序的投递结果。项目处理器排在最前，然后按集成在 `astryx.config` 中出现的顺序排列各集成处理器。合格处理器会运行；未获同意的 public 处理器记录 `consent_required`，但不被调用。任何处理器都不覆盖或取代其他处理器。

#### 按 handle 函数身份去重

当同一 `handle` 函数引用在有效集合中出现多次（例如项目配置重新导出了某集成的处理器），它只运行一次，位置在其首次出现处。

#### 每处理器 structuredClone 副本

每个处理器收到自己独立的 `structuredClone` 事件副本。修改输入的处理器无法影响其他处理器。

#### 失败隔离 + 聚合响应

抛出异常、超时或行为异常的处理器会产生该处理器的失败投递。后续处理器仍会运行。命令的聚合响应收集每处理器的投递记录，调用方可准确看到哪些处理器成功、哪些失败及原因。

#### 生命周期 / 异步

- **超时：** 每个处理器有 30 秒墙钟超时，并收到在该预算到期时触发的 `AbortSignal`。
- **异步：** 处理器可以是异步的；CLI 会逐个等待。
- **执行顺序：** 处理器按有效集合顺序串行运行。某处理器失败不会跳过后续处理器。

#### 每处理器同意元数据

每个处理器声明 `audience` 为 `'internal'` 或 `'public'`。CLI 按处理器执行同意：`public` 受众的处理器除非调用方明确确认了 public 写入，否则不会被调用。`internal` 处理器无需额外确认。

#### 可检查的同意（consent_required 投递）

当调用方未确认同意且处理器需要同意时，CLI 为该处理器记录 `consent_required` 投递，但不调用它。调用方可在带确认重跑前检查哪些处理器需要同意。

#### 回退行为

内置回退（例如通过 `issuesUrl` 打开 GitHub Issue）**仅**在去重后有效处理器集合为空时激活。只要存在任何已配置处理器——项目或集成——就完全抑制回退。

#### issuesUrl 是目标元数据

集成清单或项目配置上的 `issuesUrl` 描述人类可读 Issue 链接的目标。它是关于目标包的元数据，不是处理器实现。处理器将其作为上下文使用；CLI 将其用于回退路由。

#### 变更敏感测试

基于此组合模型构建的功能的测试必须覆盖扇出顺序、隔离、去重、超时、同意门控、consent-required 检查、回退抑制和聚合响应形状。移除任一 fail-closed 防护必须使测试失败。

---

### Gap-report 特定要求

`gapReport` 是基于上述通用组合规则构建的首个功能。

- **FR1 — 处理器类型。** gap-report 处理器是符合 `GapReportHandler` 的普通对象：`{ audience: 'internal' | 'public', handle: (event: GapReport, context: { signal: AbortSignal }) => GapReportHandlerReceipt | Promise<GapReportHandlerReceipt> }`。`GapReportHandler` 取代 `GapReportWriter`。

- **FR2 — 事件形状。** `GapReport` 事件携带 `schemaVersion: 1`、普通 camelCase 字段名、规范化值，以及含 `{ package, version, issuesUrl }` 的 `target` 对象。无 snake_case、无缩写、无嵌套调试上下文。

- **FR3 — 回执契约。** 处理器必须返回 `GapReportHandlerReceipt`：`{ status: 'filed' | 'routed_only' | 'skipped', url?: string, message?: string }`。不接受其他状态值。`url` 和 `message` 可选，携带人类可读上下文。

- **FR4 — 处理器隔离。** 每个处理器独立运行：
  - 每个处理器在自己的 worker 中运行。`stdout` 重定向到 `stderr`，`process.exit`、`process.exitCode` 和延迟的 promise 续体都被限制在该 worker 内。
  - 抛出异常或超时（30 秒）会产生该处理器的失败投递。CLI 中止该处理器的 signal，然后在启动下一个处理器前终止其 worker。
  - 失败的处理器不影响命令自身的退出码或其他处理器。

- **FR5 — 无继承处理器退出机制。** gap-report 处理器没有类似 `inheritDebug` 的退出机制。有效集合中的每个 internal 处理器都会运行，每个 public 处理器都会收到结果，仅受显式同意门控约束。
  `package.json` 中的 `inheritDebug` 字段是 `debug` 功能特有的兼容性例外（见决策日志 DEC-2）；它不为其他功能树立先例。

- **FR6 — 有效集合。** 有效处理器集合为：项目配置的 `gapReport` 处理器在前（若存在），然后按 `astryx.config` 中集成顺序排列各集成的 `gapReport` 具名导出。按 `handle` 函数身份去重。

- **FR7 — 目标选择。** gap report 的目标包按以下方式选择：显式 `--package` 标志 → 当只有一个包拥有该组件时取唯一组件所有者 → 默认 Core。目标选择不基于处理器数量或处理器来源。

- **FR8 — 内置 GitHub 回退。** 当有效处理器集合为空（去重后零处理器）且目标有 GitHub `issuesUrl` 时，CLI 提供通过 `gh issue create` 提交。此回退受同意门控（`public` 受众）。任何其他 `issuesUrl` scheme 产生 `routed_only` 回执。只要存在至少一个处理器，回退就被完全抑制。
  `astryx swizzle` 的回执在 `gh` 位于 PATH 时提供其 `gh issue create` 行，无论其登录状态。未登录的 `gh` 在该行运行时请求自行登录。文本回执显示命令；JSON 回执保留命令和 issues URL。

- **FR9 — 项目配置字段。** `astryx.config` 新增 `gapReport` 字段，接受 `GapReportHandler`。这是项目级处理器，在有效集合中最先运行。

- **FR10 — 集成具名导出。** 集成在其集成模块中以 `export const gapReport: GapReportHandler` 声明处理器。该具名导出刻意不是默认清单字段，与 `debug` 功能的导出约定一致。

- **FR11 — 每处理器同意。** 每个处理器的 `audience` 独立评估。混合受众的扇出会无条件调用 `internal` 处理器，并对 `public` 处理器按调用方同意门控。即使受门控的同级被阻止，未受门控的处理器仍会运行。

- **FR12 — 聚合响应。** 命令响应包含 `deliveries` 数组，每个有效处理器或回退对应一个条目。每个投递记录 `handlerType`、`handler`、`audience`、`status`、`url` 和 `message`。响应形状在 `--json` 和人类可读输出模式下相同。

- **FR13 — 调用方指定的目标。** 当调用方用 `--to` 指定目标时，该目标替换该报告的处理器和 FR8 回退。`architecture:cli-surface` INV36 管辖该选项及谁选择目标。

#### 平台支持

- 支持：Node >= 22.13，CLI 支持的任何 OS。
- 不支持：依赖网络访问、原生二进制或平台特定 API 的处理器由处理器作者负责；CLI 提供隔离但不提供能力检测。

---

## 当前状态影响

`debug` 已将每个集成具名导出与一个项目函数组合，按项目优先/配置顺序投递，带身份去重和每处理器副本。其同步进程退出生命周期和现有 `inheritDebug` 兼容开关保持不变。

PR #6200 最初选择一个可执行 gap writer。本规范在发布前替换该模型：`gapReport` 变为一个项目处理器加每个已加载集成处理器，每个处理器返回自己的回执。无处理器的现有使用方保留受同意门控的 `issuesUrl` 回退。由于单 writer 形状尚未发布，无需迁移已发布的配置或集成 API。

---

## 验证

| 契约 | 验证 | 代表性状态 | 变更或失败预期 |
| -------- | ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| FR1–FR3  | gap-report 处理器类型和回执契约测试 | 有效处理器；缺少 audience；错误状态值；非对象返回 | 无效处理器形状或回执被静默接受 |
| FR4      | 处理器隔离测试 | 处理器写 stdout；处理器调用 process.exit；处理器抛出异常；处理器超过 30 秒 | stdout 污染进入 JSON 信封，或退出/抛出异常停止后续处理器 |
| FR5      | 无退出机制断言测试 | 项目在 package.json 中设置 `inheritDebug: false`；gap-report 处理器仍运行 | 退出机制抑制 gap-report 处理器 |
| FR6      | 有效集合排序和去重测试 | 项目 + 两个集成；重复 handle 引用；仅项目；仅集成 | 顺序错误、重复执行或缺少处理器 |
| FR7      | 目标选择测试 | 显式包；唯一所有者；需要 `--package` 的歧义所有者；无组件匹配 | 处理器数量影响目标、歧义被猜测，或未知组件未能默认到 Core |
| FR8      | 回退激活和抑制测试 | 零处理器 + GitHub issuesUrl；零处理器 + 非 GitHub issuesUrl；存在一个处理器 | 存在处理器时回退触发，或非 GitHub issuesUrl 声称已提交 |
| FR9–FR10 | 配置字段和具名导出加载测试 | 项目配置含 gapReport；集成含具名导出；两者都存在；两者都不存在 | 配置字段被忽略、具名导出被忽略，或旧 CLI 在具名导出上出错 |
| FR11     | 混合受众同意测试 | 全 internal；全 public 且有同意；全 public 且无同意；混合且部分同意 | public 处理器未经同意运行，或 internal 处理器被同意阻止 |
| FR12     | 聚合响应形状测试 | 全部成功；一个失败；一个超时；consent_required 条目；空集合（回退） | 缺少投递条目、来源归属错误，或 --json 与人类可读响应形状不同 |
| FR13     | 具名目标测试 | `--to` 指定含 GitHub issuesUrl 的集成，有和没有同意；非 GitHub issuesUrl；未知包 | 处理器在 `--to` 下运行，或未知目标发送任何内容 |
| 通用  | 变更敏感组合测试（structuredClone） | 处理器修改事件字段；处理器向事件添加属性 | 变更泄漏到后续处理器 |

---

## 决策日志

### DEC-1 — 处理器叠加组合；无单 writer 选择

**参考：** `spec:AST-031/DEC-1`
**决策者：** `josephfarina`，`2026-09-10`

每个已配置处理器都会运行。先前模型基于目标所有权选择一个 writer，这意味着集成无法观察关于其不拥有的组件的报告。叠加组合让每个相关方收到每份报告，同时将失败隔离到单个投递。

已拒绝：按目标所有权选择单 writer；项目处理器压制集成处理器的基于优先级的覆盖；"首个匹配获胜"路由。

### DEC-2 — 运行时功能无通用退出机制

**参考：** `spec:AST-031/DEC-2`
**决策者：** `josephfarina`，`2026-09-10`

`debug` 功能现有的 `inheritDebug` 开关仍是由该功能拥有的兼容性例外。它不建立通用 `inherit<Feature>` 约定。Gap 报告是显式命令，其目的是通知每个已配置处理器，因此加载集成即选择加入其处理器。任何未来的退出机制需要单独的所有者决策。

已拒绝：自动将 `inheritDebug` 克隆为通用 package.json 模式；添加 gap-report 退出机制，静默阻止已加载集成接收显式报告。

### DEC-3 — 目标选择使用组件所有权，而非处理器数量

**参考：** `spec:AST-031/DEC-3`
**决策者：** `josephfarina`，`2026-09-10`

gap report 的目标包是拥有该组件的包。当多个包声称同一组件名时，CLI 要求显式 `--package` 而非猜测。无精确所有者的名称默认到 Core。处理器数量和处理器顺序从不改变目标归属。

已拒绝：按处理器数量或集成顺序选择目标；静默选择一个歧义所有者；将目标所有权分配给每个提供处理器的包。

## 开放问题

无。