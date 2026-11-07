---
schema_version: 4
template_version: 1
kind: system-spec
id: spec:AST-025
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-09-11
phase: accepted
owners: [cixzhang]
affects_architecture:
  [
    architecture:container-padding,
    architecture:public-component-api,
    architecture:interaction-modality,
  ]
affects_families: [family:layout-regions, family:layout-primitives]
affects_contributing: []
affects_consumer_docs:
  [Layout, Stack, Table, Popover, DropdownMenu, Carousel, CodeBlock, Tooltip]
---

# 可滚动容器行为系统规范

<!-- review-applicability:v1 -->

```json
{
  "scope": "global",
  "triggers": {
    "scrolling": ["FR1", "FR12", "FR13", "FR14", "FR15", "FR16", "FR18"]
  }
}
```

## 意图

用户应能触达并滚动溢出内容；当嵌套区域无内容可滚动时，应能继续滚动外层页面；并应清楚哪些内容在移动时保持固定。构建者不应被迫分别协调无障碍、粘性定位、过度滚动包含、滚动条呈现和浏览器特定的溢出行为。

本规范定义一项轴感知的滚动容器能力。共享工具 Hook 构成规范的可复用行为接口，使已拥有合适结构的组件可直接参与。`ScrollableArea` 将这些 Hook 组合为便捷组件和参考实现；它并非获得该行为的唯一途径。

## 非目标

- 在本草案中确定最终导出的 Hook 名称、Prop 名称或 TypeScript 拼写；Hook 优先的所有权与 `ScrollableArea` 参考组合已由 DEC-1 确定。
- 用 JavaScript 驱动的移动替代浏览器滚动。
- 要求自定义滚动条 DOM，或将各平台滚动条样式统一。
- 将任意溢出值或独立的 `overflow-x` / `overflow-y` 组合纳入设计系统公开词汇。
- 允许内容透过同一视口的非滚动轴可见绘制；CSS 无法可靠提供该组合。
- 定义 Sticky 的完整公开 API、层叠模型或视觉处理。
- 自动迁移所有现有 `overflow: auto` 调用点。
- 更改 Layout 的五槽拓扑或 AppShell 对页面外壳的所有权。

## 术语

- **视口（Viewport）：** 其滚动偏移会移动内容的元素。
- **内容盒（Content box）：** 视口内的真实布局盒，其几何可被观测，无需任意后代参与。
- **请求轴（Requested axis）：** 所有者意图允许滚动的逻辑轴：`inline`、`block` 或 `both`。
- **有效滚动轴（Effective scroll axis）：** 请求轴中，解析后的计算溢出值可由用户滚动，且内容范围超出视口范围超过共享容差。
- **有效滚动所有者（Effective scroll owner）：** 具有与所解析行为匹配的有效滚动轴的最近祖先。
- **滚动条呈现器（Scrollbar presenter）：** 原生 CSS 或可选自定义 UI，表示视口现有滚动状态，不创建另一视口。

## 需求

- **FR1 — 滚动意图使用逻辑轴。** 系统必须将滚动意图表示为 `inline`、`block` 或 `both`。书写方向和书写模式决定物理轴。左、右、上、下不得定义共享契约。
- **FR2 — 滚动所有权显式且局部。** 参与容器必须标识一个视口及其内容盒。无论能力通过独立组件还是集成到现有组件自有元素交付，后代必须收到相同的逐轴所有者状态。
- **FR3 — 有效滚动需要可滚动样式和可测量溢出。** 读取物理几何前，先通过视口的计算书写模式解析请求逻辑轴。在水平书写模式下，inline 比较 `scrollWidth` 与 `clientWidth`，block 比较 `scrollHeight` 与 `clientHeight`；垂直和侧向书写模式反转这些物理测量。仅当解析后的计算溢出为 `auto`、`scroll` 或旧版 `overlay` 值，且内容范围超出视口范围超过共享容差时，轴才有效。初始共享容差为 1 CSS 像素。仅 `overflow: auto` 声明或仅几何超出不得使容器成为有效所有者。
- **FR4 — 有效状态按轴区分。** 容器可在一个、两个或零个请求轴上有效。可聚焦性、粘性所有权、过度滚动包含、滚动条呈现和边缘状态必须使用适用轴，而非单一未区分的 `hasOverflow` 布尔值。
- **FR5 — 实时测量观测视口和内容几何。** 系统必须同时观测视口盒和真实内容布局盒。仅视口的 `ResizeObserver` 不足，因为内容可能增长而视口自身盒不变。`display: contents` 不是有效的内容观测目标，因为它没有可靠盒几何。
- **FR6 — 现有结构可提供内容盒。** 已拥有合适内容根的组件可直接注册它。可复用行为不得静默在任意子元素周围插入包装器。当实现创建内容包装器时，必须记录并验证该包装器的格式化、尺寸、flex/grid 参与、百分比尺寸和固有尺寸影响。
- **FR7 — 失效覆盖范围超出元素尺寸变化。** 系统必须在初始连接、视口或内容盒尺寸变化、请求轴或相关样式变化，以及从不可测量/隐藏转为渲染后测量。溢出可在两个观测盒均未调整尺寸时变化的集成，必须提供显式失效路径或等效有界信号。同一帧内重复失效必须合并为一次测量。
- **FR8 — 不可测量不等于适配。** 断开连接、`display: none` 或零几何视口不得被视为内容适配的证据。实现可保留最后有效状态或暴露内部未知状态，但视口变为可测量时必须重新测量。
- **FR9 — 状态更新稳定。** 共享观察器仅当有效轴或请求边缘信息变化时发布新状态。测量必须收敛，不产生溢出/滚动条反馈循环，包括原生滚动条占用布局空间的平台。
- **FR10 — Sticky 按轴针对有效所有权解析。** block-start 或 block-end Sticky 参与者必须使用最近的有效 block 所有者。inline-start 或 inline-end 参与者必须使用最近的有效 inline 所有者。未溢出的候选和仅在另一轴有效的候选必须跳过。流动内容中的任意 Sticky 放置必须仍可行；固定页眉/页脚槽不是通用替代。
- **FR11 — 原生 CSS sticky 是有条件实现细节。** 当浏览器最近的 CSS 滚动容器也是系统在请求轴上的有效所有者时，可使用原生 `position: sticky`。当 CSS 溢出轴耦合使这些所有者不一致时，实现不得依赖原生祖先选择。本规范不选择回退定位机制。
- **FR12 — 键盘访问跟随有效滚动。** 有效滚动内容必须按 WCAG 2.1.1 保持键盘可达。将键盘滚动分配给视口的交付表面，必须仅当至少一个适用轴有效时将其加入 Tab 顺序，并提供适当角色和可访问名称。角色为 `group`，除非视口本身是地标。名称为拥有该视口的组件的可访问名称：调用方标签，否则该组件的翻译默认值。作为页面其角色唯一地标的视口（如单个 `main`）可依赖该角色且无需名称；作为其角色多个地标之一的视口必须命名。集成也可依赖现有可聚焦后代，前提是该路径让键盘用户访问所有溢出内容。在共享 `contentOrViewport` 模式下，命名视口在有效可滚动时必须保留 `tabIndex=0`。正向键盘 Tab 进入时，Hook 必须检查一次第一个顺序后代，且仅可委托给保留原生滚动键的原生链接或按钮。不得跳过被排除的第一个目标以到达后续合格目标。复合组件、嵌套滚动所有者、可编辑表面和拥有导航键的角色必须保留视口停靠点。从委托的第一个子元素反向遍历时必须跳过视口且不循环。指针和程序化焦点不得委托。合格性必须在每次键盘进入时重新计算，不进行持久子树或祖先观察以确定键盘合格性；现有几何观察仍受 FR5–FR9 约束。两条路径都必须保留清晰的可访问命名和原生 Arrow/Page 访问完整滚动范围。工具提示是唯一例外：内容过高超出视口时在其内部滚动，无键盘路径。根据 ARIA 工具提示模式，工具提示从不获取焦点，其触发器在显示时保持焦点；完整内容通过触发器的可访问描述对辅助技术可用；长或重要内容应放入 HoverCard 或 Popover，它们会获取焦点；替代方案——将溢出留在视口外——无人可及。
- **FR13 — 失去溢出不移动焦点。** 若聚焦视口停止溢出，系统可将其从未来顺序导航中移除，但不得仅因几何变化而移动或模糊当前焦点。
- **FR14 — 滚动链是默认行为。** 当当前区域在请求方向无移动时，嵌套滚轮、触摸和键盘滚动必须继续到祖先。包含策略可停止链，但其 `overscroll-behavior` 效果必须仅在当前溢出的轴上生效。这避免在非溢出 `overflow: auto` 元素上遵守包含的引擎中出现死滚动区。在手指下的触摸呈现中，谁获得拖拽在开始时一次性决定（`spec:AST-094` FR14）。
- **FR15 — 原生滚动条是默认呈现。** 默认呈现器必须保留原生滚动，并在支持时使用基于标准的 CSS，如 `scrollbar-width`、`scrollbar-color` 和 `scrollbar-gutter`。不支持的声明必须回退到平台滚动条，不改变滚动所有权或访问。
- **FR16 — 自定义滚动条是可选呈现。** 自定义轨道或滑块可表示现有视口，但不得创建第二滚动容器、替代原生滚动机制或维护独立事实来源。它必须使用与视口相同的有效轴、范围、偏移、书写方向和强制颜色状态。隐藏原生滚动条而无等效可见可操作提示不是通用默认。
- **FR17 — 轴组合封闭。** 系统不得暴露 `visible`、`clip`、`hidden`、`auto` 和 `scroll` 的任意配对。当与另一轴的非可见值配对时，CSS 将 `visible` 计算为 `auto`，`clip` 计算为 `hidden`，因此这些表面组合不保留独立轴语义。必须逃逸非滚动轴的内容需要单独结构或 Layer。
- **FR18 — 嵌套所有权确定性。** 适用轴上最近的有效所有者胜出。正交嵌套区域可共存。同轴嵌套必须默认保留链，且必须在拥有组件的契约中有意为之，而非源自偶然溢出样式。
- **FR19 — Table 可集成而非包装。** Table 可在其现有滚动包装器上采用该能力，使列尺寸、键盘访问、滚动阴影、粘性列和粘性页眉共享一个视口。Table 不得仅为参与而要求单独的公开 ScrollableArea 组件。任意外部包装器无法满足 Table 的内部所有权契约，除非 Table 明确接受并协调该所有者。
- **FR20 — 参考组件集成容器几何。** ScrollableArea 必须发布其内容盒的实际逻辑内边距，未请求内边距时使用零。其视口仅可通过显式全出血选项消耗继承内边距；仅可滚动性不得逃逸父容器。
- **FR21 — 适配滚动意图不意味着 Sticky 包含。** 请求轴无超出几何的参与视口必须在两个物理轴上使用 `clip`。这防止测量前绘制溢出而不创建 CSS 滚动容器，使原生 Sticky 后代可解析到外层有效所有者。内容超出视口时，视口必须在轴成为有效所有者之前切换到请求的 `auto`/`hidden` 对。公开 `stickyContainment="always"` 选项可在适配时保留该边界，但包含必须显式，而非滚动意图的偶然结果。

### 交付需求

- **IR1 — 共享 Hook 是行为核心。** 共享工具 Hook 必须拥有有效轴测量、观察、失效和所有者发布。`ScrollableArea` 和组件集成形式必须使用该核心，而非创建并行检测器。
- **IR2 — 行为不要求 `ScrollableArea`。** 已拥有视口和内容根的组件必须能够采用 Hook，无需添加便捷组件或更改其语义外层元素。
- **IR3 — `ScrollableArea` 是参考组合。** 该组件拥有并记录其视口/内容结构，应用共享无障碍、链、粘性所有者和滚动条呈现规则，并演示另一拥有结构的组件如何集成 Hook。它不得包含直接 Hook 采用者不可用的行为。
- **IR4 — 结构和 Prop 组合保持显式。** Hook 必须通过 Prop getter 或等效组合契约接受调用方拥有的视口/内容 Prop 和 ref。它必须组合 ref、事件处理器、class/style 输入和 ARIA，不使展开顺序决定调用方或行为丢失。组件拥有的无障碍和不可取消行为在已记录冲突中胜出；可取消处理器使用共享事件契约。Hook 不假装结构不必要，也不静默包装任意子元素。
- **IR5 — 公开 API 跟随意图。** 未来 Hook 和组件名称暴露语义滚动、链、无障碍或呈现意图。原始 CSS 值、观察器控制、容差旋钮和浏览器变通保持内部，除非单独准入决策证明调用方所有权。

### 候选共享 Hook 边界

确切名称和 TypeScript 拼写仍属 API 审查工作。候选边界为：

```ts
type ScrollAxis = 'inline' | 'block' | 'both';

type KeyboardAccess =
  | {owner: 'content'}
  | {owner: 'viewport'; label: string; role?: 'group' | 'region'}
  | {
      owner: 'contentOrViewport';
      label: string;
      role?: 'group' | 'region';
    };

interface ScrollAxisState {
  isScrollable: boolean;
  atStart: boolean;
  atEnd: boolean;
}

interface UseScrollableAreaOptions {
  axis: ScrollAxis;
  keyboardAccess: KeyboardAccess;
  overscroll?: 'allow' | 'contain';
  stickyContainment?: 'whenScrollable' | 'always';
}

type ElementProps<E extends HTMLElement> = BaseProps<E> &
  React.RefAttributes<E>;

interface UseScrollableAreaResult {
  getViewportProps<E extends HTMLElement>(
    props: ElementProps<E>,
  ): ElementProps<E>;
  getContentProps<E extends HTMLElement>(
    props: ElementProps<E>,
  ): ElementProps<E>;
  state: {
    inline: ScrollAxisState;
    block: ScrollAxisState;
  };
}
```

每个 Prop getter 接收采用方已解析的元素 Prop 和公开 ref，通过共享 Prop/ref 工具与 Hook 行为组合，并返回一个安全展开对象。`getViewportProps` 消耗调用方 `xstyle`，添加行为拥有的适配 clip、轴特定活动溢出、显式 Sticky 包含、键盘、ARIA、事件、data-state、过度滚动和注册行为，不选择视口尺寸或滚动条呈现。`getContentProps` 将内容观察与现有内容 ref 和 Prop 组合。松散 Prop 包或单独回调 ref 均不使展开顺序成为契约一部分。

原生滚动条宽度、颜色和槽保持为视口上的 CSS，而非 Hook 选项。可选自定义呈现器和滚动阴影通过更窄订阅消耗相同注册指标，使普通所有者不会为每个滚动偏移重新渲染。Sticky 使用基于已注册 DOM 祖先的私有最近所有者消费者；采用方不渲染 Provider。

显式失效方法保持内部，除非真实采用方证明溢出变化既无法被观测盒也无法被有界平台信号检测。

### 行为矩阵

| 请求轴   | Inline 有效 | Block 有效 | 有效所有者状态         | Sticky 所有权                                      |
| -------- | ----------- | ---------- | ---------------------- | -------------------------------------------------- |
| `inline` | 否          | 不适用     | 无                     | 无 inline 所有者；block Sticky 跳过候选            |
| `inline` | 是          | 不适用     | inline                 | inline Sticky 在此解析；block Sticky 跳过          |
| `block`  | 不适用      | 否         | 无                     | 无 block 所有者；inline Sticky 跳过候选            |
| `block`  | 不适用      | 是         | block                  | block Sticky 在此解析；inline Sticky 跳过          |
| `both`   | 是/否       | 是/否      | 每个有效轴独立         | 每个 Sticky 边缘仅在其匹配有效轴上解析             |

由于溢出计算值耦合，浏览器仍可能将一个视口在两个物理轴上归类为 CSS 滚动容器。该平台事实不改变本表中的系统所有者状态。

### 滚动条呈现矩阵

| 呈现           | 滚动机制                   | 渲染规则                                                                 | 必需回退                                                         |
| -------------- | -------------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------- |
| 原生/默认      | 浏览器在视口上拥有         | 平台滚动条；基于标准的宽度、颜色和槽样式可优化                           | 忽略不支持的样式并保留原生行为                                   |
| 原生/隐藏      | 浏览器在视口上拥有         | 仅当另一清晰提示和键盘路径暴露溢出时允许                                 | 在强制颜色或替代不可用时恢复原生呈现                             |
| 自定义         | 浏览器在同一视口上拥有     | 轨道/滑块镜像有效轴和实时偏移                                            | 原生滚动、触摸、滚轮、键盘和程序化 API 仍为权威                 |

### 平台支持

- 基线实现必须在 Astryx Core 支持的所有浏览器中工作，使用 `ResizeObserver` 和 DOM 滚动几何。CSS 滚动状态查询是未来优化，而非唯一实现，因为本提案提出时 `scroll-state(scrollable: ...)` 在 Firefox 和 Safari 中仍不可用。
- `scrollbar-width`、`scrollbar-color` 和 `scrollbar-gutter` 在 Astryx 浏览器兼容窗口内，构成支持的的原生呈现表面。强制颜色和平台滚动条偏好优先于装饰性匹配；不支持或被忽略的值仍降级到平台滚动条，不改变所有权或访问。
- 浏览器证据必须覆盖 Chromium、Firefox 和真实 Safari。必须包括可用时的经典和覆盖滚动条、LTR 和 RTL、缩放、书写模式变化、隐藏到