---
schema_version: 4
template_version: 1
kind: system-spec
id: spec:AST-012
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-10-24
phase: accepted
owners: [cixzhang, rubyycheung, imdreamrunner]
affects_architecture:
  [
    architecture:component-theming-surface,
    architecture:public-component-api,
    architecture:theme-authoring-contract,
    architecture:theme-compilation,
    architecture:theme-application,
  ]
affects_families: []
affects_contributing: []
affects_consumer_docs: [theme, AppShell]
---

# 主题适配

<!-- review-applicability:v1 -->

```json
{
  "scope": "global",
  "triggers": {
    "theming": ["FR4", "FR5", "FR6", "FR7"]
  }
}
```

## 意图

主题应能针对环境条件表达可选的 token 与组件适配，无需第二个主题、手写媒体查询 CSS 或 React 状态。系统命名为 `adaptations` 而非 `responsive`，因为条件可能涵盖宽度、指针精度、对比度偏好、动效偏好以及未来经评审的环境维度。

```ts
defineTheme({
  adaptations: {
    widthBreakpoints: {
      sm: 640,
      md: 768,
      lg: 1024,
      xl: 1280,
      '2xl': 1536,
    },
    rules: [
      {
        when: {
          width: {from: 'lg', below: 'xl'},
          pointer: 'coarse',
          contrast: 'more',
        },
        value: {
          components: {/* overrides */},
        },
      },
    ],
  },
});
```

`widthBreakpoints` 是宽度层级起始的命名点。`width.from` 包含该点，`width.below` 不包含，因此 `{from: 'lg', below: 'xl'}` 恰好是 `lg` 区间。`when` 中的字段为 AND 关系。

仅定义断点映射不产生任何 CSS。只有带适配值的规则才会产生 CSS。

## 非目标

- 为内置主题添加适配规则或选择产品特定值。
- 为 FormLayout、布局原语或产品布局定义响应式策略。
- 接受原始媒体查询字符串、用户自定义条件字段或用户自定义断点名。新增环境维度仍须显式扩展 API。
- 改变颜色模式解析或 MediaTheme 的表面检测及其含义。
- 让响应式组件消费主题适配规则；组件可以解析共享宽度点，但其行为仍独立归属。
- 适配 canvas、数据可视化或其他 JavaScript 消费者使用的非 CSS token 读取；这些仍解析根主题值。

## 需求

- **FR1 — 宽度断点是固定的起始点。** 每个主题的有效 `widthBreakpoints` 映射为 `sm: 640`、`md: 768`、`lg: 1024`、`xl: 1280`、`2xl: 1536` CSS 像素，除非为这些固定名称覆盖值。值 MUST 为有限、正数且严格递增。根值是 `sm` 以下的隐式基础；不存在可配置的 `xs: 0` 点。仅定义或覆盖该映射 MUST NOT 产生 CSS。
- **FR2 — 条件是命名的、封闭的且可组合的。** `when` 对象可包含 `width`、`height`、`pointer`、`contrast` 和 `motion`；字段为 AND 关系。
  `width` 接受 `from` 和 `below`，使用五个固定名称，MUST 包含其一或两者。`from` 为包含，`below` 为排除，任一边界可省略但不可同时省略，有界宽度 MUST 满足 `from < below`。
  `height` 仅接受 `{below: 500}`：视口高度低于 500（排除），即 FR9 紧凑定义的高度边界。`pointer` 接受 `coarse | fine`；`contrast` 接受 `more | less | no-preference`；`motion` 接受 `reduce | no-preference`。
  未来环境字段须显式加入此封闭词汇表。
- **FR3 — 规则是命名对象。** `adaptations.rules` 包含 `{when, value}` 对象，而非位置元组。空 `when` 无效。若某值须在先前规则之后的所有宽度生效，由后续规则覆盖 `{width: {below: 'sm'}}` 和 `{width: {from: 'sm'}}` 表达。无变更值的规则可不产生 CSS。
- **FR4 — 规则使用主题值契约。** `value` 可包含 `typography`、`color`、`radius`、`motion`、`tokens`、`components` 和 `localTokens` 覆盖。`rules[].value.localTokens` 不是注册面：只有根 `localTokens` 或已注册的精确基础可声明名称。规则只能替换有效谱系中已注册的精确名称，并使用 AST-006 的 `TokenValue` 规范化与校验。identity、inheritance、syntax、icons、indicators、media-surface 字段、adaptations 和 `motion.reducedMotion`（`spec:AST-108` FR5）仍归根所有。在单条规则内，值使用根展开、深合并和 kind 优先级。生成轴针对根轴元数据展开，每个产出的叶子计为该规则的写入，但有效根 `components` 上已声明的精确叶子路径处的不变 type-scale 组件默认值除外。生成的 token 和规则编写的组件叶子仍为写入。缺失的必需 scale 字段 MUST 显式提供，而非从近似内置默认值合成。规则中的组件写入 MUST 使用与根 `components` 相同的 target、axis、value-domain 和 extension 校验；adaptations 不定义单独的组件值校验策略。即使有效根没有针对该 target 或 value 的样式规则，规则 MAY 为内置值或权威开放原语域接受的值设置样式。有效性依赖主题注册的值 MUST 已在有效根 `components` 面上注册；规则 MUST NOT 条件性引入它。共享根校验产生的未解析结果原样带入适配校验。它既不是永久适配豁免，也不是轴可扩展的证据。状态归组件所有，绝不由主题生成。
- **FR5 — 媒体表面更具体。** 适配值在根主题之上解析。当 `onDark` 或 `onLight` 覆盖与适配写入同一解析叶子时，媒体表面值胜出。适配不创建嵌套媒体表面语法。
- **FR6 — 规则顺序决定优先级。** 根值先于适配规则应用，媒体表面覆盖除外，后者按 FR5 在其后应用。规则按声明顺序级联，对每个解析 token 或组件叶子的最后匹配写入胜出。条件形状不产生独立特异性分数。重复的 `when` 对象有效。重排重叠规则是有意的行为变更。后续仅 scale 规则在精确路径存在于根时保留先前编写的组件叶子；根中不存在时，生成的组件叶子仍为写入，若两条规则均匹配则胜出。
- **FR7 — 主题扩展保留级联顺序。** 子主题继承基础宽度断点覆盖和有序规则。断点按名称合并。继承规则保持相对顺序并针对子主题的有效映射解析；子规则追加其后。无移除操作符：空子值为无操作，继承行为只能由后续子规则写入受影响叶子来中和。子主题的根值绝不覆盖匹配的继承规则，因为根值先于适配规则应用。
- **FR8 — AppShell 共享宽度点。** `mobileNav.breakpoint` 接受五个宽度断点名称加 `none`。对于选定名称，移动模式仅在 `width < effectiveWidthBreakpoints[name]` 时生效；相等属于较宽一侧。`none` 永不匹配。AppShell 读取最近的 Theme 上下文；无 provider 上下文时遵循 `architecture:theme-application` 中的根主题回退，无活动主题时使用 FR1 默认值。`defaultIsMobile` 仍为 SSR 提示；客户端查询在水合后使用有效映射。
- **FR9 — 紧凑在每个平台只有一个定义。** 屏幕视口宽度低于有效 `md` 点（FR1）或视口高度低于 500 时为紧凑。各平台数字相同：Web 为 CSS 像素，iOS 为点，Android 为密度无关像素；原生呈现不将紧凑映射到平台尺寸类。主题导出此唯一定义，所有按屏幕尺寸选择呈现的解析器均使用它，包括 Web 触摸呈现解析器、Selector 的 `adaptive` 呈现和 `@astryxdesign/native`；均不保留自己的尺寸数字。同时需要粗指针的解析器将其与紧凑做 AND。紧凑仅决定呈现选择，例如用 sheet 代替 popover；调用方选择的布局宽度（如 AppShell 的 `mobileNav.breakpoint`，FR8）不是紧凑。主题 `adaptations` 仍仅限 Web（`spec:AST-082` FR18）；原生从可移植主题读取紧凑数字。
- **IR1 — 原子化规范化与拒绝。** 一条路径在 `defineTheme` 和 `astryx theme build` 产生部分输出之前，校验宽度断点、命名条件字段、宽度范围、主题扩展、已注册 local-token 名称和具体展开值。`defineTheme` 在每个有效 `DefinedTheme` 上保留规范化生成轴元数据；静态构建按 IR4 保留相同元数据。
- **IR2 — 适配值以 CSS 优先。** 主题样式表应用后，适配值仅从 CSS 解析，无需 JavaScript 重渲染、resize 监听或水合修正。运行时 Theme 注入和静态构建使用相同的规范化规则、作用域和级联。根值先于规则输出；规则块保持声明顺序；媒体表面覆盖在两者均适用时于规则后输出。输出 MUST NOT 在会改变级联时合并、重排或丢弃规则块，包括重复 `when` 对象和恢复根值的后续规则。每个组件叶子降级到与其根等价物相同的声明目标，包括派生私有变量展开。`useTheme` token 读取和服务端安全 token 辅助函数仍为根值读取。
- **IR3 — 工具链可见仅规则值。** 校验、组件诊断、私有变量检查、字体提示和 `light-dark()` 颜色方案检测 MUST 检查仅在规则中声明的值。规则中的组件写入 MUST 经过与根组件写入相同的共享校验路径。适配仅增加无条件注册检查：仅通过主题注册接受的值 MUST 已存在于有效根组件面。内置有限值和权威开放原语域接受的值不要求匹配的根样式声明。规则校验 MUST 自动继承根校验的未来改进；工具链 MUST NOT 维护适配专用允许列表、未解析域例外或域推断路径。
- **IR4 — 构建主题保留扩展语义。** 构建主题的 JavaScript 模块保留有效宽度断点映射、规范化生成轴元数据、已注册 local-token 谱系和源等价 `extends` 所需的有序规范化规则；不嵌入生成的 CSS 文本。子主题从构建基础继承和解析适配，与从源基础完全一致。

### 平台支持

- 适配在 Astryx 浏览器支持底线内使用视口宽度、视口高度、主指针、对比度偏好和减少动效查询；无 JavaScript 样式回退。
- 真实 Chromium MUST 验证条件、嵌套、AppShell 一致性和首屏绘制。真实 Safari MUST 验证排除宽度边界和主指针分类；仅 Playwright WebKit 不构成 Safari 证据。

## 当前状态影响

当前 `main` 无主题适配 API。AppShell 仅导出 `sm | md | lg | none`，硬编码前三个点，并将相等视为移动。本提案新增 `xl` 和 `2xl`，通过活动主题解析所有名称，并使上边界为排除。AppShell 联合类型扩展为增量；相等变更属破坏性行为变更，实现时需对应发布说明。

本变更影响 `architecture:component-theming-surface`、`architecture:public-component-api`、`architecture:theme-authoring-contract`、`architecture:theme-compilation` 和 `architecture:theme-application`。在本草案被接受并实现之前，其当前契约仍为权威。

#5543 的公开层级 API 被 DEC-1 和 DEC-2 取代，MUST NOT 按原样落地。其值解析器、显式值保留、CSS 生成/构建一致性和扩展元数据仍为有用的实现证据。

主题未导出紧凑定义，`when` 无 `height` 字段。Selector、MultiSelector、ContextMenu 和 DropdownMenu 的 `adaptive` 呈现（MoreMenu 转发至 DropdownMenu）以及 DropdownMenu 的紧凑触摸子菜单决策使用 `(max-width: 768px) and (pointer: coarse)`，ContextMenu 和 MoreMenu 消费者文档将其描述为"768px 及以下"：粗指针屏幕恰好 768 像素宽且至少 500 高时计为紧凑，而宽度超过 768 像素但高度低于 500 的横屏手机则不计。按 FR9，前者打开 popover，后者打开底部 sheet；该变更的发布说明将 `presentation` 列为保留表面的方式。`PortableTheme` 不携带紧凑定义，`@astryxdesign/native` 尚无紧凑解析器；获得时采用 FR9。

## 验证

| 契约          | 验证                                                  | 代表状态                                                                                                                                                                                                                               | 失败信号                                                                                                                                                                                                                 |
| ----------------- | ------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| FR1–FR3           | 宽度断点和条件矩阵加浏览器边界 | 默认/覆盖；from/below/范围；pointer/contrast/motion；多字段 AND；空/无规则                                                                                                                                                      | 点作为上界、区间间隙/重叠、字段 OR 关系，或未使用映射产生 CSS。                                                                                                                         |
| FR4–FR7           | 共享解析器、表面、级联顺序和扩展测试  | 已注册/未注册 local token；不完整 scale；有限内置值；根中缺失的开放原语值；根注册和仅规则自定义值；未解析共享结果；后续生成 vs 先前显式叶子；表面冲突 | 规则注册 local 或组件值、有效开放值需要虚拟根规则、适配校验与根校验分歧、未解析结果成为永久豁免，或继承顺序丢失。 |
| FR8               | 主题/AppShell 集成                                    | 所有名称/`none`；SSR 提示；无/最近/根/嵌套主题；精确边界                                                                                                                                                                          | AppShell 使用错误映射、相等或作用域。                                                                                                                                                                               |
| FR2（height）、FR9 | 条件矩阵加 Web 和原生解析器测试        | 高度 800 时宽度 767/768；高度 499/500 时宽度 1024；覆盖 `md`；Selector `adaptive`；Web 触摸解析器；原生解析器                                                                                                          | 解析器有自己的尺寸数字或包含边界、原生解析器使用平台尺寸类或不同数字，或接受 `below: 500` 以外的 `height` 值                                            |
| IR1–IR3           | 运行时/构建和 CLI 正/负夹具              | 无效映射/范围/字段；源轴元数据；重复 `when`；恢复根规则；有限/开放/未解析组件域；声明目标；根注册和仅规则自定义值；非 CSS token 读取                                | 无效输入写入输出、根和适配组件校验不一致、独立有效值被强制入根、适配条件性注册自定义值，或工具链遗漏仅规则值。     |
| IR4               | 生成模块导入和大小测试                        | 无规则/有规则；源/构建父与子；生成/local-token 元数据                                                                                                                                                                      | 断点、规则、顺序或元数据消失；扩展分歧；CSS 文本进入模块；或解析层重复。                                                                                                |

## 决策日志

以下决策由 `cixzhang` 作为当前已接受规范的一部分批准。

### DEC-1 — 使用 adaptations 与宽度专用断点

**参考：** `spec:AST-012/DEC-1`
**决策者：** `cixzhang`，`2026-09-02`

使用 `adaptations` 而非 `responsive` 或 `responsiveness`，因为系统涵盖多个环境维度。宽度点位于 `widthBreakpoints` 下，而非通用 `breakpoints` 字段，为未来独立的 `heightBreakpoints` 命名空间留出空间。Astryx 拥有 `sm`、`md`、`lg`、`xl` 和 `2xl`，默认 640、768、1024、1280 和 1536。宽度点标记层级起始，无规则时不产生 CSS。

已拒绝：仅响应式命名、通用断点存储、自定义宽度名称和上界命名层级。

### DEC-2 — 使用带嵌套宽度条件的命名规则

**参考：** `spec:AST-012/DEC-2`
**决策者：** `cixzhang`，`2026-09-02`

规则使用 `{when, value}` 对象。`width: {from, below}` 表达包含下界和排除上界。`when` 中的环境字段为同级，AND 关系。

已拒绝：位置元组、扁平 `minWidth` / `maxWidth`、嵌套字面 `@media` 键和任意原始查询。

### DEC-3 — 使用声明规则顺序作为优先级

**参考：** `spec:AST-012/DEC-3`
**决策者：** `cixzhang`，`2026-09-02`

规则遵循级联顺序。根值先应用，继承规则保持顺序，子规则追加。在任何匹配环境中，最后写入解析叶子的规则胜出。重复条件有效，重排重叠规则是有意的行为变更。

已拒绝：从条件形状推断优先级或对不可比条件拒绝冲突。

### DEC-4 — 通过构建扩展保留适配

**参考：** `spec:AST-012/DEC-4`
**决策者：** `cixzhang`，`2026-09-02`

构建和源主题为 `extends` 保留等价宽度断点和规范化规则数据，生成的 CSS 仍为独立产物。

已拒绝：从构建基础编译掉编写意图并静默收窄现有主题