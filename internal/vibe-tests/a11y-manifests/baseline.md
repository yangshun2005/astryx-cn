# 基线（Radix/shadcn）无障碍清单

**来源：** https://www.radix-ui.com/primitives/docs/overview/accessibility
**生成时间：** 2026-02-05T18:03:04.623Z

## Button

**内置无障碍支持：**

- 原生 button 元素，具有正确的 role
- 键盘激活（Enter/Space）
- 焦点管理

**键盘操作：**

- Enter/Space 激活

**开发者必须提供：**

- 为纯图标按钮提供 aria-label
- 切换按钮使用 aria-pressed
- 控制可展开内容的按钮使用 aria-expanded

**备注：**

- 基于原生 <button> 元素

## Dialog

**内置无障碍支持：**

- 焦点陷阱——焦点保持在对话框内
- Escape 键关闭对话框
- 对话框上设置 aria-modal="true"
- aria-labelledby 指向标题
- aria-describedby 指向描述
- 关闭时将焦点返回触发元素
- 打开时阻止 body 滚动

**键盘操作：**

- Escape 关闭
- Tab/Shift+Tab 在内部导航

**开发者必须提供：**

- 提供 DialogTitle 供 aria-labelledby 使用
- 提供 DialogDescription 供 aria-describedby 使用（或使用 aria-describedby={undefined}）

**备注：**

- WAI-ARIA Dialog 模式

## DropdownMenu

**内置无障碍支持：**

- 内容上设置 role="menu"
- 菜单项上设置 role="menuitem"
- 方向键导航
- 输入首字母跳转到菜单项
- 关闭时焦点返回触发元素
- Escape 关闭菜单
- 支持子菜单，并具有正确的 aria-haspopup

**键盘操作：**

- 方向键下/上导航
- Enter/Space 选择
- Escape 关闭
- 方向键右/左进入子菜单
- A-Z 首字母跳转

**备注：**

- WAI-ARIA Menu 模式

## Select

**内置无障碍支持：**

- 触发元素上设置 role="combobox"
- 内容上设置 role="listbox"
- 选项上设置 role="option"
- 触发元素上设置 aria-expanded
- 使用 aria-activedescendant 管理焦点
- 方向键导航
- 支持首字母跳转

**键盘操作：**

- 方向键下/上导航
- Enter/Space 选择
- Escape 关闭
- A-Z 首字母跳转
- Home/End 跳转

**开发者必须提供：**

- 提供无障碍标签（Label 组件或 aria-label）

**备注：**

- WAI-ARIA Combobox 模式（仅选择型）

## Checkbox

**内置无障碍支持：**

- role="checkbox"
- aria-checked 状态
- 键盘激活（Space）
- 支持不确定状态

**键盘操作：**

- Space 切换

**开发者必须提供：**

- 提供无障碍标签（Label 组件或 aria-label）

**备注：**

- WAI-ARIA Checkbox 模式

## Switch

**内置无障碍支持：**

- role="switch"
- aria-checked 状态
- 键盘激活（Space）

**键盘操作：**

- Space 切换

**开发者必须提供：**

- 提供无障碍标签（Label 组件或 aria-label）

**备注：**

- WAI-ARIA Switch 模式

## Tabs

**内置无障碍支持：**

- 列表上设置 role="tablist"
- 触发元素上设置 role="tab"
- 内容上设置 role="tabpanel"
- 活动标签上设置 aria-selected
- 标签之间方向键导航
- 聚焦时自动激活（或手动模式）

**键盘操作：**

- 方向键左/右导航标签
- Home/End 跳转到第一个/最后一个标签
- Tab 进入面板

**备注：**

- WAI-ARIA Tabs 模式

## Tooltip

**内置无障碍支持：**

- aria-describedby 将触发元素与内容关联
- Escape 关闭
- 显示前延迟（可配置）
- 触摸友好行为

**键盘操作：**

- Escape 关闭

**开发者必须提供：**

- 确保触发元素可聚焦（使用 asChild 配合可聚焦元素）

**备注：**

- 内容应为补充信息，而非必要信息

## Popover

**内置无障碍支持：**

- 焦点管理
- Escape 关闭
- 点击外部关闭
- 触发元素上设置 aria-haspopup

**键盘操作：**

- Escape 关闭
- Tab 在内部导航

**开发者必须提供：**

- 在弹出内容中妥善管理焦点

**备注：**

- 默认非模态——焦点不被陷阱捕获

## AlertDialog

**内置无障碍支持：**

- 焦点陷阱
- aria-modal="true"
- role="alertdialog"
- aria-labelledby 指向标题
- aria-describedby 指向描述
- 默认禁用 Escape（需要明确操作）

**键盘操作：**

- Tab/Shift+Tab 在内部导航

**开发者必须提供：**

- 提供 AlertDialogTitle
- 提供 AlertDialogDescription
- 至少包含一个可聚焦的操作按钮

**备注：**

- WAI-ARIA AlertDialog 模式——用于重要中断

## Progress

**内置无障碍支持：**

- role="progressbar"
- aria-valuenow
- aria-valuemin
- aria-valuemax

**开发者必须提供：**

- 提供 aria-label 或 aria-labelledby 以说明上下文

**备注：**

- 非百分比值使用 aria-valuetext

## Slider

**内置无障碍支持：**

- role="slider"
- aria-valuenow、aria-valuemin、aria-valuemax
- 方向键调整
- Home/End 跳到最小/最大值
- Page Up/Down 进行较大步进

**键盘操作：**

- 方向键左/下减小
- 方向键右/上增大
- Home 跳到最小值
- End 跳到最大值
- Page Up/Down 进行较大步进

**开发者必须提供：**

- 提供 aria-label 以说明上下文

**备注：**

- WAI-ARIA Slider 模式

## Accordion

**内置无障碍支持：**

- 面板上设置 role="region"（展开时）
- 触发元素上设置 aria-expanded
- aria-controls 将触发元素与面板关联
- 触发元素之间方向键导航

**键盘操作：**

- Enter/Space 切换
- 方向键下/上导航
- Home/End 跳转

**备注：**

- WAI-ARIA Accordion 模式

## RadioGroup

**内置无障碍支持：**

- role="radiogroup"
- 选项上设置 role="radio"
- aria-checked 状态
- 方向键导航
- 漫游 tabindex

**键盘操作：**

- 方向键导航并选择
- Tab 进入/离开组

**开发者必须提供：**

- 在组上提供 aria-label 或 aria-labelledby

**备注：**

- WAI-ARIA Radio Group 模式

## Label

**内置无障碍支持：**

- 原生 label 元素
- 点击标签聚焦关联控件
- 正确的 htmlFor 关联

**备注：**

- 始终与表单控件配合使用

## Input

**内置无障碍支持：**

- 原生 input 元素

**开发者必须提供：**

- 提供无障碍标签（Label 组件、aria-label 或 aria-labelledby）
- 错误/帮助文本使用 aria-describedby
- 错误状态设置 aria-invalid

**备注：**

- shadcn Input 是带样式的原生 input

## Card

**开发者必须提供：**

- 若可点击，使用 button 或链接元素，而非在 div 上使用 onClick
- 若可交互，提供适当的 role

**备注：**

- Card 纯属视觉元素——无语义含义

## Badge

**开发者必须提供：**

- 通过 aria-label 或周围文本提供上下文
- 状态徽章考虑使用 aria-live 进行动态更新

**备注：**

- Badge 纯属视觉元素——无语义含义

## Avatar

**开发者必须提供：**

- 在 AvatarImage 上提供 alt 文本
- AvatarFallback 为屏幕阅读器提供回退内容

**备注：**

- 装饰性头像应设置空的 alt=""

## Skeleton

**开发者必须提供：**

- 在父容器上设置 aria-busy="true"
- 考虑使用 aria-live 区域播报加载状态

**备注：**

- Skeleton 是装饰性的——加载状态应单独播报

## Table

**内置无障碍支持：**

- 原生 table 元素，具有正确的结构
- th 元素用于表头
- 正确的表格语义

**开发者必须提供：**

- 使用 TableCaption 提供表格描述
- 在表头上使用 scope="col" 或 scope="row"

**备注：**

- 使用原生表格语义，避免 role="presentation"