---
schema_version: 4
template_version: 2
kind: system-spec
id: spec:AST-075
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-10-15
phase: accepted
owners: [vjeux]
affects_architecture:
  [architecture:interaction-modality, architecture:public-component-api]
affects_families: [family:input-fields, family:overlay-dismissal]
affects_contributing: []
affects_consumer_docs:
  [BaseTypeahead, Typeahead, ChatComposerInput, useCombobox, useMultiCombobox]
---

# 基于调用方渲染字段的 Combobox 系统规范

## 意图

用户在过滤后的列表中选择，同时焦点保持在产品自行渲染的文本字段中：聊天输入框的 textarea、表情网格上方的搜索框、结果替换页面区块的设置搜索框、匹配项内联显示在对话框中的“添加人员”字段。Astryx 为该字段提供与其自带 typeahead 输入相同的 combobox 语义和键盘模型。产品提供字段、行以及列表的渲染位置，并通过 Hook 组合行为；每个此类字段的行为和播报方式都一致。

## 非目标

- 字段的外壳与外观、行的外观以及列表的渲染位置（图层或页面流）由调用方负责。
- 搜索、过滤、防抖和异步数据源由调用方负责。
- 在字段文本中定位查询（例如输入框中触发字符之后的文本）由调用方负责。
- 焦点位于按钮上的 combobox（`Selector`、`MultiSelector`）或位于模态命令界面中的 combobox（`CommandPalette`）保留各自的记录；本规范不为所有 combobox 设定统一的交互模型。
- 满足本契约的等效内部实现仍然有效。

## 要求

- **FR1 — 焦点保持在字段中。** 移动活动选项不得将 DOM 焦点移出字段。字段必须通过 `aria-activedescendant` 指明活动选项，该属性引用一个已渲染的选项；没有活动选项时，字段不得携带 `aria-activedescendant`。
- **FR2 — 展开状态跟随列表。** 列表显示时，字段必须暴露 `aria-expanded="true"` 以及指向 listbox 的 `aria-controls`。列表未显示时，字段必须暴露 `aria-expanded="false"`，且不得引用未渲染的元素。`textarea` 字段（FR3）以相同方式暴露 `aria-controls`，且不携带 `aria-expanded`。
- **FR3 — 角色。** 字段必须暴露可访问名称。`input` 或 contenteditable 字段必须暴露 `role="combobox"`。`textarea` 不允许显式角色，保留其隐式 `textbox` 角色，且必须暴露 `aria-autocomplete="list"`。列表必须是一个具名的 `listbox`，其行为 `option`，活动项上带有 `aria-selected="true"`。listbox 可以渲染在文档中的任意位置，包括页面流。非选项的控件必须渲染在 listbox 之外。
- **FR4 — 按键。** 列表显示时，ArrowDown 和 ArrowUp 必须移动活动选项；在活动选项上按 Enter 必须选中它，且仅在此之后才阻止字段自身的 Enter；Escape 必须隐藏列表，且在同一次按键中不得关闭外层图层（`family:overlay-dismissal`）。列表隐藏时，单行 `input` 字段中的 ArrowDown 在有选项时必须显示列表，与 `component:BaseTypeahead` 对其自身输入的行为一致；在多行字段（`textarea` 或 contenteditable 元素）中，每个按键都必须原样到达字段及其周围环境，因此方向键继续在行间移动光标。列表显示时，Home 和 End 在 OQ2 裁定之前不属于本要求，其他所有按键必须原样到达字段。
- **FR5 — 输入法。** 输入法组合进行中时，按键不得激活 combobox 命令，包括 Escape 的关闭（`family:overlay-dismissal`）。
- **FR6 — 指针。** 指针移到选项上必须使其成为活动项，且不移动 DOM 焦点。按下选项必须选中它，字段必须在按下过程中保持焦点。
- **FR7 — 结果数量。** 当选项因新查询而变化时，选项数量（或没有选项）必须通过共享播报器礼貌播报，而非每个字段各自的 live region，通过字段自身的 `useAnnounce({hasReplace: true})` 实例（`spec:AST-086` FR9），使较新的数量替换尚未播报的旧数量。
- **FR8 — Hook 可作为 Hook 被发现。** 实现本契约的每个公共 Hook 必须由 `astryx hook <name>` 和 `astryx search` 返回，包含其参数、返回字段以及其满足的上述要求，使产品无需先找到使用它的组件即可发现它。

### 平台支持

- 支持的 feature/engine 下限：仓库支持的引擎；无新增 CSS 或 DOM 特性。
- 不支持的行为：无；屏幕阅读器对 contenteditable combobox 上的 `aria-activedescendant` 播报不一致，FR1 在此仍然成立。
- 浏览器证据：FR1–FR3 的 Chromium 可访问性树凭据，以及 FR4–FR6 的真实键盘和指针输入，屏幕阅读器相关声明受 `spec:AST-009` 约束。

## 当前状态影响

- `component:BaseTypeahead` 对其渲染的输入满足 AR1–AR5；本契约将相同义务延伸到调用方渲染的字段。
- `component:ChatComposerInput` 仅通过内部 Hook 在其自身的 contenteditable 字段内为其触发菜单暴露 combobox 语义。
- `useCombobox`（从 `@astryxdesign/core/Selector` 导出）跟踪活动选项、打开和关闭，并处理 Selector 和 CommandPalette 的按键。对于调用方渲染的单行 `input`，在任意具有字符串 `value` 的选项类型上，其 `getFieldProps`、`getListboxProps` 和 `getOptionProps` 承载 FR1–FR6。它尚不支持多行字段（DEC-3），调用方通过自身的 `useAnnounce({hasReplace: true})` 实例播报结果数量（FR7），如 DEC-2 所述。
- `useMultiCombobox`（从 `@astryxdesign/core/MultiSelector` 导出）对 MultiSelector 自身的控件做同样的事。它接受 MultiSelector 的选项类型，将 ARIA 接线留给调用方，且在输入法组合期间不拦截按键。
- `useTypeahead` 已在输入法组合期间忽略按键，`useAnnounce` 是 FR7 所指的共享播报器。
- 两个 combobox Hook 都附带 `astryx hook` 和 `astryx search` 返回的 Hook 文档。`useCombobox` 文档列出其满足的上述要求（FR8）；`useMultiCombobox` 文档不涵盖调用方渲染的字段，也未列出任何要求。

## 验证

| 契约 | 验证 | 代表性状态 | 变异或失败预期 |
| -------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| FR1–FR3  | `spec:AST-021` 下的 Chromium 可访问性树绑定；单元测试 | input、textarea、contenteditable；列表在图层中和在流中 | 焦点离开字段，残留过期的 `aria-activedescendant` 或 `aria-controls`，或 axe 在字段上报告 `aria-allowed-role` 或 `aria-allowed-attr` |
| FR4, FR6 | Chromium 中的真实键盘和指针输入 | 方向键、Enter、对话框内的 Escape、悬停和按下；`input` 和 `textarea` 中隐藏列表上的 ArrowDown | 按键或按下在错误时机选中、关闭或使字段失焦 |
| FR5      | 组合事件测试 | 用 Enter 确认候选选择 | 组合期间按 Enter 选中了选项 |
| FR7      | 共享播报器测试 | 有结果、无结果、新查询、一帧内两次查询 | 无播报，或来自每字段 live region 的播报，或被取代的数量与较新的数量一起播报 |
| FR8      | 针对 core 导出 Hook 的 CLI Hook 发现测试 | DEC-2 中列出的每个公共 Hook；从组件入口点导出的 Hook | combobox Hook 导出时没有 Hook 文档，或 `astryx hook <name>` 否认其存在 |

## 决策日志

### DEC-1 — 调用方渲染字段和列表；Astryx 拥有语义和按键

**参考：** `spec:AST-075/DEC-1`
**决策者：** `cixzhang`，`2026-10-15`

已经渲染输入框、搜索框或对话框字段的产品无法在不替换该字段并将其列表移入图层的情况下采用 typeahead，因此它手工接线 combobox 模式，而每个副本在 FR1–FR7 所固定的细节上各不相同。将字段和列表留给调用方、语义由 Astryx 拥有后，每个此类界面都满足一个经过测试的契约。

已否决：要求产品的字段必须是 typeahead 自身的输入——textarea 或 contenteditable 输入框无法做到，且流内列表不是图层。

### DEC-2 — 契约以可组合 Hook 形式发布，构建于 `useCombobox` 之上

**参考：** `spec:AST-075/DEC-2`
**决策者：** `cixzhang`，`2026-10-15`

产品通过组合 Hook 在其自身字段上构建 combobox，与 Astryx 构建自身的方式相同：`useCombobox` 负责活动选项、展开状态和按键，从 Selector 的选项类型泛化到任意选项列表和调用方渲染的字段；输入法守卫 `useTypeahead` 已适用；`useAnnounce` 负责结果数量。Astryx 自身的 combobox 组合相同的 Hook，因此一个经过测试的实现支撑本规范涵盖的每个界面，且每个 Hook 都作为 Hook 记录文档（FR8）。泛化 `useCombobox` 使其已发布的选项和返回字段对现有调用方保持不变；新输入是可选的补充。组件仍是常见场景的便捷路径；Hook 是其他所有字段的路径。

已否决：新建组件，或让 `BaseTypeahead` 接受调用方的字段，这会将行为锁在一个组件的标记内。

### DEC-3 — 隐藏列表仅在单行字段中通过 ArrowDown 重新打开

**参考：** `spec:AST-075/DEC-3`
**决策者：** `cixzhang`，`2026-10-15`

在单行字段中，方向键没有光标工作要做，因此 ArrowDown 重新打开隐藏的列表，这是 Astryx 自身 typeahead 输入已提供的键盘模型。在多行字段中，方向键在行间移动光标，这是撰写消息的人所需要的，因此列表隐藏时它们到达字段。

已否决：列表隐藏时将所有按键都留给所有字段，这会使单行搜索字段除了打字外没有键盘方式回到其列表。

## 开放问题

- **OQ2 — Home 和 End**（`human-design`）：在调用方渲染的字段中，它们要么继续移动光标，要么移动到第一个和最后一个选项，如 `component:BaseTypeahead` FR5 对其自身输入的行为。
- **OQ3 — 二维列表**（`human-api`）：选项网格（例如表情选择器）属于本契约还是单独的契约。