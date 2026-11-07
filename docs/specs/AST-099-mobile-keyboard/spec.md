---
schema_version: 4
template_version: 2
kind: system-spec
id: spec:AST-099
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-10-24
phase: accepted
owners: [cixzhang]
affects_architecture: [architecture:react-component-runtime]
affects_families: [family:input-fields]
affects_contributing: []
affects_consumer_docs:
  [
    packages/cli/assets/docs/browser-support.doc.mjs,
    packages/core/src/BottomSheet/BottomSheet.doc.mjs,
  ]
---

# 屏幕键盘系统规范

<!-- path-applicability:v1 -->

```json
{
  "paths": [
    "packages/core/src/BottomSheet/**",
    "packages/core/src/TextInput/**",
    "packages/core/src/TextArea/**",
    "packages/core/src/NumberInput/**",
    "packages/core/src/DateInput/**",
    "packages/core/src/DateTimeInput/**",
    "packages/core/src/TimeInput/**",
    "packages/core/src/Typeahead/**",
    "packages/core/src/Tokenizer/**",
    "packages/core/src/Field/PanelSearchInput.tsx",
    "packages/core/src/Selector/**",
    "packages/core/src/MultiSelector/**",
    "packages/core/src/PowerSearch/**",
    "packages/core/src/CommandPalette/**",
    "packages/core/src/Chat/ChatComposer.tsx",
    "packages/core/src/Chat/ChatComposerInput.tsx",
    "packages/core/src/Chat/ChatLayout.tsx",
    "packages/core/src/keyboard/**",
    "packages/core/src/motion/android/**",
    "packages/core/src/utils/layoutViewport.ts",
    "packages/core/src/utils/layoutViewport.test.ts"
  ]
}
```

<!-- 描述系统本身，而非项目：用现在时，说明它做什么。记录中不写提案、历史、Pull Request 或研究；参见 docs/contributing/spec-writing.md，并在 Pull Request 中报告其评分结果。 -->

## 意图

在手机上打字的人能看到自己正在输入的字段及其周围的界面，位于屏幕键盘上方；字段获得焦点时页面不会突然跳动。每个响应键盘的界面都用同一种方式测量键盘并保持页面不动，因此键盘在底部面板、聊天输入框或搜索字段中的行为一致。Core 输入组件与键盘协同工作：每个唤起键盘的 Core 字段都能在页面中、在 Core 宿主中、以及在应用停靠在底边的栏中保持自己在键盘上方，应用无需开启任何开关。

## 非目标

- 层几何：Popover 及其他锚点模式层如何保持在键盘上方，属于 `spec:AST-059`。
- 硬件键盘访问与焦点顺序，由 `spec:AST-025`、`spec:AST-020` 及各组件记录负责。
- 随键盘移动界面的动画曲线或时序，超出 FR3 的同步与 FR8 的限制范围；在减少动态效果下遵循 `design:reduced-motion`。
- 应用的视口 `<meta>` 元素，由应用负责。
- `@astryxdesign/native`，其中适用平台自身的键盘避让（`spec:AST-082` FR11）。
- 满足本契约的等效内部实现仍然有效。

## 需求

- **FR1 — 所有组件共用一个键盘高度。** 每个相对于屏幕键盘进行定位、调整尺寸或关闭的 Core 和 Lab 组件，都在同一时刻响应同一个键盘高度（FR2），包括 BottomSheet、聊天输入框、置于底部的搜索字段、标签栏搜索、拖动关闭以及任何键盘布局参考线，因此任意两个组件对键盘位置的判断都不会不一致。`architecture:react-component-runtime` INV11 负责该高度的唯一来源，以及 Core 之外的组件如何遵循。
- **FR2 — 平移或缩放无法破坏的高度。** 键盘高度等于布局视口高度减去视觉视口高度，再乘以视觉视口的缩放比例，单位为 CSS 像素。未显示键盘时为 0，包括页面处于双指缩放状态时。页面平移或视觉视口偏移是需要还原的平移（FR4），绝不是更小的键盘。
- **FR3 — 键盘高度保持内部，并保持布局正确。** 任何公开的 CSS 自定义属性、导出或 prop 都不暴露键盘高度。每个组件在自己的元素上应用该高度，例如底部偏移、内边距或最大高度，因此无需运行动画布局即正确：聊天输入框、置于底部的搜索字段和标签栏搜索据此定位，完全展开的面板据此扩展自身的滚动范围。FR9 的停靠随动是唯一例外：Core 按 FR9 的规定移动应用停靠的栏。该值如何到达样式由 `architecture:react-component-runtime` INV11 负责。随键盘运动而叠加的动画层建立在该布局之上，因此布局从不依赖它；FR8 在适用处要求它。当界面随动时，例如输入框、滚动视图和头部，它们沿同一运动同步移动到静止值，不产生逐帧布局工作，并且每个界面只向布局提交一次静止值，作为偏移、内边距或占位，因此不会有内容被隐藏或无法滚动。运动如何共享由 `architecture:react-component-runtime` INV11 负责；任何公开 API 或 CSS 自定义属性都不暴露它。在 iOS 的 Safari 中，键盘只在落下后才上报，此时一个完全展开的面板，其字段因焦点转移到既不唤起键盘也不唤起选择器的控件而失焦，会保持其键盘布局，并在键盘自身隐藏所播放的运动（UIKit 的键盘弹簧）上，从失焦时刻起，以无逐帧 JavaScript 的平移方式将内容移动到其静止滚动位置。视觉视口恢复后，布局撤销、平移清除，一次提交完成，因此不会有任何移动。在减少动态效果下不产生位移，变化遵循 `design:reduced-motion` DR2。
- **FR4 — 焦点绝不平移页面。** 进入键盘感知界面内文本输入控件的每条路径，包括点击、键盘的上一个和下一个按钮、Tab 以及程序化聚焦，都在浏览器不滚动文档的情况下传递焦点，并由界面用自己的滚动容器将控件带入视野。当键盘的 Done 按钮结束输入时，焦点移到界面，使下一次点击以相同方式处理。页面处于滚动锁定期间仍发生的文档滚动，例如返回应用时，在同一帧内还原。
- **FR5 — 移动界面会关闭键盘。** 当持有焦点控件的界面开始移动时，例如在高度之间拖动的面板、拖动关闭或关闭操作，控件失焦以关闭键盘，并且界面保持其键盘布局直到视觉视口恢复，因此内容不会在移动的第一帧跳动。
- **FR6 — 能力检测。** 键盘处理在视觉视口因键盘而收缩、而布局视口保持高度的地方生效。它从平台的视口和键盘能力中检测这一点，绝不从用户代理检测（`spec:AST-013` FR6）。当浏览器还上报屏幕键盘的几何信息时，如 Android 上的 Chrome，高度在那里也来自 FR2，FR8 规定界面如何随键盘运动，适用于任何显示屏幕键盘的设备；提供该上报但不显示屏幕键盘的浏览器读数为 0。非安全上下文的页面得不到该上报，因此 FR8 在那里不适用。
- **FR7 — 仅限顶层文档。** 在 iframe 内，键盘打开时框架的视觉视口不变，因此那里的键盘高度为 0，不会有任何变化。本记录的行为和证据来自顶层文档。
- **FR8 — 上报键盘的浏览器平滑随动。** 在自行上报屏幕键盘几何信息的浏览器中，如 Android 上的 Chrome，键盘高度在那里也来自 FR2 的视觉视口测量（DEC-4），Core 的键盘感知界面在显示和隐藏时随键盘运动：它们随键盘滑动而移动，在已测量的键盘上与其相差不超过一帧，没有一步跳跃，没有界面留在键盘下方，无逐帧 JavaScript，静止时一次布局提交（FR3）。适用两个例外（DEC-6）。在 Android 上的 Chrome 中，键盘首次上报时，内容在一帧内放置，使焦点字段的底部正好位于键盘顶部之上，然后随键盘滑动的其余部分运动。如果上报后约 100–170 ms 字段仍绘制在键盘下方，Chrome 会自行平移页面，因此不能依赖滑动。在 Android 上的 Chrome 中，页面无法预测的隐藏，例如 Back，会在键盘滑动中途上报；内容从该上报起向下随动。在减少动态效果下不随键盘产生位移，变化遵循 `design:reduced-motion` DR2，在触摸屏上为交叉淡化。Lab 界面提交静止值而不随动（`architecture:react-component-runtime` INV11）。Astryx 绝不覆盖应用自身的视口设置。当应用设置 `interactive-widget=resizes-content`（Astryx 记录但不设置）时，Android 上的 Chrome 随键盘收缩布局视口，因此 FR2 读数为 0：键盘感知字段在那里键盘高度为 0 且不随动，因为浏览器已经腾出空间。在其他所有情况下，键盘感知界面随视觉视口高度运动（DEC-4）。没有键盘几何 API、仅收缩视觉视口的 Android 浏览器，如 Firefox for Android，在 FR6 下获得 FR2–FR5。
- **FR9 — 每个唤起键盘的 Core 字段都保持在键盘上方。** 当屏幕键盘打开时，获得焦点的 Core 文本输入字段保持在键盘上方可见，应用无需做任何事，无论它渲染在页面的滚动内容中、Core 宿主（如面板或聊天输入框的停靠区）中，还是应用停靠在视口底边且自身未设置 `translate` 的固定栏中。滚动内容或 Core 宿主中的字段与键盘顶边保持 10 CSS px 的间距，即 `spec:AST-059` FR9 在层下方保持的间距；应用停靠栏中的字段随栏运动，栏的底部与键盘顶边相接。Core 不移动任何其他固定容器（浮在底边之上的、自身设置了 `translate` 的，或占满视口的）；由应用定位。键盘顶部即 FR2 的高度。Core 为显示或移动字段时，页面绝不平移或平移。在 FR9 留给应用的固定容器内（浮在底边之上的、自身设置了 `translate` 的，或占满视口的），若字段与该容器之间有滚动容器，则遵循滚动内容规则：由最近的滚动容器显示它，页面不平移。只有字段与该容器之间没有滚动容器时，才保留浏览器自身的焦点显示，这在 Safari 中可能平移页面。
  - **滚动内容。** 滚动内容中的字段由最近的滚动容器（包括文档）在键盘尺寸上报的同一帧内滚动到位。当该滚动容器无法滚动足够远时，它在键盘显示期间于末尾获得缺失的滚动范围，并在键盘隐藏时失去该范围。其他内容不动。
  - **停靠容器。** 停靠在视口底部的字段，位于底部栏、固定工具栏或面板中，随键盘运动：其停靠容器在键盘打开时上移、关闭时下移，在适用处按 FR8 的随动（DEC-2），在其他情况下（如 iOS 的 Safari）遵循视觉视口高度（FR2）。Core 宿主通过在自己的元素上应用键盘高度来随动（FR3），其中的字段不会被再次移动。
  - **应用拥有的停靠容器。** 应用拥有的容器在固定定位、不占满视口且停靠时也随动：其布局盒（在任何变换之前）位于视口底边。停靠容器的绘制盒一旦移动后无法容纳在键盘上方的空间内，例如全高侧边抽屉或固定头部下方的窗格，则由字段处理：若有滚动容器持有该字段，字段通过它显示，容器不移动；若无滚动容器持有它，容器只移动到焦点字段避开键盘所需的距离，绝不超过容器自身的重叠量，因此即使容器顶部离开屏幕，字段仍保持可见。能容纳的停靠容器上移的距离，从其绘制后的底边（在应用自身施加的任何抬升之后）到键盘顶边，在每次键盘上报后测量，包括焦点留在栏中时键盘高度变化的情况，以及每次焦点移动后，在页面自身的监听器运行之后。因此，应用已抬升到键盘的栏不会被再次抬升，应用部分抬升的栏移动剩余部分，保持不动的栏移动完整重叠量。Core 不移动应用正用 CSS 过渡或动画移动的栏（Core 测量时正在其位置上运行的），因此该栏绝不会被抬升两次。Core 无法看到应用每帧从脚本移动的栏；从脚本抬升栏的应用用自己的 `translate` 定位它（Core 绝不移动该值）以保持其避开键盘。该移动叠加在应用设置的任何变换之上。自身计算出的 `translate`（应用设置的值，而非 Core 的移动）不为 `none` 的栏，例如用 `translate: -50% 0` 居中的栏，不会被移动。栏的静止状态就是该移动，而非对其布局的更改；字段失焦后键盘隐藏，或字段卸载后，该移动消失。Core 不更改应用元素上的其他任何内容。这是 FR3 的唯一例外（DEC-5）。
  - **无 API。** 没有 prop、选项、上下文或其他公开 API 开启、关闭或调整此行为，对于上述位置的字段，应用无需做任何事即可获得。字段获得焦点的每条路径（FR4）都得到相同行为。
  - **覆盖的字段：**
    - TextInput、TextArea 和 NumberInput；
    - DateInput、DateTimeInput 和 TimeInput 的文本输入字段，在其渲染该字段的每种呈现形式中；
    - Typeahead 和 Tokenizer 的输入框（由 BaseTypeahead 渲染），以及 Tokenizer 的面板搜索字段；
    - Selector 和 MultiSelector 的搜索行；
    - PowerSearch 的查询输入框、其值编辑器的文本和数字字段，以及其面板流程的搜索和值字段；
    - CommandPalette 的搜索输入框；
    - ChatComposer 的输入框。

  以后新增的唤起键盘的 Core 字段受同一规则覆盖。不唤起键盘的字段不在其内：无输入模式的只读触摸选择器字段、原生日期或时间选择器、DateRangeInput 的触发器、FileInput、Slider，以及 Selector 或 ComplexSelector 的触发器。Lab 字段遵循 INV11 对 Core 之外组件的规则。

### 平台支持

- 支持的功能/引擎下限：包支持的浏览器（`spec:AST-013` FR3）。FR2–FR5 适用于 iOS 的 Safari（浏览器中和作为主屏幕应用）以及 Firefox for Android；FR8 的随动适用于浏览器自行上报键盘的任何地方，如 Android 上的 Chrome。
- 不支持的行为：无屏幕键盘或无视觉视口支持时，键盘高度为 0，不会有任何变化。服务端渲染期间不渲染任何内容，也不读取视口。
- 浏览器证据：iOS 的 Safari（在 iOS 模拟器中或设备上，顶层文档）证明 FR2–FR5 的 iOS 相关声明；Android 上的 Chrome（Android 模拟器中配 Gboard）证明 FR8；Firefox for Android（模拟器中）证明那里的 FR2–FR5（`spec:AST-013` FR10）；DOM 和视觉视口测试证明 FR1–FR7 的其余部分。顶层文档中的模拟器和仿真器证据是合并的门槛。Android 证据是在确切 head 上的仿真器证据，动画处于正常速度和移除状态，外加慢速设备配置，无需人工设备检查（`spec:AST-094` DEC-27）。键盘模块在发布版本中发布前，需要在物理 iPhone 上通过，包括手机的真实刷新时序（DEC-3）。

## 当前状态影响

- BottomSheet 的键盘 Hook 位于 Core 的键盘模块 `packages/core/src/keyboard/`，即 `architecture:react-component-runtime` INV11 指定的唯一来源。它为完全展开的 Tall 面板实现 FR2、FR4 和 FR5，按能力决定（FR6），并通过面板自身元素上的函数式样式应用高度（FR3）。BottomSheet 使用它（`component:BottomSheet`）。
- `main` 上没有其他组件测量键盘；BaseTypeahead 仅读取视觉视口高度来调整其列表尺寸（Gap 4）。以后响应键盘的组件使用同一高度（FR1）。
- FR9 覆盖的每个字段都获得其在 `main` 上缺失的键盘行为（Gap 2–11）；输入家族引用 FR9（`family:input-fields` FR13）。
- 拥有自己的固定底部栏的应用：键盘打开时保持不动的栏在 FR9 下随动。应用已随键盘移动的栏在两种情况下行为如旧：应用正用 CSS 过渡或动画移动、且 Core 测量时正在运行的栏，Core 留给应用；以及应用用自身 `translate` 从脚本移动的栏，Core 绝不移动。应用已抬升的栏按绘制测量，因此 Core 不移动它；应用仅部分抬升、且无 CSS 运动运行的栏，移动剩余部分，叠加在应用自身变换之上。Core 测量后应用启动的 CSS 抬升可能被抬升两次；应用每帧从脚本在 `transform` 上驱动的抬升也可能被抬升两次，Core 无法看到。无需退出 API。
- 用 `translate` 定位的栏，以及浮在底边之上的固定容器，不随动：Core 将它们留在应用放置的位置。由滚动容器持有的字段仍由该滚动容器显示；字段与该容器之间没有滚动容器时，可能留在键盘下方。希望随动的应用将栏停靠到视口底边，并用 `transform` 定位或不使用 `translate`（后续工作）。停靠容器过高无法容纳在键盘上方时，例如全高侧边抽屉或固定头部下方的窗格，若有滚动容器持有字段则不移动，否则只移动到焦点字段所需的距离。
- `architecture:react-component-runtime` INV11 允许 FR9 的停靠随动：Core 在应用停靠的固定容器（其自身 `translate` 为 `none`）上写入并移除一个 `translate`，等于从其绘制底边到键盘顶边的距离，应用元素上不更改其他内容（DEC-5）。
- 层上限不考虑键盘：`spec:AST-059` 负责层几何，保持 Popover 及其他锚点模式层在键盘上方是对该记录的修订，消费此高度。
- 键盘模块的高度包含 FR2 的缩放项