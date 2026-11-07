---
schema_version: 4
template_version: 2
kind: system-spec
id: spec:AST-087
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-10-22
phase: accepted
owners: [cixzhang, imdreamrunner]
affects_architecture:
  [
    architecture:component-theming-surface,
    architecture:icon-resolution-and-component-slots,
    architecture:theme-application,
    architecture:theme-authoring-contract,
    architecture:theme-compilation,
  ]
affects_families: []
affects_contributing: []
affects_consumer_docs: [theme, Icon]
---

# Icon 变体、主题尺寸与组件图标插槽系统规范

<!-- 描述系统本身，而非项目：用现在时，说明它做什么。记录中不写提案、历史、Pull Request 或研究；参见 docs/contributing/spec-writing.md，并在 Pull Request 中报告其评分结果。 -->

## 意图

主题决定图标的外观：

- `Icon` 接受可选的 `variant`。主题一次性列出它支持的变体名称，主题图标条目可以是最终尺寸与该变体的纯函数，因此主题可将两者映射到其图标库自有的图形或 Props。
- 主题通过 Icon 样式表目标设置每个 Icon 尺寸的尺寸值。
- 组件将可主题化的图标暴露为 `componentIcons` 插槽，插槽条目可根据组件的类型化状态解析，因此主题可决定按下、选中或禁用状态的图标外观。

## 非目标

- 外观或字重轴、按图标的能力声明、自适应变体树，或由 Astryx 定义的变体名称。
- 新的 Icon 尺寸名称。
- 额外的组件图标映射、图标角色概念、按状态嵌套的映射，或复合状态键。
- 直接传给 `Icon` 的图标组件的变体。它们按原样渲染，永不接收 `variant`；Icon 的目标、尺寸和颜色仍对其生效。
- 替换组件通过指示器绘制的有状态控件标记。这些仍归 `defineTheme({indicators})`（FR15）管理。
- 组件自有图标框如何跟随主题尺寸，以及哪些组件字形通过 `Icon` 渲染。
- DOM 或 SVG 测量、SVG 合成，或为单个请求混合两个来源的图形。
- 满足本契约的等效内部实现仍然有效。

## 需求

### 变体与主题条目

- **FR1 — 主题图标条目是节点或请求的函数。**
  `DefineThemeInput.icons` 和进程级注册对语义键或命名空间键接受 React 节点，或接收 `{size, variant}` 的函数，其中 `size` 是最终 Icon 尺寸，`variant` 是活动主题列出的变体，或 `undefined`。概念上：

  ```ts
  type IconRequest<Variant extends string = IconVariant> = Readonly<{
    size: 'xsm' | 'sm' | 'md' | 'lg';
    variant?: Variant;
  }>;

  type IconEntry<Variant extends string = IconVariant> =
    React.ReactNode | ((request: IconRequest<Variant>) => React.ReactNode);
  ```

  主题将尺寸和变体映射到其库自有的 Props 或图形：

  ```tsx
  defineTheme({
    name: 'brand',
    iconVariants: ['outline', 'filled'],
    icons: {
      info: ({size, variant}) => (
        <InfoIcon
          weight={
            variant === 'filled' ? 'fill' : size === 'sm' ? 'bold' : 'regular'
          }
        />
      ),
    },
  });
  ```

- **FR2 — 主题一次性列出其变体名称。** `DefineThemeInput.iconVariants` 是适用于整个主题的变体名称列表，绝不针对单个图标。设置 `iconVariants` 的子主题替换继承的列表；省略它的子主题继承该列表。
- **FR3 — 该列表为类型化主题和请求提供类型。** 在 `defineTheme` 内，条目函数接收 `variant`，静态插槽请求和插槽解析器返回它，类型为主题列出的名称。`Icon` 的 `variant` prop 使用 `IconVariant`，即公开、可增强的 `IconVariantMap` 中的名称。主题构建根据其构建的主题写入该映射的增强，方式与增强自定义组件值相同，应用也可自行增强该映射。Astryx 绝不从多个主题的列表推导变体类型；当应用加载多个已构建主题时，TypeScript 合并它们的增强，FR4 在活动主题未列出所请求名称处渲染普通图标。
- **FR4 — 未列出的变体渲染普通图标。** 仅当活动主题列出所请求的 `variant`，且对于主题提供的条目，定义该条目的主题也列出它时，`Icon` 才将所请求的 `variant` 传给选中的条目；子主题继承的条目保留定义它的主题的列表。否则条目接收 `undefined` 并渲染其普通图形：省略的变体、未列出的变体，以及无活动主题列表的请求均如此。进程级注册接收活动主题列出的变体。节点条目对每个变体渲染同一节点。
- **FR5 — 变体仅适用于注册表图标。** 直接传给 `Icon` 的图标组件按原样渲染，不接收 `variant`；`Icon` 不将 `variant` 转发给它或 DOM。`variant` 仅选择图形：`icon` 主题化目标不反映它，因此它没有样式选择器。
- **FR6 — 来源在请求之前解析。** 主题条目优先于进程级注册，后者优先于内置默认值；空值静态条目落到下一个来源。选中函数的结果对该次解析是最终的：空值结果不渲染任何字形，也绝不落到另一个来源。
- **FR7 — 条目和解析器是纯且同步的。** 它们仅根据输入计算结果，不使用 Hook、DOM 或 SVG 测量、浏览器全局对象、网络访问、可变模块状态或副作用。
- **FR8 — 构建工具可枚举主题的图标。** 主题条目可接收的每个请求是其键与四种尺寸之一，以及无变体或该条目在 FR4 下可接收的一个变体。生成或子集化图标图形的构建工具可对恰好该集合调用每个条目；结果与条目在运行时的返回一致。

### 尺寸

- **FR9 — 条目接收解析后的尺寸。** `Icon` 通过 `component:Icon` ORD1（`xsm`、`sm`、`md`、`lg`）解析其尺寸，并将其传给条目，由条目为该尺寸选择图形。
- **FR10 — 主题在样式表中设置每个尺寸的尺寸值。** Icon 主题化目标在其尺寸选择器上保证 `width` 和 `height`，因此 `components.icon['size:sm']` 设置 `sm` 尺寸值。无主题样式时，尺寸在 16px 根字号下为 12、16、20 和 24px，并随根字号缩放。

### 组件图标插槽

- **FR11 — 组件图标是 `componentIcons` 插槽。** 组件自有的可主题化图标是 `ComponentIconSlotMap` 中的插槽，通过 `componentIcons` 映射，如 `architecture:icon-resolution-and-component-slots` 所述。`true` 值声明无状态插槽；对象类型声明插槽的只读状态字段。
- **FR12 — 插槽值是名称、`null`、静态请求或状态解析器。** `componentIcons` 接受 `IconName`、`null`、静态部分请求，或接收插槽状态并返回部分请求的纯同步函数。概念上：

  ```ts
  type ComponentIconRequest = Readonly<{
    icon?: IconName | null;
    variant?: IconVariant;
    size?: 'xsm' | 'sm' | 'md' | 'lg';
  }>;

  type ComponentIconValue<State> =
    | IconName
    | null
    | ComponentIconRequest
    | ((state: Readonly<State>) => ComponentIconRequest | null | undefined);
  ```

  无状态插槽接受除函数外的所有形式。组件提供其状态；主题在普通解析器代码中处理状态组合：

  ```tsx
  componentIcons: {
    'example-nav-item-leading': ({selected}) => ({
      variant: selected ? 'filled' : 'outline',
    }),
  }
  ```

- **FR13 — 部分结果逐字段回退。** 调用方传入的图标内容优先于插槽。否则请求的 `icon` 替换组件的回退名称，`variant` 和 `size` 应用于实际渲染的名称。缺少 `icon` 保留组件原本会渲染的名称，缺少 `variant` 保留组件原本会请求的变体（或无），缺少 `size` 保留组件的尺寸。`null`、`null` 结果或 `icon: null` 隐藏图标；`undefined` 结果不改变任何内容。
- **FR14 — 插槽尺寸仅改变图标尺寸。** 解析后的 `size` 替换该插槽 Icon 的组件尺寸。控件的高度、触摸目标、位置、颜色、交互和可访问性仍由组件设置，包括家族契约跨状态固定的任何外部尺寸。
- **FR15 — 主题通过插槽设置状态相关图标。** 将状态相关图标暴露为 `componentIcons` 插槽的组件通过插槽传递该状态，以便主题解析它。根据自身状态选择共享图标名称而无插槽的组件保持原样，直到其自身契约添加插槽。调用方传入的按状态图标 Props（如 `pressedIcon` 或 `selectedIcon`）作为调用方提供的图标内容。组件通过指示器绘制的有状态控件标记（如选择勾选、单选按钮或复选框方框）仍为指示器：指示器接收其家族的状态，主题通过 `defineTheme({indicators})` 按名称替换它。同一视觉元素绝不同时是指示器和 `componentIcons` 插槽。
- **FR16 — 插槽状态是文档化的主题元数据。** 插槽的状态字段与其插槽元数据一起列在所属组件的文档中，以及 CLI 和文档站点发现中。更改已发布的状态字段是兼容性变更。

### 读取、已构建主题与服务器渲染

- **FR17 — 插槽解析是插槽函数的唯一调用方。** 组件（包括增强 `ComponentIconSlotMap` 的包中的组件）将其插槽状态传给由 `architecture:icon-resolution-and-component-slots` 拥有的共享插槽解析。该解析调用主题的函数并通过 `Icon` 渲染结果。任何公开主题读取都不调用或返回插槽函数。
- **FR18 — 公开读取返回节点和静态值。** 注册表、扩展、快照和 Hook 读取、`IconRegistry`、`DefinedTheme['icons']` 以及已构建主题的图标导出返回 React 节点；函数条目在 `{size: 'md'}` 下读取。`DefinedTheme['componentIcons']` 和已构建主题的插槽导出是静态视图：按编写原样的名称、`null` 和静态请求，解析器插槽省略。`DefinedTheme['iconVariants']` 是主题的列表。产品通过 `Icon` 请求尺寸或变体。
- **FR19 — 运行时、已构建和服务器主题一致。** 运行时主题、`extends`、构建产物、服务器渲染和水合解析为相同节点。子主题的条目替换同一键的继承条目。构建无法携带的函数使构建失败。服务器代码可用持有函数的主题解析图标；客户端代码将该主题传给 `Theme`。
- **FR20 — 格式错误的解析器输出视为缺失。** 请求中未知的 `icon`、主题未列出的变体，或四种名称之外的尺寸，逐字段忽略，并仅产生一条开发环境警告。

### 平台支持

- 支持下限：Astryx Core 支持的所有 React、服务器渲染和浏览器目标。
- 不支持的行为：异步解析器，以及将持有函数的主题作为 prop 从服务器组件传给客户端组件。
- 浏览器证据：主题切换、嵌套主题、组件状态、已列出和未列出的变体，以及每个 Icon 尺寸在主题尺寸值下渲染预期节点并水合到该节点。

## 当前状态影响

现有节点条目、直接图标组件、命名空间键、实例 Props 和读取继续有效。无 `iconVariants`、函数条目或尺寸样式的主题渲染相同的图标。以下所有者承担此变更：

- `component:Icon` — 注册表模式接受 `variant`（FR4、FR5），用 ORD1 解析的尺寸选择图形（FR9），其目标在尺寸选择器上保证 `width` 和 `height`（FR10）。
- `architecture:component-theming-surface` — INV14 将 `Icon.variant` 接纳为主题可扩展轴，其回退为无变体的请求（FR3、FR4）。
- `architecture:icon-resolution-and-component-slots` — 条目是节点或函数，选中函数的结果是最终的（FR1、FR6）；从未发布的 `selector-selected-option` 插槽离开 `ComponentIconSlotMap`，因为 Selector 通过 `check` 指示器绘制其选中选项标记（FR15）；插槽值为 `true` 或状态形状，INV3 接纳静态请求和状态解析器（FR11、FR12）；INV8 允许插槽请求设置插槽的图标尺寸和变体（FR13、FR14）；指示器保持独立（FR15）；插槽解析是插槽函数的唯一调用方（FR17）。
- `architecture:theme-authoring-contract` — `iconVariants`、函数条目和插槽函数被规范化并继承，公开读取保持节点值或静态（FR2、FR18、FR19）。
- `architecture:theme-compilation` — 构建产物重现函数条目、插槽解析器和变体列表，构建工具可枚举主题的请求（FR8、FR19）。
- `architecture:theme-application` — 客户端代码将持有函数的主题传给 `Theme`（FR19）。
- 消费者 `theme` 和 `Icon` 文档描述 `iconVariants`、函数条目、`variant`、尺寸样式和插槽解析器。

## 验证

| 契约      | 验证                                                                | 代表性状态                                                                                                                              | 变异或失败预期                                                                                     |
| --------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| FR1–FR7   | 注册表类型、解析器和 Icon 渲染测试                                  | 节点；函数；null；省略、已列出和未列出的变体；无活动列表；替换列表下的继承条目；主题、全局、默认；直接                                 | 来源混合、未列出的变体到达条目、`variant` 到达组件或 DOM                                          |
| FR2–FR3   | 类型测试和主题构建增强夹具                                          | 条目和解析器中列出的名称；子主题替换列表；构建增强；未知名称                                                                           | 条目看到未列出的名称类型、子主题合并列表，或 Astryx 从两个主题的列表推导一个类型                   |
| FR8       | 针对运行时解析的构建枚举测试                                        | 每个键 × 四种尺寸 × 无变体及条目可接收的每个变体；替换列表下的继承条目                                                                 | 枚举请求与运行时输出不同，或枚举了条目无法接收的变体                                               |
| FR9–FR10  | 主题样式和尺寸测试                                                  | 每个尺寸在有和无主题样式下                                                                                                              | 有样式的尺寸不改变字形尺寸值，或无样式的尺寸改变                                                   |
| FR11–FR16 | 插槽类型、状态、回退、指示器、几何和元数据测试                      | `true` 插槽；状态插槽；静态请求；部分和完整结果；`null`；调用方内容；文档化状态                                                       | 回退顺序改变、`null` 不再隐藏、插槽尺寸改变控件，或标记同时属于两种类型                             |
| FR17–FR19 | 公开读取、SSR、水合和运行时/构建/extends 测试                       | 单参数读取；静态视图；嵌套主题；子主题替换；构建函数条目；客户端主题                                                                   | 公开读取返回函数、构建和运行时结果不同                                                             |
| FR20      | 类型和返回字段测试                                                  | 无效状态形状；未知图标；未列出的变体；无效尺寸                                                                                         | 格式错误字段改变回退，或生产环境警告                                                               |

## 决策日志

### DEC-1 — 单一变体请求

**引用：** `spec:AST-087/DEC-1`
**决策者：** cixzhang，2026-10-22

变体系统在设计系统中是标准做法。`Icon` 携带一个可选 `variant`，更重或更轻的笔画是主题条目针对最终尺寸的调整。

已拒绝：外观和字重轴，因为它们因产品而异。

### DEC-2 — 主题条目是尺寸和变体的函数

**引用：** `spec:AST-087/DEC-2`
**决策者：** cixzhang，2026-10-22

主题条目是节点，或接收最终尺寸和可选变体并返回所提供图形的纯同步函数，因此主题拥有其图标库特有的所有内容。

已拒绝：按图标的能力声明和自适应变体树。

### DEC-3 — 样式表设置图标尺寸值

**引用：** `spec:AST-087/DEC-3`
**决策者：** cixzhang，2026-10-22

主题通过 Icon 样式表目标设置每个 Icon 尺寸的尺寸值。

已拒绝：主题添加的尺寸名称，因为使用它的组件在主题缺失时没有值。

### DEC-4 — `componentIcons` 持有组件图标

**引用：** `spec:AST-087/DEC-4`
**决策者：** cixzhang，2026-10-22

全局图标映射命名每个图标的用途，`componentIcons` 按组件插槽覆盖它。插槽可在普通解析器代码中从类型化组件状态解析其请求。

已拒绝：第二个组件图标映射、图标角色概念、按状态嵌套的映射和复合状态键。

### DEC-5 — 主题设置状态相关图标

**引用：** `spec:AST-087/DEC-5`
**决策者：** cixzhang，