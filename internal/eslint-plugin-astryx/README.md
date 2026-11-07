# @astryx/eslint-plugin

用于 Astryx 设计系统 token 强制执行的 ESLint 插件。

## 理念：双层 Lint

本插件实现双层 lint 策略：

| 模式            | 受众      | 行为            | 触发条件                            |
| --------------- | --------- | --------------- | ----------------------------------- |
| **推荐** | 人类      | 仅警告          | 默认（本地开发）                    |
| **严格**        | Agent/CI  | 错误（构建失败）| `CI=true` 或 `ASTRYX_STRICT_LINT=1` |

### 为什么分两层？

- **Agent** 应完美遵守严格规则；它们没有违反规则的理由
- **人类** 在开发中需要灵活性；警告只提示，不阻塞

## 规则

### `@astryx/no-raw-intl-locale`

`InternationalizationProvider` 是唯一的面向用户的 locale 来源。本规则在 lint 范围内禁止三类独立行为：

1. **直接访问 `Intl`** —— 构造对 locale 敏感的 `Intl` 格式化器，**直接**调用 `toLocaleString`/`toLocaleDateString`/`toLocaleTimeString`/`toLocaleUpperCase`/`toLocaleLowerCase`/`localeCompare`，或在批准文件之外引用全局 `Intl` 对象——无论调用是否带 locale 参数，也无论该参数是字面量、变量还是 `navigator.language`。显式的 locale 表达式本身**不能**满足本规则；只有通过批准的 locale 感知边界才行。除直接调用形式外，规则还会标记别名（`const DTF = Intl.DateTimeFormat`）、解构（`const {DateTimeFormat} = Intl`）以及用计算键索引（`Intl[key]`）——这些写法都会构造格式化器，却不会以直接调用检查所监视的调用形式出现，因此被报告的是引用本身。
2. **将 `navigator.language`/`navigator.languages` 作为 locale 来源** —— 在**任何**位置，不仅限于作为 `Intl`/locale 方法调用的参数。`recognition.lang = lang ?? navigator.language` 即使不涉及 `Intl` API 也会被标记。
3. **不带 locale 的日期辅助函数调用** —— 从 Astryx 辅助模块导入的 `plainDateFormat`、`formatSharedDate`、`parseDateInput` 和 `isLocaleDayFirst` 调用必须传入 locale 参数。规则会跟踪命名导入、别名和命名空间导入，但不会匹配恰好同名的无关本地函数。

已发布的组件代码应通过公开的 provider 感知工具读取 locale——`useLocale()`/`useCollator()`（从 `@astryxdesign/core/i18n` 导出）——或使用现有的格式化辅助函数，如 `plainDateFormat`/`formatInstant`/`formatFilterValue`。日期辅助函数的调用方显式传入 `useLocale()`，使值始终可追溯到 provider。

**原始 `Intl` 由规则源码中的两个封闭文件列表控制：**

- `APPROVED_IMPLEMENTATION_FILES` 包含纯格式化器/解析器实现。其中的直接 Intl 调用仍要求语法上显式的 locale；`Intl.NumberFormat(undefined)`、省略 locale、以及不带 locale 参数的 locale 方法都是错误。对 Intl 的别名或解构也会被拒绝。
- `APPROVED_TEST_ORACLE_FILES` 包含刻意构造独立 Intl 期望值的具名测试，以免断言循环。
- `DATE_HELPER_DEFAULT_TEST_FILES` 仅包含验证向后兼容默认 locale 的两个辅助函数测试套件。此豁免仅适用于辅助函数调用的 locale 要求，不适用于原始 Intl 或 navigator 访问。

批准的实现：

- `packages/core/src/utils/plainDate.ts`、`.../utils/dateParser.ts` —— 日期格式化/解析核心
- `packages/core/src/Timestamp/formatInstant.ts`、`.../Timestamp/formatRelativeTime.ts`、`.../Timestamp/tooltipEntries.ts` —— Timestamp 绝对/相对格式化及其非显示时区有效性探测
- `packages/core/src/PowerSearch/formatFilterValue.ts`
- `packages/core/src/NumberInput/numberParser.ts` —— 在字段 locale 下读取键入和粘贴的数字
- `packages/core/src/utils/calendarDateParser.ts` —— 任意日历下 DateInput 和 DateTimeInput 的键入日期：从 Intl 读取并解析回来的名称、纪元、字段顺序和数字，提交的文本，以及日历名称
- `packages/core/src/utils/timeFormat.ts` —— TimeInput 和 DateTimeInput 的时刻文本：locale 的小时周期、数字、分隔符和日时段，格式化并解析回来
- `packages/core/src/i18n/useCollator.ts`
- `packages/charts/src/formatters.ts`

具名测试 oracle：

- `packages/charts/src/formatters.test.ts`
- `packages/core/src/Calendar/Calendar.test.tsx`
- `packages/core/src/NumberInput/NumberInput.test.tsx`
- `packages/core/src/NumberInput/numberParser.test.ts`
- `packages/core/src/NumberInput/numberParser.docblock.test.ts`
- `packages/core/src/Table/plugins/tree/useTableTreeState.test.tsx`
- `packages/core/src/Timestamp/tooltipEntries.test.ts`
- `packages/core/src/PowerSearch/formatFilterValue.test.ts`
- `packages/core/src/Timestamp/Timestamp.test.tsx`

具名辅助函数默认测试文件：

- `packages/core/src/utils/plainDate.test.ts`
- `packages/core/src/utils/dateParser.test.ts`

这些列表是**唯一**的例外机制。没有规则选项或 `eslint.config.js` 覆盖可以扩大它们。新增条目需要修改规则源码并添加针对性的规则测试。`navigator.language`/`navigator.languages` 无条件被拒绝，无论实现还是测试 oracle。

`Intl.Locale` 在任何形式下都不会被标记——裸引用、别名、调用——它检查标签而非格式化显示输出。`Intl.Segmenter` 在所有地方（无论是否 infra）对恰好两种形式豁免：带字素分割的直接调用（包括其标准定义的默认粒度——字素边界不会因 locale 产生有意义的差异，这也是 `packages/core/src/utils/characters.ts` 和 `packages/core/src/hooks/useStreamingText.ts` 不带参数调用它的原因），以及这两个文件与之搭配的 `typeof Intl.Segmenter === 'function'` 特性检测惯用法（不构造任何东西）。词、句及其他未知分割选项确实对 locale 敏感，遵循与其他所有格式化器相同的策略——并且与其他所有格式化器一样，在批准文件之外，对 `Intl.Segmenter` 进行**别名**而非直接调用**不**豁免：豁免的是调用形式，不是引用，因此 `const Seg = Intl.Segmenter; new Seg(undefined, {granularity: 'grapheme'})` 即使最终调用仅限字素仍会被标记。

**已知限制（仅语法，设计如此）：**

- 规则不会将别名追溯回其来源：`const lang = navigator.language; new Intl.DateTimeFormat(lang)` 会标记 `navigator.language` 读取，但不会将后续 `Intl` 调用标记为 navigator 来源（它仍会得到通用的原始 Intl 消息，因为在批准文件之外）。
- 对 locale 敏感的原型方法仅按**方法名**匹配，不识别接收者类型。恰好定义了同名方法用于无关的非 locale 敏感行为的自定义类仍会被标记——这是仅语法规则无法排除的误报。当前代码库中未出现此情况。
- **保存在变量中的计算方法名**——`date[computedMethodName]()`——在没有值流分析的情况下无法解析为 `'toLocaleString'`（或任何其他名称），因此**不会**被捕获。`Intl[key]` 仅对 `Intl` 对象本身（单个可命名的全局引用）填补了等价缺口；它无法推广到任意接收者上的任意方法名。

本地遮蔽的 `Intl` 或 `navigator`（参数、导入、本地工厂）不是平台全局对象，不会被当作全局对象标记——被遮蔽的 `Intl` 完全跳过所有 Intl 特定检查（包括其自身的别名），被遮蔽的 `navigator` 不被视为浏览器全局对象（因此通过它的 `navigator.language` 不会被标记为 `navigatorLocale`——基于遮蔽 Intl 但真实 navigator 组合的调用仍会因真实 navigator 读取而得到 `navigatorLocale`，与 Intl 遮蔽无关）。没有自动修复：选择正确的 locale 来源是 API 决策。

仓库对 `core`、`charts`、`richtext` 和 `vega` 中已发布的源码和测试将该规则启用为错误。它有意不覆盖 `lab`、应用、脚本或 stories。Lab 在组件毕业为已发布包时必须满足此要求。

**错误：**

```ts
new Intl.DateTimeFormat().format(date);
new Intl.DateTimeFormat(locale).format(date); // 仅显式 locale 不够
new Intl.DateTimeFormat(navigator.language).format(date);
value.toLocaleString(locale);
left.localeCompare(right, locale);
const DTF = Intl.DateTimeFormat; // 别名也会绕过直接调用检查
const {DateTimeFormat} = Intl; // ……解构也一样
```

**正确：**

```tsx
// 组件代码：通过公开 Hook 读取 provider locale。
import {useLocale, useCollator} from '@astryxdesign/core/i18n';

function Example() {
  const locale = useLocale();
  return <span>{plainDateFormat(date, DATE_FORMAT_LONG, locale)}</span>;
}

// 字符串比较：使用 useCollator()，而非原始 Intl.Collator/localeCompare。
function useSortedNames(names: string[]) {
  const collator = useCollator({numeric: true});
  return useMemo(
    () => [...names].sort((a, b) => collator.compare(a, b)),
    [names, collator],
  );
}
```

### `@astryx/no-internal-barrel-import`

报告包自身已发布源码中的相对导入，该导入解析到文件夹的 index 文件（barrel）而非定义所导入内容的模块：`import {List} from '../List'` 而非 `import {List} from '../List/List'`。Barrel 会将每个同级导出拉入导入者的模块图，因此通过 barrel 的懒加载模块会拖入无关代码，在单文件打包中还会拖入导入循环。它检查 `import`、`export … from` 和字面量 `import()`；仅类型导入和导出会被编译器擦除，允许。公共消费者继续从包入口（`@astryxdesign/core`）导入，本规则从不读取。

仓库配置对 `packages/*/src` 启用它，排除测试、stories 和测试辅助文件，采用严格模式严重级别（CI 中为错误）。规则引入时已存在的 barrel 导入按文件列在 `no-internal-barrel-import.baseline.json` 中，在 codemod 移除前保持允许；修复导入时移除条目，永远不要添加。`node internal/eslint-plugin-astryx/no-internal-barrel-import.baseline.mjs` 从源码重写列表（`--check` 验证），例如在将 `main` 合并到本规则自己的分支后或 codemod 后。

### `@astryx/no-raw-private-attribute`

报告在其所有者模块之外手写的私有 `data-astryx-*` 属性（`architecture:public-component-api` INV23）。每个私有属性在包中有一个所有者模块写入它，`packages/<package>/src/privateAttributes/<name>.stylex.ts`，以 camelCase 命名（`focusOwner.stylex.ts` 拥有 `data-astryx-focus-owner`），它一次性写入名称和选择器，并导出即用样式、setter props 及其包所需的运行时读取器；其他所有文件展开这些导出。

在 Core 中，所有者也可以是纯 TS 模块，`packages/core/src/privateAttributes/<name>.ts`，未编译的主题引擎可以导入它。它可以导出其名称及基于它构建的选择器文本，且只有其 `.stylex.ts` 同级文件和 `packages/core/src/theme/` 中的主题引擎可以导入它；`.stylex.ts` 所有者从不导出它们。当纯 TS 所有者存在时，其 `.stylex.ts` 同级文件不是所有者：它仅以字面量形式写入属性选择器，这些字面量到达其自己的 `stylex.create` 调用（直接或通过仅在那里读取的模块常量），因为 StyleX 仅在写入处解析选择器。它从不设置属性或导出选择器，无论通过字面量还是从其所有者导入的名称，也从不重新导出所有者。

```ts
// 在 privateAttributes/press.ts（纯 TS 所有者）中没问题
export const PRESS_ATTRIBUTE = 'data-astryx-press';
// 在其同级 privateAttributes/press.stylex.ts 中没问题
const PRESSED_ON = '[data-astryx-press="on"]';
const styles = stylex.create({
  base: {opacity: {default: 1, [PRESSED_ON]: 0.8}},
});
import {PRESS_ATTRIBUTE} from './press';
const isPressed = element => element.hasAttribute(PRESS_ATTRIBUTE);
// 一旦 press.ts 存在，在 privateAttributes/press.stylex.ts 中被标记
const pressOn = {'data-astryx-press': 'on'};
el.setAttribute('data-astryx-press', 'on');
export const pressOn = {[PRESS_ATTRIBUTE]: 'on'};
export {PRESS_ATTRIBUTE} from './press';
// 在除主题引擎外的任何其他 Core 文件中被标记，包括相邻文件
import {PRESS_ATTRIBUTE} from '../privateAttributes/press';
```

```ts
// 在所有者模块之外被标记
el.setAttribute('data-astryx-popover-fallback-close', '');
const styles = stylex.create({
  base: {transform: {default: null, '[data-astryx-press="on"]': 'scale(0.98)'}},
});
<div data-astryx-submenu-host="" />;
style.setAttribute(dataAttr('theme-prose'), name);

// 没问题：INV23 列出的公共属性
<div data-astryx-theme="dark" />;
const scope = `[data-astryx-media="${surface}"]`;
el.setAttribute(dataAttr('theme'), name);
```

它检查任意位置的字符串字面量（对象键、调用参数、`stylex.create` 和 `stylex.when.*` 内的选择器）、模板字面量（包括 `data-${dataAttrNamespace}-…`）、JSX 属性名以及 `dataAttr()` 调用。无法读取的名称（`data-astryx-${part}`、`dataAttr(name)`）报告为 `data-astryx-*`。在所有者模块内只能写入其自己的属性，且 `.stylex.ts` 所有者不得将原始名称或选择器作为字符串导出：不得作为导出或默认导出的值，不得在导出的对象、数组或函数返回值内，也不得通过其中之一的模块常量（包括在模板中或用 `+` 从该常量构建的选择器）。名称在那里仍可作为对象键（setter props 和 `stylex.create` 选择器）和调用参数（写入器，或 `closest`/`matches` 读取器）使用。导入另一个包的所有者模块会被报告。INV23 列出的公共属性从不报告：`theme`、`media`、`aspect-ratio-override`、`token`、`token-value`、`menu-press`、`pressable` 和 `edge-comp`，每个都在 `data-astryx-` 之后。构建每个名称的 `naming.ts` 不检查。

仓库配置对 `packages/*/src` 启用它，排除测试、stories 和测试辅助文件，采用严格模式严重级别（CI 中为错误）。仓库配置不 lint CLI 之外的 `.mjs`，因此 `.doc.mjs` 文本留给公共 API 审查。规则引入时已存在的站点按文件列在 `no-raw-private-attribute.baseline.json` 中，每个站点一次，在移入所有者模块前保持允许；移动站点时移除条目，永远不要添加。`node internal/eslint-plugin-astryx/no-raw-private-attribute.baseline.mjs` 从源码重写列表（`--check` 验证）。

### `@astryx/no-comment-provenance`

报告源码注释中的出处信息。注释只解释代码无法展示的内容：原因、约束、不变量或平台怪癖。记录链接代码，因此 `pnpm specs:for <path>` 能找到它；注释中的引用会过时并重复该链接。`docs/architecture/knowledge-contracts.md` 中的"Source comments"规则是权威。

```ts
// 被标记
// Matches spec:AST-108 FR4 and component:Button (#9428).
// Decided 2026-10-31; true as of Oct 2026.
// The panel MUST close when focus leaves it.

// 没问题
// Safari fires blur before click, so the close waits a frame.
```

它仅在注释中（从不在字符串中）查找四类文本：

- 记录 id（`spec:AST-108`、`component:Button`、`module:Icon/registry`、`architecture:…`、`family:…`、`design:…`、`theme:…`、裸 `AST-094`）以及带已知前缀的声明 id（`FR12`、`INV3`、`DEC-4`、`AR2`、`DR1` 等）；
- Pull Request 和 Issue 引用：`#1234`、`PR #77`、`issue 12`，以及 GitHub Pull Request、Issue、讨论和提交 URL。3、6 或 8 位数字的裸 `#123` 可能是十六进制颜色，因此不报告；
- 日期戳：引号外的 ISO 日期（`2026-10-31`）和 `as of Oct 2026`；
- 引用块：