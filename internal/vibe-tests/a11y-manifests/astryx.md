# Astryx 无障碍清单

**来源：** packages/core/src/\*/README.md
**生成时间：** 2026-02-05T18:03:04.596Z

## Avatar

**内置无障碍支持：**

- 无障碍：支持正确的 role 和 aria-label

## Button

**内置无障碍支持：**

- 变体：`primary`、`secondary`、`ghost`、`destructive`
- 焦点可见：提供无障碍焦点轮廓，颜色随变体变化

**开发者需提供：**

- 为纯图标按钮提供 aria-label
- 为切换按钮使用 aria-pressed

**说明：**

- `ButtonVariant` 类型通过 `keyof typeof variants` 从 `variants` StyleX 对象派生
- destructive 变体的焦点轮廓颜色使用 `colorTokens.negative`

## CheckboxInput

**内置无障碍支持：**

- 无障碍：始终包含标签（可视觉隐藏）
- 支持视觉隐藏标签（屏幕阅读器仍可访问）

**说明：**

- 标签可点击，并通过 `htmlFor`/`id` 与输入框正确关联
- 标签为必填 Prop——组件负责标签关联

## DateInput

**内置无障碍支持：**

- 日历弹出层：点击图标或使用键盘打开日历选择器
- 无障碍：完整键盘导航、焦点捕获、屏幕阅读器支持
- 支持视觉隐藏标签（屏幕阅读器仍可访问）

**说明：**

- 标签为必填 Prop——组件负责标签关联

## Field

**内置无障碍支持：**

- 无障碍：标签通过 htmlFor/id 与输入框正确关联
- 使用 aria-describedby 关联描述
- 支持视觉隐藏标签（屏幕阅读器仍可访问）

**说明：**

- 标签始终为无障碍而渲染；使用 `isLabelHidden` 视觉隐藏
- 隐藏标签使用 CSS 技术，屏幕阅读器仍可访问
- 提供描述时渲染描述；若同时提供 `descriptionID`，描述元素会获得该 ID 用于 `aria-describedby` 关联
- 可选/必填文本与标签显示在同一行

## Icon

**内置无障碍支持：**

- 主题颜色：颜色变体映射到 Astryx 图标颜色令牌
- 无障碍：图标默认对屏幕阅读器隐藏（aria-hidden）

**说明：**

- 默认使用 `aria-hidden="true"`，因为图标通常为装饰性
- 对于有意义的图标，设置 `aria-hidden={false}`、`role="img"` 和 `aria-label`

## Selector

**内置无障碍支持：**

- 触发器使用 `role="combobox"`
- 下拉列表使用 `role="listbox"`
- 分组使用 `role="group"`
- 焦点使用 `aria-activedescendant`
- 支持视觉隐藏标签（屏幕阅读器仍可访问）

**键盘操作：**

- ↑↓ 导航，Enter/Space 选择，Escape 关闭，Home/End 跳转，A-Z 首字母定位。

## TextArea

**内置无障碍支持：**

- 无障碍：标签通过 htmlFor/id 与 textarea 正确关联
- 支持视觉隐藏标签（屏幕阅读器仍可访问）

**说明：**

- 使用 `useId` Hook 实现无障碍的标签-textarea 关联
- 标签始终为无障碍而渲染；使用 `isLabelHidden` 视觉隐藏
- 隐藏标签使用 CSS 技术，屏幕阅读器仍可访问
- 包装 `Field` 组件以处理标签、描述和可选/必填
- 可选/必填文本与标签显示在同一行
- 标签为必填 Prop——组件负责标签关联

## TextInput

**内置无障碍支持：**

- 无障碍：标签通过 htmlFor/id 与输入框正确关联
- 为错误状态设置 aria-invalid
- 支持视觉隐藏标签（屏幕阅读器仍可访问）

**说明：**

- 使用 `useId` Hook 实现无障碍的标签-输入框关联
- 标签始终为无障碍而渲染；使用 `isLabelHidden` 视觉隐藏
- 隐藏标签使用 CSS 技术，屏幕阅读器仍可访问
- 包装 `Field` 组件以处理标签、描述和可选/必填
- 可选/必填文本与标签显示在同一行
- 标签为必填 Prop——组件负责标签关联

## theme

**内置无障碍支持：**

- 主题提供者：包装应用以提供 CSS 变量