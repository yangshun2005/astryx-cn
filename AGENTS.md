# Astryx 仓库指南

Astryx 是一个公开的设计系统仓库。切勿提交内部链接、标识符、服务名称、私有运维说明或其他仅限 Meta 内部的上下文。

## 指令面

本 `AGENTS.md` 是仓库的规范指令面，与工具无关。共享的 agent 指南应在此处添加或修改，不要在特定工具的指令文件中重复。真正与路径相关的指南放入嵌套的 `AGENTS.md`。

## 从这里开始

- 产品构建者：使用 `astryx docs`、组件 `{Name}.doc.mjs` 文件和 `packages/cli/assets/docs/`。
- 贡献者：阅读 `CONTRIBUTING.md` 以及 `docs/README.md` 中链接的相关指南。
- Pull Request：选择一个主要意图，使用 `.github/PULL_REQUEST_TEMPLATE/` 下对应的模板；在开启或审查混合变更前，先阅读 `docs/contributing/pull-requests.md`。请求审查前，在新会话中运行 `pnpm review:pre` 及其检查清单。
- 组件工作：推导审查的语义触发器，用 `node scripts/review-global-baselines.mjs --authority-commit <base-sha> --review-head <head-sha> --triggers <comma-separated-triggers>` 加载匹配的 `current` 全局基线声明，然后阅读组件的 `docs/specs/components/<ComponentRoot>/{Name}.spec.md`（若存在）及其列出的所有 `module:*` 记录。先加载全局记录，再加载更窄的归属者；但当直接组件或族归属者管辖确切变更时，优先解析它。全局路由只暴露所列声明，绝不让整条记录管辖组件或变更。在审查回执中保留每条匹配的记录、声明、触发器和匹配原因。
- 编辑路径前，运行 `pnpm specs:for <path>`。编辑后、提交或推送前，运行 `pnpm specs:for --diff origin/main` 并遵循每条按关注点路由的声明。
- 任意路径：`pnpm specs:for <path>` 列出链接到该路径的当前记录、决定这些记录的规格，以及其组件触发器路由的全局声明。它读取工作树，因此在 Pull Request 分支上显示该分支提议的记录；它只是查询，不是审查权威。它是对上述组件审查步骤的补充，不能替代它们。
- 组件本地视觉意图：仅当持久的本地意图必须在未来重新设计中保留时，才复用或修改当前组件/模块规格中可选的 `Design decisions` 表。精确调优保留在代码/证据中，共享规则保留在 `docs/design/` 中。DESIGNOWNER 作者可以在一个仅涉及外观的原子 PR 中，将本地意图连同像素和证据一并提交。外部贡献者遵循正常贡献流程，绝不被要求编写规格；缺失的权威由维护者负责。
- 跨组件工作：阅读 `docs/families/` 下的相关契约、`docs/design/` 下的适用设计规格，以及 `docs/architecture/` 下的当前架构。
- 重大共享系统变更：使用 `docs/specs/` 下的记录。
- 编写或审查任何知识记录：对记录运行 `docs/contributing/spec-writing.md` 中的评分标准，并将结果放入 Pull Request 描述的 `## Writing rubric results` 下。记录用现在时描述系统；提案、历史、Pull Request 和研究不纳入其中。审查时，对照报告的结果阅读记录；缺少结果的记录 Pull Request 是不完整的，因此在审查内容前先要求提供。
- 集成贡献工作：阅读 `docs/specs/AST-039/spec.md`。每个新的可发现项都拥有一个强类型、同名的 `.doc.mjs`；根清单定位目录，绝不编目其条目。永远不要有目录文件（如 `themes/manifest.json`），也不要在清单中放逐项映射。

## 权威

知识记录声明 `authority`：

- `draft`：非权威；可能仍需证据或归属者审查；
- `current`：明确批准且权威；
- `archived`：仅作上下文，附带原因如 `superseded`、`withdrawn` 或 `historical`，以及替代链接（若存在）。

只有 `current` 记录管辖实现和审查。绝不从已合并代码、沉默、旧审查或现有 wiki 页面推断批准。

## 判断边界

从代码、测试和浏览器证据中解决可检查的行为。仅当稳定的公共 API、主题契约、归属边界、兼容性策略或真正主观的视觉方向仍未确定时，才询问人类。一次只问一个问题。

在起草、审查或实现提议结果前，使用提议的规范归属者/id、受影响的路径和导出符号，以及行为的语义术语，搜索当前记录和开放的 Pull Request。默认扩展现有规范归属者或对其进行投影。仅为不同的事实边界创建新记录，并说明现有归属者为何无法容纳它。不要为已完成、已有归属或已被取代的工作创建新策略。

## 验证

编辑知识记录或模板后运行 `pnpm check:knowledge`。实质性的模板形状变更需要提升 schema 版本并迁移活跃记录；仅更改模板指南不会重写已接受的历史。

## 自定义命令

### `/vibe-test [count]` - 运行 vibeability 测试

测试 AGENTS.md 帮助 LLM 生成正确 Astryx 组件代码的效果。

**用法：**

```
/vibe-test 5                    # 运行 5 个分层样本测试（一次性）
/vibe-test                      # 运行全部 21 个测试（一次性）
/vibe-test 5 --degradation      # 运行 5 个测试并生成退化曲线（10 轮）
```

**执行方式：**

1. 运行 `pnpm -F @astryxdesign/vibe-tests interactive --sample <count>` 设置迭代
2. 启动并行子代理（每个测试提示一个）以：
   - 从 `results/<iteration>/tasks/{promptId}.json` 读取任务文件
   - 使用 Astryx 组件为提示生成代码（AGENTS.md 自动注入）
   - 自评估成功/逃生舱
   - 将 `.tsx` 结果写入 `results/<iteration>/results/{promptId}.tsx`
   - 将 `.json` 元数据写入 `results/<iteration>/results/{promptId}.json`
3. 触发 `gh workflow run vibe-screenshots.yml` 构建预览并捕获截图
4. 运行 `pnpm -F @astryxdesign/vibe-tests aggregate --iteration <id>` 查看结果

**退化模式（--degradation）：**
测试 10 轮对话中的上下文保持能力，包含填充轮、干扰轮和恢复轮。
在第 0、6、8、10 轮探测以衡量质量退化。结果以折线图显示每个测试的进展。

**结果格式：**

```json
{
  "id": "<iter>-<promptId>",
  "timestamp": "...",
  "model": "claude-code-interactive",
  "persona": "naive",
  "promptCategory": "...",
  "trajectoryDepth": 0,
  "prompt": "...",
  "response": "<code>",
  "evaluation": {"success": true, "componentsUsed": [...], "escapeHatches": [...]}
}
```

运行器还可以在结果元数据旁写入可选的 `<promptId>.provenance.json` 附属文件。版本化、执行器中立的契约和回退行为记录在 `internal/vibe-tests/docs/execution-provenance.md` 中。

## AI 上下文

有关架构上下文、决策和研究，请参阅 **[GitHub Wiki](https://github.com/meta/astryx/wiki)**：

- **决策** — API 约定、为何选择 StyleX、StyleX 分发
- **架构** — 系统架构、组件编写指南
- **研究** — AI + 设计系统、AI 模型轨迹、Swizzle 人体工程学
- **未来** — 动画系统、RSC 工具、分发策略

有关组件特定文档，请参阅 `packages/core/src/` 下每个组件目录中的 `{Name}.doc.mjs` 文件（例如 `Button/Button.doc.mjs`）。这些是带有 JSDoc 类型注解的普通 JS 文件，导出一个 `ComponentDoc` 对象（通过 `@astryxdesign/cli/authoring` 类型化）。

## 文档标准

文档存在于两个地方：

1. **文件头** — 每个源文件都有一个结构化的 JSDoc 头，包含 `@input`、`@output`、`@position`，最多约 12 行
2. **组件文档** — 每个组件目录中的 `{Name}.doc.mjs` 文件（Props、特性、示例）

**更新协议**：修改代码时，保持文件头注释准确。留意 `SYNC:` 注释作为提醒。

**源码注释**（[`architecture:knowledge-contracts`](docs/architecture/knowledge-contracts.md#source-comments) INV29）：让代码通过名称、结构和控制流自我解释。注释只解释非显而易见的工作：原因、约束、不变量或浏览器怪癖。绝不叙述代码、重述名称或类型，或引用 Pull Request、Issue、日期或规格记录（`pnpm specs:for <path>` 可找到记录）。当引用是文件与记录或声明的唯一链接时，在移除前先将该链接添加到记录中：对于整条记录，通过 `applies_to`、`verified_by` 或 `path-applicability:v1` 链接文件；对于如 `AST-108 FR4` 的声明，在文件自身的组件、模块或主题记录中添加声明范围的引用，如 `system_specs: [spec:AST-108/FR4]`。用 ASD-STE100 简化技术英语编写：短句、每句一个主题、主动语态、现在时。保留公共 JSDoc、命名其同步系统的 Effect 注释，以及工具读取的指令：lint 和 TypeScript 指令、`@pr-a11y` 行，以及仓库工具解析的任何其他注释行。记录中有完整的修改前后示例。

**受众**：每个 `.doc.mjs` 以及 `packages/cli/assets/docs/` 下的所有内容，都是为**使用** Astryx 构建的人编写的——不是为构建 Astryx 的人编写的。内部审查评分标准、就绪门禁、审计检查清单和 lab→core 标准属于 wiki。当调用者使用公共编写质量评分标准来评估他们通过 Astryx 创建的产物，且当前系统规格分配了该归属时，该评分标准属于随附文档。[`packages/cli/assets/docs/README.md`](packages/cli/assets/docs/README.md) 有测试和每类材料应放置的页面。

## 快速参考

- **包管理器**：pnpm 11，由 `packageManager` 字段固定（安装选项见 CONTRIBUTING.md——Corepack 是多种选项之一，Node 25+ 不再捆绑它）
- **测试**：Vitest（同目录测试）
- **组件**：`packages/core/`
- **Storybook**：`apps/storybook/`
- **包内导入**：导入定义模块（`'../List/List'`），绝不导入文件夹的索引桶（`'../List'`）；公共消费者导入 `@astryxdesign/core`。`@astryx/no-internal-barrel-import` 强制执行此规则。

## JSDoc 约定

- **`@example` 代码围栏必须使用普通 ` ``` `，而非 ` ```tsx `。**
  Storybook 的 autodocs 解析器无法正确处理 JSDoc 中带语言标记的围栏——代码块将无法渲染为正确的代码块。在 `@example` 块中始终使用无标记围栏。

<!-- STYLEX-CAPS:START -->

[StyleX v0.19.0 CSS 支持]|使用 CSS 原生解决方案。不要为受支持的特性构建 JS 变通方案。
|AT-RULES: @media, @supports, @container (+named), @starting-style, @scope — 支持
|AT-RULES: @layer, @property (explicit) — 不支持（可编译但 CSS 输出无效）
|PSEUDO-CLS: :hover, :focus, :focus-visible, :focus-within, :active, :disabled — 支持
|PSEUDO-CLS: :first-child, :last-child, :nth-child(), :where(), :is(), :has(), :not() — 支持
|PSEUDO-CLS: :placeholder-shown, :checked, :empty, :modal, :user-valid, :user-invalid — 支持
|PSEUDO-EL: ::before, ::after, ::placeholder, ::selection, ::backdrop, ::marker, ::view-transition-_ — 支持
|COMPOUND: ::backdrop+condition, RTL :is([dir="rtl"] _), nested @media+pseudo — 支持
|VALUES: var(), calc(), clamp(), light-dark(), color-mix(), container-type/name — 支持
|ANIM: transition (shorthand+individual), transitionBehavior:allow-discrete, animation longhands, stylex.keyframes — 支持
|ANIM: animation shorthand — 不支持（使用 animationName/duration/timingFunction 等）
|WHEN: stylex.when.ancestor(':hover'/':focus-within'/':active'/':disabled') — 支持
|WHEN: stylex.when.descendant(':hover'), siblingBefore(':checked'), siblingAfter(':checked'), anySibling(':hover') — 支持
|WHEN: stylex.when.ancestor('[data-attr]') — 支持
|NESTING: CSS nesting with & — 不支持（使用 stylex.when.ancestor/descendant/sibling 处理父子状态）
|API: stylex.firstThatWorks() 用于 CSS 回退（例如 display: grid 带 flex 回退）— 支持
|API: stylex.positionTry() 用于锚点定位 @position-try — 支持
|API: stylex.types.color/length/etc 用于 defineVars 中的类型化 CSS 变量 — 支持
|API: stylex.defineConsts() 用于编译时常量 — 支持
|DYNAMIC: stylex.create 中的函数用于运行时值 — 支持
|VARS: stylex.defineVars, stylex.createTheme（需要 .stylex.ts 文件）— 支持
|LAYOUT: grid, flex+gap, aspect-ratio, overscrollBehavior, scrollbar-gutter/width — 支持
|PATTERN: 对话框进入动画 -> @starting-style（不是 useState+rAF）
|PATTERN: 父悬停子样式 -> stylex.when.ancestor(':hover', marker)（不是 CSS 嵌套）。在 .stylex.ts 文件中使用 stylex.defineMarker() 创建作用域标记。祖先元素必须在其 stylex.props() 调用中包含 marker.marker。绝不要对表单控件（CheckboxInput、RadioList、Switch）使用 stylex.defaultMarker()——它会从 Popover 等外部容器泄漏 hover/focus-within。始终使用组件作用域的 defineMarker()。
|PATTERN: 触摸悬停 -> @media (hover: hover) 守卫
|PATTERN: 斑马纹 -> :nth-child(even)（不是 index%2 JS）
|PATTERN: 容器响应式 -> @container（不是 ResizeObserver）
|PATTERN: CSS 回退值 -> stylex.firstThatWorks()（不是手动回退）
|PATTERN: 动态/运行时值 -> stylex.create({ s: (val) => ({ prop: val }) })（不是内联样式）
|PATTERN: 条件样式 -> stylex.props(condition && styles.x)（不是 className 切换）
|PATTERN: 链接元素 -> useLinkComponent()（不是硬编码 <a>）。消费者通过 LinkProvider 为框架路由器（Next.js、React Router）替换
|VERIFY: node internal/stylex-capabilities/scan.mjs

<!-- STYLEX-CAPS:END -->

<!-- ASTRYX-CLI:START -->

Astryx CLI|从仓库根目录运行。在任何组件工作前加载 agent 文档。
astryx() { node packages/cli/clients/cli/bin/astryx.mjs "$@"; }
BOOTSTRAP（每个分支运行，<500ms）：
astryx help # 发现所有命令和选项
astryx docs # 列出可用文档主题
astryx docs principles --dense # 设计规则、反模式、xstyle、令牌
astryx docs tokens --dense # 间距、颜色、圆角、排版、阴影
astryx docs theme --dense # 主题提供者、浅色/深色、覆盖
astryx component --list # 按类别分组的所有组件
astryx template --list # 可用页面模板
按需：
astryx docs <topic> --index # 章节索引：每节一行，含其键
astryx docs <topic> <section> # 按键或标题查看一节
astryx component <Name> --dense # 单个组件的 Props、变体、用法、解剖
astryx template <name> # 输出完整页面源码
astryx template <name> --skeleton # 带空间注释的布局骨架
astryx swizzle <Name> # 弹出组件源码以深度定制
astryx upgrade --from <old version> --apply # 运行版本迁移 codemod
选项：--detail compact|brief 更少输出 | --dense 令牌高效 | --zh 中文
规则：每个分支始终运行 bootstrap——文档反映分支的实际 API
规则：修改组件前始终运行 astryx component <Name> --dense
规则：@astryxdesign/core 升级后，始终运行 astryx upgrade --from <old version> --apply

<!-- ASTRYX-CLI:END -->