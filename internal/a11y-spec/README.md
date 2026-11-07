# @astryxdesign/a11y-spec

可复用的无障碍规范测试：每个已采用的控件模式对应一份可追溯至标准的契约，并绑定到实现该模式的组件。**内部使用，不发布。**

契约的编写方式见 [`docs/specs/AST-020`](../../docs/specs/AST-020/spec.md)，现有组件测试如何迁移进契约见 [`docs/specs/AST-021`](../../docs/specs/AST-021/spec.md)。请先阅读这两份文档；本文件说明实现它们的代码。

## 为什么用契约代替更多组件测试

以 switch 模式为例。每个采用该模式的组件都承担相同的义务：控件被报告为 switch；其名称不随状态变化；开/关状态对外暴露且与渲染一致；指针和键盘都能开启和关闭；按下后滑出并在别处释放会被撤销；状态变化时焦点保持不动；焦点能进入和离开控件。逐组件编写意味着每次都要重新解读 WCAG 和 APG，而解读会漂移。

因此模式只写一次，以数据形式存在，组件绑定到它。绑定声明自己有哪些状态、每个状态应是什么；契约向浏览器询问它实际暴露了什么。

## 结构

```
src/
├── contract.ts    Expectation、PatternContract、definePattern（Schema 校验）
├── checklist.ts   每个模式都必须回答的完整性维度
├── harness.ts     Harness/Subject 接缝与证据层词汇
├── check.ts       checkAccessibilitySpec — 供报告/变异证明使用的底层结果
├── expect.ts      expectAccessibilitySpec — 面向组件的渲染 + 主体断言
├── report.ts      分离的事实，以及基于这些事实的校验
├── harness/
│   ├── jsdom.ts       观察单元 + DOM。拒绝更高层的一切。
│   └── chromium.ts    观察 DOM + 无障碍树 + 真实浏览器。
├── spoken.ts      可见标签如何与计算名称比较
├── storybook.ts   基于已构建 Storybook 的静态服务器，供浏览器通道使用
├── playwright.ts  每个浏览器 spec 的基础测试：上下文不访问远程主机
└── patterns/
    ├── radio-group.*        单选组模式，同样四个文件
    ├── checkbox.*           复选框模式，同样四个文件
    ├── switch.*             开关模式，同样四个文件
    ├── button.*             按钮模式，同样四个文件
    ├── text-input.*         原生文本输入模式，同样四个文件
    ├── modal-dialog.*       原生模态对话框模式，同样四个文件
    ├── status-message.*     实时区域与进度状态机制
    ├── tabs.*               显式水平 ARIA Tabs 语义
    ├── listbox.*            listbox、group 和 option 语义
    ├── spinbutton.*         数值角色、值、边界、状态和方向键步进
    ├── disclosure.*         独立 disclosure 状态/内容语义
    ├── menu-button.*        控件的 button 角色和名称、aria-haspopup、展开状态、controls 以及键盘打开
    ├── menu.*               已显示菜单的角色和名称、菜单项角色/名称/选中/禁用状态，以及方向键/Home/End/Enter/Escape/Tab 键（typeahead 为建议项）
    ├── toolbar.*            toolbar 的角色和名称、方向、内部控件、单一 Tab 停靠点，以及方向键（循环、RTL 镜像、垂直时 Down/Up）和 Home/End 键
    ├── tooltip.*            tooltip 的角色、由它描述的触发器、键盘焦点显示它、Escape 关闭它、悬停保持它（blur、hover-show 和 pointer-leave 为建议项）
    ├── window-splitter.*    可聚焦调整分隔条的角色、名称、位置/边界、窗格关系，以及方向键/Enter 键（Home/End 为建议项）
    ├── grid.*               复合 grid 的角色、名称、行/单元格结构、单一 Tab 停靠点、选择，以及方向键/Home/End/Enter 键
    ├── alert-dialog.*       确认界面的 alertdialog 角色、标题/消息引用、模态性、初始焦点、Escape 和点击旁侧关闭
    ├── carousel.*           非轮播集合、幻灯片、屏幕外和焦点语义
    ├── tree-view.*          树的角色、名称、展开状态、单一 Tab 停靠点，以及方向键/Home/End/Enter 或 Space/typeahead 移动
    ├── slider.*             滑块手柄的角色、名称、值、文本、方向，以及方向键/Page/Home/End 键（RTL 镜像方向键）
    ├── progressbar.*        进度指示器的角色和名称、值及范围或不确定状态、值文本，以及无可聚焦内容的子树
    ├── accordion.*          组的标题角色和名称、面板状态、单展开折叠、页面 Tab 顺序
    ├── breadcrumb.*         地标、列表轨迹、当前页面和分隔符语义
    ├── combobox.*           控件标识、状态、弹出层和活动后代语义
    ├── landmark.*           单个组件拥有的地标区域的角色、名称和内容边界
    ├── link.*               单个可用链接的角色、名称、当前状态、焦点和激活
    └── table.*              静态数据表的角色和名称及其表头和单元格角色；Table 模块组合各自的语义
```

## 模式

| 模式             | 采用自                                                                                                                                                   | 绑定方                                                                                                                                                                                                                                                                                                   |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `radio-group`    | [APG 单选组](https://www.w3.org/WAI/ARIA/apg/patterns/radio/)                                                                                            | RadioList、SegmentedControl；DropdownMenu 单选菜单项的角色/状态部分                                                                                                                                                                                                                                      |
| `checkbox`       | [APG 复选框](https://www.w3.org/WAI/ARIA/apg/patterns/checkbox/)                                                                                         | CheckboxInput、CheckboxListItem、DropdownMenuCheckboxItem、SelectableCard                                                                                                                                                                                                                                |
| `switch`         | [APG 开关](https://www.w3.org/WAI/ARIA/apg/patterns/switch/)                                                                                             | Switch                                                                                                                                                                                                                                                                                                   |
| `button`         | [APG 按钮](https://www.w3.org/WAI/ARIA/apg/patterns/button/)                                                                                             | Button、IconButton、ClickableCard、SideNavCollapseButton、ChatSendButton                                                                                                                                                                                                                                 |
| `toggle-button`  | [APG 按钮模式](https://www.w3.org/WAI/ARIA/apg/patterns/button/)中的切换按钮语义                                                                         | ToggleButton，包括独立和分组状态                                                                                                                                                                                                                                                                         |
| `text-input`     | 原生 HTML 控件和 [WAI-ARIA textbox](https://www.w3.org/TR/wai-aria-1.2/#textbox)                                                                         | TextInput、TextArea                                                                                                                                                                                                                                                                                      |
| `modal-dialog`   | [APG 对话框（模态）](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)                                                                             | Dialog                                                                                                                                                                                                                                                                                                   |
| `status-message` | [WCAG 2.2 状态消息](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html)                                                                    | Toast、FieldStatus、Spinner、ChatSystemMessage、ChatTypingIndicator、ProgressBar                                                                                                                                                                                                                         |
| `tabs`           | [APG Tabs](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/)                                                                                               | 显式 `role="tablist"` 的 TabList、Tab 和调用方编写的 tabpanel                                                                                                                                                                                                                                            |
| `listbox`        | [WAI-ARIA 1.2 Listbox](https://www.w3.org/TR/wai-aria-1.2/#listbox) 和 WCAG 2.2 语义                                                                     | Selector 和 MultiSelector 弹出层 listbox、group 和 option 部分                                                                                                                                                                                                                                           |
| `spinbutton`     | WCAG 2.2 语义和 [APG Spinbutton](https://www.w3.org/WAI/ARIA/apg/patterns/spinbutton/)                                                                   | NumberInput                                                                                                                                                                                                                                                                                              |
| `disclosure`     | [APG Disclosure](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)                                                                                   | 独立 Collapsible 触发器及其受控内容                                                                                                                                                                                                                                                                      |
| `menu-button`    | [APG Menu Button](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/) 必需部分；Astryx 提供的可选 ArrowDown/ArrowUp 打开为建议项                      | 锚定 DropdownMenu 和 MoreMenu 触发器：button 角色和名称、`aria-haspopup="menu"`、`aria-expanded`、`aria-controls`、Enter/Space 在第一个启用项上打开（建议项：ArrowDown 在第一个，ArrowUp 在最后一个）                                                                                                     |
| `menu`           | [APG Menu 和 Menubar](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/) 菜单；方向键（循环）和 Home/End 按 AST-020 FR15；typeahead 为建议项             | DropdownMenu 指针菜单：命名 `menu`、菜单项角色/名称、`aria-checked`、`aria-disabled`、循环方向键、Home/End、Enter 和 Escape 返回触发器、Tab 移出（建议项：typeahead）                                                                                                                                     |
| `toolbar`        | [APG Toolbar](https://www.w3.org/WAI/ARIA/apg/patterns/toolbar/)；方向键（循环）和 Home/End 按 AST-020 FR15                                              | Toolbar：命名 `toolbar`、`aria-orientation`、内部控件、单一 Tab 停靠点从第一个启用控件进入、前/后方向键（RTL 镜像，垂直时 Down/Up）循环、Home/End                                                                                                                                                         |
| `tooltip`        | [APG Tooltip](https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/) 角色和关系按 AST-020 FR15；WCAG 1.4.13；三行为建议项                                     | Tooltip、useTooltip：`tooltip` 角色、`aria-describedby` 和计算描述、焦点显示它（焦点保持）、Escape 关闭、悬停时在触发器和 tooltip 上保持（建议项：blur 隐藏、hover 显示、指针离开隐藏）                                                                                                                   |
| `accordion`      | [APG Accordion](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/) 必需部分                                                                            | CollapsibleGroup 标题：button 角色和名称、展开状态、受控面板、Enter/Space、单展开同级折叠、页面 Tab 顺序、无方向键移动（单展开折叠、分组 Tab 顺序、无方向键移动在 `component:CollapsibleGroup` AR4 成为当前之前为建议项）                                                                              |
| `breadcrumb`     | WCAG 2.2 语义；[APG Breadcrumb](https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/) 作为上下文                                                         | Breadcrumbs 地标、列表轨迹、当前页面状态和分隔符                                                                                                                                                                                                                                                         |
| `combobox`       | WCAG 2.2 + [WAI-ARIA 1.2 Combobox](https://www.w3.org/TR/wai-aria-1.2/#combobox)；APG 作为上下文                                                         | Selector 触发器标识、状态、弹出层和活动后代关系                                                                                                                                                                                                                                                          |
| `landmark`       | WCAG 2.2 语义；[APG 地标区域](https://www.w3.org/WAI/ARIA/apg/patterns/landmarks/) 作为上下文                                                             | LayoutHeader、LayoutContent、LayoutFooter 和 LayoutPanel 调用方声明的地标                                                                                                                                                                                                                                |
| `slider`         | [APG Slider](https://www.w3.org/WAI/ARIA/apg/patterns/slider/) 角色、值、名称、方向和按键                                                               | Slider 手柄：角色、名称、now/min/max、值文本、方向、方向键（RTL 镜像）、Page、Home/End、禁用时保持值                                                                                                                                                                                                     |
| `progressbar`    | WCAG 2.2 4.1.2 和 [WAI-ARIA 1.2 progressbar](https://www.w3.org/TR/wai-aria-1.2/#progressbar)；不存在 APG 模式                                          | ProgressBar、Lab CircularProgress：progressbar 角色和名称、now/min/max、不确定状态不声明值、值文本、无可聚焦后代（目标标记位于进度条旁）                                                                                                                                                                 |
| `treeview`       | [APG Tree View](https://www.w3.org/WAI/ARIA/apg/patterns/treeview/) 结构和键盘模型                                                                       | TreeList 行：tree/treeitem/group、aria-expanded、level/position、单一 Tab 停靠点、Up/Down、Right/Left（RTL 镜像）、Home/End、Enter 或 Space、typeahead                                                                                                                                                   |
| `carousel`       | [APG Carousel](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/) 和 WCAG 2.2 语义                                                                      | Carousel 容器、幻灯片、滚动区域和稳定的导航控件焦点                                                                                                                                                                                                                                                      |
| `alertdialog`    | [APG Alert Dialog](https://www.w3.org/WAI/ARIA/apg/patterns/alertdialog/) 角色、引用、模态性、焦点、关闭                                                 | AlertDialog：alertdialog 角色和名称、指向消息的 aria-describedby、模态、Cancel 优先（建议项）、Escape 关闭、点击旁侧惰性（建议项）；模态机制通过 Dialog 保留在 modal-dialog 契约中                                                                                                                     |
| `grid`           | [APG Grid](https://www.w3.org/WAI/ARIA/apg/patterns/grid/) 角色、结构、名称、选择和 data-grid 按键                                                       | Calendar 月网格：grid 角色/名称、行和 gridcell/columnheader 结构、单一 Tab 停靠点、aria-selected/aria-multiselectable、按阅读顺序的方向键、Home/End、Enter 选择（建议项）                                                                                                                                 |
| `windowsplitter` | [APG Window Splitter](https://www.w3.org/WAI/ARIA/apg/patterns/windowsplitter/) 角色、值、名称、关系和按键                                              | ResizeHandle：separator 角色/名称、now/min/max、方向、指向窗格的 aria-controls、方向键、Enter 折叠/恢复、Home/End、禁用时保持位置                                                                                                                                                                        |
| `link`           | WCAG 2.2 语义；[APG Link](https://www.w3.org/WAI/ARIA/apg/patterns/link/) 角色和 Enter 机制                                                              | Link，包括 LinkProvider 和 `as` 路由组件                                                                                                                                                                                                                                                                 |
| `table`          | [APG Table](https://www.w3.org/WAI/ARIA/apg/patterns/table/) 角色和名称按 AST-020 FR15；无键盘（模式未定义）；模块添加各自的键盘                         | Table，在数据驱动和 children 模式下：table 角色和名称、table 内的 `row`/`columnheader`/`rowheader`/`cell` 角色；排序模块仅在已排序表头上添加 `aria-sort`                                                                                                                                                |

`progressbar` 契约规定单个渲染的进度指示器在每个状态下暴露什么：progressbar 角色和操作名称（绝不是 meter 或 status）、值在其范围内、进度未知时无值或值文本、格式化值文本，以及键盘无法落点的子树（因为 progressbar 的后代是呈现性的）。该暴露在进度和完成更新后是否保持焦点不动，由 `status-message` 契约负责；钳制、环形几何和目标标记自身的 Tooltip 触发器由组件负责。Spinner 暴露 status 而非 progressbar，因此只绑定 `status-message`。

toggle-button 契约负责按钮语义的持久操作扩展：按下状态必须暴露、可通过指针、Enter 和 Space 逆转、在中断按下时保持稳定、不可用时惰性。它还检查状态变化时焦点保持，并将 APG 标签稳定性报告为建议项，因为当前 Astryx 权威记录未将该 APG 专有细节设为门禁。ToggleButtonGroup 的命名、选择协调、Action 排序、待定反馈和渲染外观保留其现有的组件、家族和视觉负责人。

`spinbutton` 契约负责 NumberInput 的必需角色、持久名称、已提交数值、可选边界和格式化值文本、禁用和只读暴露，以及键盘可达性。ArrowUp 和 ArrowDown 步进记录为建议性 APG 证据，因为当前没有 Astryx 权威记录将这些机制采纳为共享要求。类型化草稿解析、格式化、提交/钳制策略、回调、可选按钮和日期/时间段保留其现有负责人。

disclosure 契约仅负责 disclosure 专有状态、可选的触发器到内容关系、同步可见性，以及完整的指针、Enter 和 Space 转换。通用角色、命名、焦点导航和不可用按钮语义保留在现有 `button` 契约中。首次迁移仅绑定独立 Collapsible 状态；CollapsibleGroup 协调以及 Accordion、Table 和 SideNav 的采用不在本契约范围内。

breadcrumb 契约负责命名导航地标、列表轨迹、可选的当前页面状态和装饰性分隔符暴露。其首个 Breadcrumbs 绑定覆盖显式和自动当前页面选择、自定义地标标签、RTL/辅助呈现，以及有意的无当前页面退出。Link、操作按钮和同级菜单语义保留在各自模式中；APG Breadcrumb 页面仍作为上下文，因为当前没有 Astryx 记录将其采纳为额外必需门禁。

combobox 契约负责命名控件、展开/忙碌/只读状态、其与渲染弹出层的关系，以及声明的活动后代。其首个 Selector 绑定覆盖关闭、打开、只读、忙碌和底部抽屉触发器状态，以及带活动选项的搜索输入。当绑定自身的模态对话框弹出层使 combobox 惰性时，浏览器正确将其排除在无障碍树之外，因此标识