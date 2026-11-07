---
schema_version: 4
template_version: 2
kind: system-spec
id: spec:AST-109
authority: draft
archive_reason: null
superseded_by: null
approved_by: null
approved_at: null
phase: proposed
owners: [cixzhang]
affects_architecture: [architecture:public-component-api]
affects_families: [family:input-fields]
affects_contributing: []
affects_consumer_docs: [Selector]
---

# 清除值系统规范

<!-- path-applicability:v1 -->

```json
{
  "paths": ["packages/core/src/Selector/**"]
}
```

<!-- 描述系统而非项目：用现在时，说明它做什么。记录中不写提案、历史、Pull Request 或研究；参见 docs/contributing/spec-writing.md，并在 Pull Request 中报告其评分结果。 -->

## 意图

清除 `Selector` 会将其恢复到起点，起点由调用方指定。起点为"所有人"的筛选器清除后回到"所有人"，而不是空白字段。搜索缩小范围时，代表起点的行仍留在列表中，因此用户总能从列表本身回到起点。

## 成员

本规则仅适用于 `Selector`。其值类型为 `string`，其清除选项（FR4）是 `value` 为 `clearValue` 的选项，相等性为字符串相等。`family:input-fields` 的其他所有成员，包括 `Typeahead`、`Tokenizer`、`MultiSelector` 和 `ComplexSelector`，都保留普通布尔 `hasClear`，没有 `clearValue`。

## 非目标

- 与清除分离的重置概念。清除只有一项职责、一个 Prop，`hasClear` 的 `clearValue` 指定其落点。
- 默认值或非受控模式。`Selector` 是受控的，没有 `defaultValue`。
- 搜索匹配与查询归一化。`spec:AST-056` FR4 负责匹配下限。
- `hasClear` 的对象形式用于 `Selector` 以外的任何输入：`Typeahead`、`Tokenizer`、`MultiSelector`、`ComplexSelector`、`PowerSearch`，以及文本、数字、日期、时间和文件输入。
- 满足本契约的等价内部实现仍然有效。
- 内部模块、文件、函数名、算法、数据结构、存储布局、清单、日志、锁、事务协议以及 CI 作业/工作流拓扑属于架构或实现，除非调用方或互操作系统有意将该确切机制作为公共协议依赖。若是，需说明谁依赖它以及为何等价实现无法满足契约。

## 公共契约

```tsx
<Selector
  label="Owner"
  hasClear={{clearValue: 'all'}}
  hasSearch
  value={owner} // string, never null
  onChange={setOwner} // (value: string) => void
  options={[
    {value: 'all', label: 'Everyone', icon: UsersIcon},
    {type: 'divider'},
    ...owners,
  ]}
/>
```

## 需求

- **FR1 — `hasClear={{clearValue}}` 指定清除落点。** `Selector` 的 `hasClear` 接受 `boolean | {clearValue: string}`。`true` 的行为与现在相同：清除报告 `null`。对象形式开启清除，并使所有清除路径通过 `onChange` 和 `changeAction` 报告 `clearValue`：触发器的清除按钮、触发器上的 Delete 和 Backspace，以及选择清除选项（FR4）。`false` 和缺省保持现有含义。
- **FR2 — 类型随对象形式收窄，现有调用方不变。** 使用 `hasClear={{clearValue}}` 时，`value`、`onChange` 和 `changeAction` 取 `string`，不含 `null`。使用 `true`、`false` 或无 `hasClear` 时，类型与已发布版本完全一致，包括 `hasClear` 下的 `null`。
- **FR3 — 处于清除值时无可清除内容。** 当值等于 `clearValue` 时，触发器的清除按钮隐藏，与现在值为空时相同。触发器照常显示该值：该选项的标签和图标，不显示 `placeholder`。
- **FR4 — 清除选项留在列表中。** `value` 等于 `clearValue` 的选项即清除选项。搜索永不隐藏它，包括无匹配的查询——此时空搜索消息渲染在其后。它不算结果：结果数播报和空搜索状态只统计匹配项。查询激活时，默认高亮落在第一个匹配项上，永不落在清除选项上，因此 Enter 不会误选它；方向键可以到达它。它保持调用方在顶层条目中的位置。`clearValue` 不匹配任何选项时不固定任何行，并在开发环境警告。
- **FR5 — 选择清除选项即选择值。** 它使用选项行的结构、尺寸、按压模型、触摸目标、主题目标和呈现方式，关闭触发器上的键入查找匹配其标签。选择它会关闭浮层并将焦点返回触发器（`component:Selector` FR1）。清除按钮的焦点返回与现在相同。
- **FR6 — 禁用、只读、必填和表单保持其含义。** `Selector` 禁用或只读时清除不可用（`component:Selector` FR6）。空 `clearValue` 与现在空值一样无法通过 `isRequired` 校验；非空值则通过。使用 `htmlName` 时，清除后提交的值为 `clearValue`。

### 无障碍契约

- **AR1 — 名称与状态。** 清除选项的无障碍名称为其标签，值等于 `clearValue` 时暴露其选中状态。清除按钮的无障碍名称和工具提示不变（`family:input-fields` FR11）。
- **AR2 — 焦点。** 同 FR5：选择清除选项将焦点返回触发器，清除按钮保持现有焦点返回行为。
- **AR3 — 播报。** 清除不额外播报：触发器读出新值，与任何选择相同。
- **AR4 — 动效。** 此处不新增动效。各浮层自身的动效和减少动效行为适用。

### 平台支持

- 支持的功能/引擎下限：所有受支持的浏览器和渲染器。此处无能力检测。
- 不支持的行为：无。
- 浏览器证据：FR2 的类型测试；名称、选中状态、焦点和播报的 DOM 与无障碍树断言；弹出层和底部面板中按压模型的真实 Chromium 交互测试。

## 当前状态影响

- `family:input-fields` 在其采用表中仅为 `Selector` 列出 `hasClear` 的 `{clearValue}` 形式，并引用本记录。其他所有成员的 `hasClear` 保持布尔。
- `component:Selector` 列出清除选项和 FR3 的清除按钮规则。
- `Selector` 的 `.doc.mjs` 记录 `hasClear` 的对象形式及其类型收窄。
- `Selector` 保留一条内部清除路径；其当前内部名称不约束公共 Prop。

## 验证

| 契约    | 验证                                                          | 代表性状态                                                                       | 变更或失败预期                                                                                     |
| ------- | ------------------------------------------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| FR1     | `Selector` 回调与 Action 套件                                 | 清除按钮、Delete/Backspace、清除选项；`hasClear` 为 `true` 和 `{clearValue}`     | 对象形式下任何清除路径报告非 `clearValue`，或 `true` 下报告非 `null`                               |
| FR2     | `Selector` 类型测试                                           | 无 `hasClear`；`true`；`{clearValue}`                                            | 对象形式接受 `null`，或已发布签名在无对象形式时发生变化                                            |
| FR3     | 触发器渲染套件                                                | 值等于 `clearValue`；不同；空                                                      | 清除按钮在 `clearValue` 时显示，或触发器在该处显示 `placeholder`                                   |
| FR4     | 带 live-region 断言的搜索套件                                 | 无查询；有匹配；无匹配；在搜索输入中按 Enter；`clearValue` 无对应选项            | 该行被过滤、计数、在查询下默认高亮，或被 Enter 选中                                               |
| FR5     | 弹出层和底部面板中的交互套件；真实 Chromium                   | 指针、触摸、键盘；键入查找                                                         | 该行偏离选项行行为，或焦点落在别处                                                                 |
| FR6     | 禁用、只读、必填和表单套件                                    | 禁用；只读；必填配合空和非空 `clearValue`；`htmlName`                            | 清除可达、必填校验错误，或提交值不同                                                               |
| AR1–AR4 | 无障碍树、焦点和播报断言                                      | `Selector` 处于和离开 `clearValue`                                                | 名称或状态缺失、焦点移往别处，或触发额外播报                                                       |

已知验证缺口：这些套件在 `main` 上均不存在。

## 决策日志

### DEC-1 — 返回即清除，落点由调用方在 `hasClear` 上指定

**引用：** `spec:AST-109/DEC-1`
**决策者：** `cixzhang`，`2026-10-28`

清除本就意味着"回到起点"；`hasClear` 的对象形式让调用方指定起点，因此一项职责保留一个 Prop（`architecture:public-component-api` INV21）。该对象是现有开关上的普通配置，不是结构设置（INV18），不新增视觉布尔（INV20），运行时零成本（INV19）。

已否决：单独的重置值或行，因为它与清除共享职责。
已否决：单独的 `clearValue` Prop，因为两个 Prop 会切换同一职责。

### DEC-2 — 清除选项即持有清除值的选项

**引用：** `spec:AST-109/DEC-2`
**决策者：** 待定

该选项已携带清除值的标签和图标，因此它就是留在列表中的行。不存在新的条目类型或标签 Prop。

已否决：在 `options` 中设专用清除条目，因为它会在持有值的选项旁第二次命名清除状态。

### DEC-3 — 类型仅在对象形式下收窄

**引用：** `spec:AST-109/DEC-3`
**决策者：** `cixzhang`，`2026-10-28`

使用 `hasClear={{clearValue}}` 时 `Selector` 永不报告 `null`，因此类型如此声明。使用其他任何 `hasClear` 值时无变化，因此对象形式是增量的。

### DEC-4 — 清除值仅属于 `Selector`

**引用：** `spec:AST-109/DEC-4`
**决策者：** `cixzhang`，`2026-10-29`

`hasClear` 的对象形式和清除选项属于 `Selector`，其筛选器从"所有人"这样的真实值开始。`Typeahead`、`Tokenizer` 及其他所有输入保留普通布尔 `hasClear`。`PowerSearch` 不是选择器。

已否决：对象形式覆盖整个选择器家族，因为只有 `Selector` 需要清除落到真实值。

## 待决问题

- **OQ1 — `hasClear` 的对象形式是否扩展到 `MultiSelector` 或 `ComplexSelector`？**（`human-api`）在裁定前，两者均保留普通布尔 `hasClear`。