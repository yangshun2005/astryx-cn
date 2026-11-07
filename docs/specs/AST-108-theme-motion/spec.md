---
schema_version: 4
template_version: 2
kind: system-spec
id: spec:AST-108
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-10-26
phase: accepted
owners: [cixzhang]
affects_architecture:
  [
    architecture:theme-tokens,
    architecture:theme-authoring-contract,
    architecture:theme-application,
  ]
affects_families: []
affects_contributing: []
affects_consumer_docs: [motion, tokens, theme]
---

# 主题动效系统规范

<!-- path-applicability:v1 -->

```json
{
  "paths": ["packages/core/src/theme/**", "packages/core/src/motion/**"]
}
```

<!-- 描述系统而非项目：用现在时，说明它做什么。记录中不写提案、历史、Pull Request 或研究；参见 docs/contributing/spec-writing.md，并在 Pull Request 中报告其评分结果。 -->

## 意图

主题通过三个旋钮设定动效的快慢与活泼程度，以及是否为所有人减弱动效。同一组旋钮驱动 CSS 动效 token 与动画器的弹簧和曲线，同时运行的动画同时开始、同时结束。所有按用途区分的动效值保持内部。

## 非目标

- 为主题提供按角色或按组件的动效设置。
- 放宽 Reduce Motion：对于已请求减弱动效的人，任何选项都不能关闭 `design:reduced-motion` DR1–DR4。
- 为主题动效值提供公开解析器（OQ1）。

## 公开契约

```ts
defineTheme({
  name: 'Brand',
  motion: {tempo: 1, character: 'standard', reducedMotion: 'system'},
});
```

## 需求

- **FR1 — 三个动效旋钮。** `defineTheme` 的 `motion` 接受
  `{tempo, character, reducedMotion}`：
  - `tempo` 是数字，默认 `1`，钳制在 0.75–1.5；它缩放所有动效时长；
  - `character` 为 `'crisp'`、`'standard'`（默认）或 `'expressive'`；它决定入场与移动曲线以及弹簧回弹。不提供连续回弹。任何 character 都不会把曲线变成弹簧，也不会播放平台测量的曲线；哪些动效用弹簧、哪些播放曲线 token 由 `spec:AST-094` FR45 规定，点击触发的展开与收起（包括动画布局滑动，`spec:AST-105` FR4）播放曲线 token；
  - `reducedMotion` 为 `'system'`（默认）或 `'always'`；`'always'` 为所有人应用 `design:reduced-motion` DR1–DR3，`'system'` 遵循每个人的设置。任何值都不放宽 Reduce Motion。
- **FR2 — 旋钮生成 CSS 动效 token。** 主题的 `--duration-*` token 为其基础值乘以 `tempo`。`character` 只设置按用途命名的 `--ease-entrance` 和 `--ease-move` token，声明于 `tokens.stylex.ts`。现有退场读取的 `--ease-standard`，以及默认值为 `cubic-bezier(0.3, 0, 0.6, 0.6)` 的 `--ease-exit`，在所有 character 下相同，因此退场保持安静。当 `tempo` 为 1、`character` 为 `'standard'` 时，token 保持已发布的值：`--duration-*` 为 130、175、230、310、410、550、730、975 和 1300 ms，`--ease-standard` 为 `cubic-bezier(0.24, 1, 0.4, 1)`；在 `'standard'` 下 `--ease-entrance` 和 `--ease-move` 等于 `--ease-standard`。`'crisp'` 和 `'expressive'` 的曲线分别是仅减速曲线和带轻微过冲的曲线。
- **FR3 — 动画器的表由同一组旋钮派生。** 动画器的基础表（`spec:AST-094` FR4 中的每个角色）在源头由主题的 `tempo` 和 `character` 派生：弹簧时长按 `tempo` 缩放，`character` 按角色固定权重乘以一个量来偏移该角色测量的回弹：`'standard'` 不增加，`'crisp'` 和 `'expressive'` 分别向下和向上偏移，因此权重为零的角色（如退场、淡入淡出或全屏移动）在任何 character 下都保持其测量的回弹。当 `tempo` 为 1、`character` 为 `'standard'` 时，每个角色精确播放其测量值。弹簧保持其测量的物理特性，按 `tempo` 缩放并由 `character` 塑造；它不跟随 CSS 时长或缓动的对应值。角色及其权重保持内部。
- **FR4a — 协同动画共享一个时钟。** 同时运行的动画同时开始、同时结束：它们播放同一动效角色，或 CSS 部分按弹簧计算出的稳定时间计时，绝不用两条独立计时的曲线。这涵盖元素及其周围滑动的兄弟元素（`spec:AST-105` 的单一共享计时）、元素的入场与退场、面板与其遮罩、菜单的展开与其内容，以及 CSS 淡入淡出与 JavaScript 弹簧配对的情况，例如 MobileNav 的关闭按其计算时长计时，BottomSheet 的遮罩淡入淡出持续面板计算出的稳定时间。
- **FR4 — 部分动效有意固定。** `tempo` 和 `character` 不影响
  跟随手指与拖拽的动效、滚动惯性与衰减、橡皮筋、屏幕键盘的动效（`spec:AST-099` FR8）、按压模型的时钟（其起始延迟、取消距离和点击闪烁）、悬停意图延迟、手势阈值（`spec:AST-002` FR3）、忙碌指示器（如 Spinner 和 Skeleton）的循环周期（它们读取主题在 `tempo` 缩放前设置的时长，因此 `MotionScaleConfig` 仍为其计时），以及 200 ms 的 Reduce Motion 上限；`tempo` 可以缩短 Reduce Motion 的交叉淡入淡出，但绝不延长。
- **FR5 — 现有动效配置继续有效。** `defineTheme` 的 `motion` 也接受现有的 `MotionScaleConfig`，`expandMotionScale` 和 `MotionScale*` 类型不变。`MotionScaleConfig` 继续生成其 CSS 时长 token，其隐含的 tempo 只驱动这些时长：除非主题自行设置 `tempo`，动画器的表保持 tempo 1，因此现有主题的渲染与之前完全一致，包括弹簧。显式的 `MotionScaleConfig` 字段和显式的 `tokens` 优先于 `tempo` 和 `character`，扩展主题逐字段继承：子主题未设置的 `MotionScaleConfig` 字段或旋钮保持基主题的值。适配规则可以像其他 `motion` 值一样设置 `tempo` 和 `character`（`spec:AST-012` FR4）；`reducedMotion` 在主题本身上设置，因为其根标记没有条件形式。
- **FR6 — 服务端渲染得到相同的值。** 主题是纯数据，主题工具在服务端运行。`'system'` 不添加主题 CSS：组件样式已基于 `prefers-reduced-motion` 处理减弱动效，因此服务端从不猜测，水合时也不会闪烁。`'always'` 标记主题根，所有基于 `prefers-reduced-motion` 处理减弱动效的组件样式也基于该标记处理；动画器、所有在脚本中读取 `prefers-reduced-motion` 的地方（`matchMedia` 或 `useMediaQuery` 检查），以及所有以 `when: {motion}` 为键的适配规则（`spec:AST-012` FR2）都将该偏好视为已设置。该标记只做叠加：祖先的 `'always'` 会传入每个嵌套主题，嵌套主题可以添加 `'always'`，但绝不移除祖先的，因为没有任何东西放宽 `design:reduced-motion` DR1–DR3（DR2a）。服务端和客户端从主题解析出相同的动效值，导入主题和动效模块不触碰 DOM（`architecture:theme-authoring-contract`）。动画器仅在客户端动画开始时读取个人的 Reduce Motion 设置。
- **FR7 — 文档随实现交付。** 动效、时长 token 和缓动 token 文档、主题模板、主题编写指南以及 `architecture:theme-authoring-contract` 在实现落地前描述三个旋钮、固定列表和优先级。

## 当前状态影响

- `defineTheme` 的 `motion` 扩展为在现有 `MotionScaleConfig` 之外接受三个旋钮；现有主题的渲染与之前完全一致。
- 在 `reducedMotion: 'always'` 发布前，公开的 `motion` 类型没有 `reducedMotion` 字段，每个主题都遵循每个人的设置，如同 `'system'`（`architecture:theme-authoring-contract` 中的已知缺口）。
- Core 新增 `--ease-entrance`、`--ease-move` 和 `--ease-exit`，以及用于 `reducedMotion: 'always'` 的主题根标记，所有基于 `prefers-reduced-motion` 处理减弱动效的组件样式也基于该标记处理；Carousel 的分页、BottomSheet 的面板与面板手势、Item 的滑动释放、MobileNav、AppShell、Chat、Markdown 的流式显示、BottomSheet 的键盘滚动，以及动画布局滑动（它们在脚本中读取该偏好）也遵循它，以 `when: {motion}` 为键的适配规则（`spec:AST-012`）同样如此。
- `design:reduced-motion` 新增 DR2a：主题可以为所有人减弱动效。
- Spinner 和 Skeleton 在 `tempo` 缩放前读取其循环时长（FR4），因此在所有现有主题下其计时不变；`component:Spinner` FR3 和 `component:Skeleton` FR4 已说明这一点。
- BottomSheet 的面板与遮罩共享 `--duration-medium`；当面板以 `springs.settle` 稳定时，其遮罩的淡入淡出采用弹簧的稳定时间（FR4a）。
- `spec:AST-094` FR1、FR4 和 DEC-15、`design:reduced-motion` DR2a、`spec:AST-012` FR4（`reducedMotion` 仍归根所有）、`architecture:theme-application` INV1（`'always'` 传入嵌套 provider）、`architecture:theme-authoring-contract`（内部动效解析器）、`component:Button` FR15（其释放淡入淡出通过 `--duration-exit` 跟随 `tempo`，该 token 引用按 tempo 缩放的时长）、草稿 `design:motion` DR3 以及 `spec:AST-090` 均相应修订。

## 验证

| 契约 | 验证 | 代表性状态 | 变异或失败预期 |
| -------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| FR1, FR2 | 针对生成的 CSS 的主题单元测试 | 每个 `tempo` 边界值和 1；每个 `character`；`reducedMotion` 两个值；无 `motion` | token 偏离公式、默认值下已发布 token 改变、退场曲线改变，或值超出钳制范围，均失败 |
| FR3 | 测试 CSS token 与动画器的表由同一组旋钮值派生 | 默认值下及每个旋钮值下的每个角色；设置 `tempo` 或 `character` 的适配规则 | 默认值下角色偏离测量值、角色忽略其遵循的旋钮，或权重为 0 的角色回弹随 `character` 改变，均失败 |
| FR4a | 每个旋钮值下每对配对动画的浏览器测试 | 元素与滑动兄弟元素、元素的入场与退场、面板与其遮罩、菜单的展开与内容、CSS 淡入淡出与弹簧 | 配对动画结束时间相差超过一帧，失败 |
| FR4 | 每个固定项的单元测试 | 手指跟踪、惯性、橡皮筋、键盘、按压与悬停时钟、循环、Reduce Motion 上限；每个 tempo 下 Spinner 和 Skeleton 的循环 | 固定项随 `tempo` 或 `character` 移动，或交叉淡入淡出超过 200 ms，失败 |
| FR5 | 使用已发布配置的主题测试 | `MotionScaleConfig` 主题、显式 token、扩展主题；设置 `tempo` 或 `character` 的适配规则 | 现有主题输出改变，或旋钮胜过显式字段，失败 |
| FR6 | 服务端渲染测试和导入检查 | 每个 `reducedMotion` 下的服务端渲染；导入主题和动效模块；在个人设置关闭时 `'always'` 跨基于 `prefers-reduced-motion` 的组件；`'always'` 主题下的嵌套主题 | 导入时或服务端访问 DOM、为 `'system'` 输出主题 CSS、水合变化，或组件样式、脚本读取器、`when: {motion}` 规则在 `'always'` 下忽略根标记，或嵌套主题在祖先 `'always'` 下恢复动效，均失败 |
| FR7 | 文档评审 | 每个列出的文档 | 文档缺少旋钮、固定列表或优先级，失败 |

## 决策日志

### DEC-1 — 主题的动效是三个旋钮；按角色的表保持内部

**引用：** `spec:AST-108/DEC-1`
**决策者：** `cixzhang`，`2026-10-26`

主题作者设定动效的快慢与活泼程度，以及是否为所有人减弱；很少有人需要按用途区分的值。三个 character 档位（而非连续回弹）是随旋钮采纳的建议。三个旋钮用同一组值驱动 CSS token 和 JavaScript 弹簧，测量的按角色值保持内部。这取代 `spec:AST-094` DEC-11 的"全部内部"，仅针对这三个旋钮。

已拒绝：为每个动效组提供公开 token。

## 开放问题

- **OQ1 — 主题的动效值是否获得公开解析器。**
  （`human-design`）开放。`resolveThemeMotion` 为内部。