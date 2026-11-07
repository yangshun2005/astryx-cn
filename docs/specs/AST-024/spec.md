---
schema_version: 4
template_version: 1
kind: system-spec
id: spec:AST-024
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-09-04
phase: accepted
owners: [rubyycheung]
affects_architecture: [architecture:theme-authoring-contract]
affects_families: []
affects_contributing: []
affects_consumer_docs: [theme, typography]
---

# 主题可扩展的 Heading 视觉角色系统规范

## 意图

产品主题有时需要 Astryx 三个内置展示角色之外的具名标题样式。主题可能需要主视觉样式、紧凑的章节标题，或其他本地命名的视觉角色，同时仍渲染正确的 `h1`–`h6` 元素。作者也可能需要修改某一处 Heading 的字重，而不创建新的视觉角色。

Astryx 已支持主题定义的 `Text` 类型，并在 `astryx theme build` 期间生成其 TypeScript 增强。`Heading` 没有提供同样的扩展点：其 `type` 是封闭联合类型，也没有 `weight` prop。本规范使 Heading 视觉角色可被主题扩展，保持文档语义与外观独立，并为生成的主题 CSS 和生成的类型提供一份可靠的契约。

## 非目标

- 向 Astryx 在每个主题中提供的选项中添加产品特定的标题名称。
- 从视觉类型名称或字号推断 `h1`–`h6` 级别。
- 允许视觉样式修复不正确的文档大纲。
- 在没有主题声明或本地源的情况下添加任意 Heading 字号。
- 替换 `Text`、排版令牌或现有的字号比例生成器。
- 定义额外排版值的计算方式；主题本地比例扩展的编写仍属于单独的提案。
- 在本次变更中扩展每个主题可用的字重选项。

## 术语

- **语义级别：** `Heading.level` 的值，用于选择 `h1`–`h6` 以及元素在文档大纲中的位置。
- **视觉类型：** 一种具名样式，控制字号、行高、字体族、字重或其他已批准的视觉属性，但不改变语义级别。
- **内置类型：** `display-1`、`display-2` 或 `display-3`。
- **自定义类型：** 通过 Heading 组件覆盖声明、并通过可增强映射暴露给 TypeScript 的主题自有视觉类型。
- **字重覆盖：** 为单个渲染的标题显式指定 `Heading.weight` 值；它优先于视觉类型或语义级别的默认值，且不改变二者。

## 当前限制

`HeadingType` 目前是封闭字面量联合类型 `'display-1' | 'display-2' | 'display-3'`。TypeScript 模块增强无法拓宽类型别名。因此主题构建器在生成 `.variants.d.ts` 时会跳过 `heading['type:hero']` 这类主题规则；生成一个无关的新接口不会改变 `HeadingProps.type`。

浏览器可以匹配生成的 `data-type="hero"` 选择器，但应用源码传入 `type="hero"` 会报类型错误。即使绕过该错误，Heading 也没有为该名称定义基线 StyleX 条目。因此当前系统允许 CSS 被其公共组件 API 无法安全使用。

## 需求

### 可扩展的视觉类型

- **FR1 — Heading 暴露真正的增强点。** 公共 Heading 子路径必须导出一个接口，其键构成 `HeadingType` 的自定义部分。内置类型无需增强即可使用。生成的声明必须拓宽 `HeadingProps.type` 所使用的同一接口；针对仅桶文件或未使用接口的声明无效。
- **FR2 — 主题构建登记已声明的自定义类型。** 当主题包含自定义 `heading['type:<name>']` 覆盖时，`astryx theme build` 必须生成相应的类型增强，并在生成的主题声明中引用它。如果增强无法生成或加载，构建必须以可操作的指引失败，而不是输出类型化源码无法使用的 CSS。
- **FR3 — 自定义类型归主题所有，但 TypeScript 可用性是程序范围的。** 消费者导入所属主题的生成声明后，自定义名称即成为类型安全的。TypeScript 模块增强作用于整个编译过程，因此无法证明某个调用点有匹配的 Theme 处于活动状态。文档必须说明这一区别。对于未导入该声明的应用，自定义名称不得扩大 Heading 的内置类型集。
- **FR4 — 运行时样式是确定性的。** 有效的自定义类型必须通过与内置类型相同的主题选择器路径反映其名称。在主题 CSS 生效前，它必须保留所选语义级别的安全基线，而不是变为无样式或继承无关的环境文本字号。如果应用源码在未定义该名称的主题下使用已导入的自定义名称，则应用安全基线。开发工具在能够确定活动主题与不匹配时应当发出警告，且不增加生产运行时开销。
- **FR5 — 部分自定义类型有效。** 自定义 Heading 类型必须包含至少一个有意义的视觉声明。字号、行高、字体族、字重、字距、颜色及其他支持的属性可以独立修改；省略的属性继承所选语义级别的基线。空或完全无效的自定义类型在输出前失败。
- **FR6 — 现有类型不变。** 未声明自定义 Heading 类型的主题必须保留其当前类型、运行时输出和生成的构建产物。现有 `display-1` 到 `display-3` 的行为保持兼容。

### 单次使用的字重

- **FR7 — Heading 接受可选的字重覆盖。** `Heading` 必须接受与 `Text` 相同的具名 `TextWeight` 值：`normal`、`medium`、`semibold` 和 `bold`。省略时，保留当前由级别或类型派生的默认值。
- **FR8 — 显式字重具有明确优先级。** 提供的 `weight` prop 必须优先于内置或自定义视觉类型的默认字重。它不得改变 `level`、`accessibilityLevel` 或所选 HTML 元素。Heading 必须通过其主题目标反映所选字重，且输出的层叠必须保证此顺序，与编写规则顺序无关：显式字重，然后视觉类型默认值，然后语义级别默认值。
- **FR9 — 主题值保持权威。** 具名字重 prop 通过现有字重令牌解析，因此主题可以改变 `bold` 或 `semibold` 的含义，而无需修改组件源码。第一版不向组件 prop 添加原始数值或任意字重字符串。

### 语义与作者指引

- **FR10 — 级别与类型保持独立。** 每个自定义类型示例必须包含显式语义 `level`。工具和文档不得从 `hero`、`title` 或 `display` 等名称推断级别。
- **FR11 — 视觉复用不能成为跳过级别的理由。** 指引必须告诉作者从文档大纲选择标题级别，然后选择视觉类型。不得建议仅为获得不同字号而修改 `level`。
- **FR12 — 非标题文本继续使用 Text。** 如果内容不是章节或页面标题，编写指引必须使用 `Text`，即使期望相同的视觉样式。共享的视觉名称可以为两个组件分别声明，但不会使非标题内容具有语义。

### 验证与生成的构建产物

- **FR13 — 源码、运行时和构建一致。** 自定义 Heading 类型必须通过 TypeScript 检查，渲染所请求的 `h1`–`h6`，暴露其 `data-type`，接收生成的声明，并在运行时注入和静态构建两种主题模式下计算出预期的 CSS。
- **FR14 — 缺失声明可见地失败。** 使用自定义类型但未导入所属主题声明的消费者必须收到 TypeScript 错误。文档必须展示所需的主题构建产物导入，并解释导入它无法证明运行时哪个 Theme 处于活动状态。
- **FR15 — 无障碍行为不变。** 添加视觉类型或字重不得改变可访问名称、标题角色、语义级别、截断行为或 `aria-level` 处理。现有 Heading 无障碍测试仍然适用。

## 已接受的 API

```ts
// Excerpt from @astryxdesign/core/Heading
export interface HeadingTypeMap {
  'display-1': true;
  'display-2': true;
  'display-3': true;
}

export type HeadingType = keyof HeadingTypeMap;

export interface HeadingProps {
  level: 1 | 2 | 3 | 4 | 5 | 6;
  type?: HeadingType;
  weight?: TextWeight;
}
```

主题随后可以声明并构建本地角色：

```ts
defineTheme({
  name: 'product',
  localTokens: {
    '--astryx-theme-product-text-hero-size': '4.5rem',
    '--astryx-theme-product-text-hero-leading': '1.05',
  },
  components: {
    heading: {
      'type:hero': {
        fontSize: 'var(--astryx-theme-product-text-hero-size)',
        lineHeight: 'var(--astryx-theme-product-text-hero-leading)',
      },
    },
  },
});
```

导入构建好的主题构建产物后，应用源码可以使用该视觉角色而不改变其语义级别：

```tsx
<Heading level={1} type="hero">
  Product overview
</Heading>

<Heading level={2} type="hero" weight="bold">
  Featured report
</Heading>
```

## 当前状态影响

已接受的规范本身不改变任何运行时或公共 API。其实现将用公共的映射支撑联合类型替换 Heading 的封闭类型别名，教会主题构建器增强该映射，为自定义类型提供语义级别基线，并将现有的具名字重词汇添加到 `HeadingProps`。

仅使用三个内置展示类型的主题和应用无需迁移。已经输出 `heading['type:<custom>']` CSS 的主题目前有一个不可用的类型化选择器；在实现并重新构建后，其生成的声明将使该现有本地名称可供导入主题构建产物的消费者使用。增强在该 TypeScript 程序中全局可用，而自定义样式仍归活动主题所有。没有任何自定义名称成为每个 Astryx 应用都必须支持的选项。

## 验证

| 契约      | 验证                                                         | 代表性失败                                                                                     |
| --------- | ------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| FR1–FR3   | 公共子路径增强和生成声明类型测试                             | CSS 构建成功但 `type="hero"` 仍失败，或文档声称 TypeScript 知道哪个 Theme 处于活动状态。      |
| FR4–FR6   | Heading 单元测试和前后 CSS 快照                              | 自定义类型丢失级别基线，有意的部分角色失败，或内置主题改变输出。                               |
| FR7–FR9   | 针对级别、内置类型和自定义类型的字重目标与层叠测试           | `weight="bold"` 输给类型默认值，依赖编写规则顺序，或绕过主题字重令牌。                         |
| FR10–FR12 | 文档和编写夹具                                               | 工具从 `hero` 推断 `h1`，为字号跳过级别，或将 Heading 用于非标题文案。                         |
| FR13–FR15 | 消费者类型检查、运行时/静态构建一致性、DOM 和无障碍测试      | 构建产物未加载，选择器因构建路径而异，或视觉 prop 改变标题语义。                               |

### 完成标准

本规范仅在以下条件满足时从 `accepted` 移至 `shipped`：

- Heading 暴露并消费一个公共可增强类型映射；
- 主题构建生成并加载自定义 Heading 类型声明；
- 自定义类型具有安全的语义级别基线和至少一个有效的视觉声明；
- Heading 接受现有具名字重值，并具有文档化的优先级；
- 显式字重通过运行时和构建主题层叠胜出，与编写规则顺序无关；
- 运行时和静态主题构建产生等效的自定义类型样式；
- 内置 Heading 行为及无自定义类型的主题保持不变；以及
- 示例和测试保持语义级别与视觉样式独立。

## 待解决问题

无。

## 决策日志

### DEC-1 — 通过公共映射使 Heading 类型可扩展

**参考：** `spec:AST-024/DEC-1`
**决策者：** `rubyycheung`，`2026-09-04`

使用与其他可扩展组件 prop 相同的、经过验证的基于映射的模块增强模型。封闭类型别名无法拓宽，生成无关接口会错误地暗示类型安全而不改变 `HeadingProps`。

### DEC-2 — 保持语义级别与视觉类型分离

**参考：** `spec:AST-024/DEC-2`
**决策者：** `rubyycheung`，`2026-09-04`

自定义名称仅控制外观。作者继续从文档结构选择 `level`。这允许同一视觉角色用于不同的有效级别，而不将排版与无障碍语义耦合。

### DEC-3 — 添加与 Text 对等的字重

**参考：** `spec:AST-024/DEC-3`
**决策者：** `rubyycheung`，`2026-09-04`

将现有具名 `TextWeight` 词汇添加到 Heading，并使显式 prop 覆盖类型或级别默认值。首次变更中不添加第二套字重词汇或原始数值。

### DEC-4 — 按组件独立登记视觉角色

**参考：** `spec:AST-024/DEC-4`
**决策者：** `rubyycheung`，`2026-09-04`

视觉角色名称不会自动对 `Text` 和 `Heading` 同时可用。作者在其打算使用的每个组件上分别登记。底层主题本地排版值可以共享，编写工具可以提供同时创建两个映射，但必须展示并确认每次使用。

这防止 Text 样式悄然变成标题样式，同时仍避免重复的值定义。

### DEC-5 — 保持组件字重选择在各主题间一致

**参考：** `spec:AST-024/DEC-5`
**决策者：** `rubyycheung`，`2026-09-04`

`Heading.weight` 和 `Text.weight` 在每个主题中继续使用相同的具名选项。活动主题决定 `bold` 等名称解析为 700、750 还是其他支持的值。自定义视觉类型也可以在主题源中声明精确字重。

拒绝：在每个组件调用点直接接受任意数值，这会将应用源码耦合到一种字体并绕过主题所有权。

### DEC-6 — 明确说明主题生成 TypeScript 的局限

**参考：** `spec:AST-024/DEC-6`
**决策者：** `rubyycheung`，`2026-09-04`

导入主题的生成声明会使该主题的自定义 Heading 名称在整个 TypeScript 程序中可用。它不能证明每个使用点周围都有匹配的 Theme 处于活动状态。运行时样式仍以主题为作用域，不匹配时保留语义级别基线，开发工具在能够可靠识别时发出警告。

拒绝：承诺 TypeScript 模块增强无法强制执行的、感知 Provider 的静态类型。

### DEC-7 — 允许部分视觉角色

**参考：** `spec:AST-024/DEC-7`
**决策者：** `rubyycheung`，`2026-09-04`

自定义 Heading 类型可以有意只修改一个支持的视觉属性，其余从语义级别继承。它必须包含至少一个有效声明，但不必重述字号和行高。

拒绝：要求每个自定义类型都是完整的排版替换，这会使小的主题变化不必要地重复。