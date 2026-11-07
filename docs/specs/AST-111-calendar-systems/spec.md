---
schema_version: 4
template_version: 2
kind: system-spec
id: spec:AST-111
authority: draft
archive_reason: null
superseded_by: null
approved_by: nynexman4464
approved_at: 2026-11-04
phase: accepted
owners: [nynexman4464]
affects_architecture:
  [architecture:internationalization, architecture:public-component-api]
affects_families: [family:input-fields]
affects_contributing: []
affects_consumer_docs:
  [
    internationalization,
    Calendar,
    DateInput,
    DateRangeInput,
    DateTimeInput,
    Timestamp,
  ]
---

# 日历系统系统规范

<!-- path-applicability:v1 -->

```json
{
  "paths": [
    "packages/core/src/i18n/**",
    "packages/core/src/utils/plainDate.ts",
    "packages/core/src/utils/dateParser.ts",
    "packages/core/src/Calendar/**",
    "packages/core/src/DateInput/**",
    "packages/core/src/DateRangeInput/**",
    "packages/core/src/DateTimeInput/**",
    "packages/core/src/Timestamp/**",
    "packages/core/src/PowerSearch/formatFilterValue.ts",
    "packages/charts/src/formatters.ts",
    "packages/lab/src/Chart/formatters.ts",
    "packages/lab/src/Schedule/**"
  ]
}
```

<!-- 描述系统，而非项目：用现在时，说明它做什么。记录中不写提案、历史、Pull Request 或研究；参见 docs/contributing/spec-writing.md，并在 Pull Request 中报告其评分结果。 -->

## 意图

使用非公历日历的人以该日历查看、输入和导航日期。波斯用户选择 8 Aban 1405，希伯来用户在闰年看到 Adar I 和 Adar II，泰国用户看到佛历年。日历由应用选择。存储值不变：每个日期组件读写公历 ISO 日期，因此应用数据、表单和服务端代码保持日历中立。

## 非目标

- 从不指定日历的区域设置标签推断日历。单独的 `fa-IR` 仍为公历。
- 携带自身日历的值，如 `2024-02-10[u-ca=hebrew]`。
- 公开的日历适配器或插件 API。
- 由 Astryx 负责的日历运算。由 Temporal 负责。
- FR1 之外的日历标识符，包括观测型伊斯兰历。
- 明治（1868）之前的日本年号。
- 1900 年之前日期的保证行为。此类日期按 `Temporal` 的计算显示和导航，尽力而为；键入、回读和年号显示仅在 1900 至 2100 年有规范和测试。更好的支持（尤其是键入输入）待应用确有需要时再提供。
- 传统数字作为键入输入：希伯来字母数字、中国历法的日期数字、干支纪年。
- `DateRangeInput` 中的键入范围输入。
- ISO 8601 之外的周编号。
- 满足本契约的等效内部实现仍然有效。
- 内部模块、文件、函数名、算法、数据结构、存储布局、清单、日志、锁、事务协议以及 CI 作业/工作流拓扑属于架构或实现，除非调用方或互操作系统有意将该确切机制作为公开协议依赖。此时需说明谁依赖它，以及为何等效实现无法满足契约。

## 公开契约

```tsx
<InternationalizationProvider locale="fa-IR" calendar="persian">
  {/* value 和 onChange 保持公历 ISO，例如 '2026-10-30' */}
  <DateInput label="Start date" value={date} onChange={setDate} />
  {/* 单个组件可以使用不同日历 */}
  <Calendar calendar="gregory" value={day} onChange={setDay} />
</InternationalizationProvider>;

{
  /* 区域设置中的 -u-ca- 关键字不选择日历 */
}
<InternationalizationProvider locale="th-TH-u-ca-buddhist" calendar="buddhist">
  …
</InternationalizationProvider>;
```

```ts
// gregory、iso8601、buddhist、roc 和 japanese 之外的日历需要
// Temporal。运行时缺少它时，应用在服务端和浏览器中加载 polyfill。
import 'temporal-polyfill/full/global';
```

`calendar` 接受 `CalendarSystem`，从 `@astryxdesign/core/i18n` 导出。`InternationalizationProvider`、`Calendar`、`DateInput`、`DateRangeInput` 和 `DateTimeInput` 接受它。`@astryxdesign/core/utils` 中的 `formatSharedDate`，以及 `@astryxdesign/charts` 和 Lab `Chart` 中的 `shortDate` 和 `monthYear`，接受可选的 `calendar` 参数，默认 `gregory`（FR18）。`@astryxdesign/core/Calendar` 中公开的 `useCalendarDays` 和 `useCalendarNavigation` Hook 接受可选的 `calendar`，默认 `gregory`，且从不读取 Provider 的日历（FR20）。

## 需求

### 日历选择与值

- **FR1 — 支持的日历。** `CalendarSystem` 为 `gregory`、`iso8601`、`buddhist`、`roc`、`japanese`、`persian`、`indian`、`hebrew`、`chinese`、`dangi`、`coptic`、`ethiopic`、`ethioaa`、`islamic-civil`、`islamic-tbla` 或 `islamic-umalqura`：即 ECMA-402 和 Temporal 要求的日历。其他标识符解析为 `gregory`，并在开发环境警告一次。
- **FR2 — 应用选择日历。** 日期组件按以下顺序使用：自身的 `calendar` prop；其 `InternationalizationProvider` 的 `calendar` prop；否则 `gregory`。区域设置从不选择日历：其默认日历和其中的 `-u-ca-` 关键字均不适用。日历与语言、数字和方向无关：`locale="en" calendar="persian"` 以英文显示波斯日期，`locale="fa-IR"` 无日历则以波斯文显示公历日期。
- **FR3 — 值保持公历 ISO。** 日历从不改变组件读取或报告的内容。`value`、`defaultValue`、`onChange`、`min`、`max`、`focusDate`、`onFocusDateChange`、`dateConstraints` 收到的 `Date`、表单提交以及每个 `system_*` 格式在任何日历下都保持公历 ISO。
- **FR4 — 结构来自日历。** 月长、一年中的月数、闰月和年号来自日历。组件中的任何网格、滚轮、列表或步进，或给定日历的 Hook（FR20），都不假设 12 个月或公历月长。闰月是独立月份，有自己的名称：希伯来闰年的 Adar I 和 Adar II，或 `chinese` 或 `dangi` 年的闰月。
- **FR5 — 部分日历需要 Temporal。** `gregory`、`iso8601`、`buddhist`、`roc` 和 `japanese` 处处可用。其他日历使用运行时的 `Temporal`，原生或应用加载的 polyfill；Astryx 不捆绑任何 polyfill。缺少时，解析这些日历的组件显示并接受公历日期，并在开发环境警告一次，指明 polyfill。服务端和浏览器以相同方式解析日历时标记一致，因此服务端渲染的应用也在服务端加载 `Temporal`。FR19 指出唯一例外。

### Calendar

- **FR6 — 网格显示日历月。** 月份标题、网格的可访问名称、日期按钮名称和播报使用日历。每个网格布局一个日历月：其第一天位于对应星期列，外部日期填充其余单元格。默认固定六行网格适用于每个月，包括科普特和埃塞俄比亚历法的五或六天第十三月；`hasVariableRowCount` 让每月只保留所需行数。日期数字在每个日历中使用区域设置的数字，与标题一致。
- **FR7 — 导航按日历月和年步进。** 上一页和下一页移动一个日历月。PageUp 和 PageDown 将聚焦日期移动一个日历月，保持其月内日期，并钳制到目标月长度。Shift+PageUp 和 Shift+PageDown 将聚焦日期移动一个日历年，到该年的同一月；目标年没有的闰月变为其平月对应（Adar I 变为 Adar），目标月缺少的日期钳制到其最后一天。`min` 和 `max` 在每个日历中按日期限制导航。
- **FR8 — 周编号为 ISO 8601。** `hasWeekNumbers` 在每个日历中显示 ISO 8601 周编号，其文档也如此说明。

### 键入输入

这些需求适用于 `DateInput` 和 `DateTimeInput` 的文本字段。

- **FR9 — 字段读取以日历书写的日期。** 字段接受按 `Intl.DateTimeFormat` 为该区域设置和日历书写方式书写的日期。全数字输入遵循区域设置的数字字段顺序；存在月份名称时，三位或更多位的数字无论位置都是年份。字段接受任何 Unicode 十进制数字（混合文字）、区域设置和英文产生的每种宽度的月份和年号名称，忽略大小写和拉丁变音符号。键入的月份数字表示该日历下区域设置数字月份的含义：希伯来历为键入年中的月份位置，`chinese` 和 `dangi` 为带区域设置闰月标记的月份数字。希伯来闰年键入的普通 Adar 为 Adar II。在公历中，不匹配任何这些形式的文本，或命名区域设置数字顺序没有的日期，当 `parseDateInput` 能读取时仍在 Enter 或失焦时提交：`en-US` 中的 `13/5/2026` 提交 2026 年 5 月 13 日。否则适用 FR11。一位数年份不是日期，即使 `parseDateInput` 能读取。
- **FR10 — 年份是日历的年份。** 键入的年份是活动日历的年份，从不是转换后的公历年：佛历字段中的 `2026` 是佛历 2026 年。格式提示和提交文本在区域设置格式包含年号时显示年号，如 `พ.ศ.`（B.E.，佛历纪元）或 `民國`（Minguo，民国纪元），并在仅年份有歧义时添加，遵循 ICU4X 年份歧义模型：日历纪元纪元之前的年份总是显示年号；在 `japanese` 和 `roc` 日历中（年号在 1900 至 2100 年间变化），省略年号的格式会获得年号，在保持区域设置字段顺序时以窄格式书写；若所有年号宽度都会重排字段，则仅最新年号之前的日期获得年号，因为无年号的年份读作最新年号。无键入年号时，一或两位日本年份属于当前年号，四位日本年份为公历；最多三位的民国年份属于民國（Minguo）年号；埃塞俄比亚年份属于 AM 年号。两位数年份解析为日历当前年之前 80 年和之后 19 年内的年份。年号之外的日本年号年份被拒绝。不应用默认年份范围；`min` 和 `max` 捕获在错误日历中键入的年份。
- **FR11 — 日历没有的日期被拒绝。** 字段拒绝并播报日历中不存在的日期，如平波斯年的 30 Esfand、平希伯来年的 Adar I，或平年埃塞俄比亚第十三月第 6 天。它从不调整日期。键入日期超出 `min`、`max` 或 `dateConstraints` 时显示并播报越界错误，以日历书写边界。
- **FR12 — `onChange` 对每个完整日期触发一次。** 用户键入时，`onChange` 仅在每个字段都存在、年份完整或为带年号的年号年份、无尾随分隔符、且最后字段无法再接受数字时触发。两位数年份和无年份输入在 Enter 或失焦时提交。`onChange` 对每个不同值最多触发一次。此规则也适用于公历。
- **FR13 — 提交文本回读为同一日期。** `date`、`date_long` 和 `date_weekday` 以日历渲染。`system_date` 保持公历 ISO，字段也接受其 ISO 输入。对于每个日历、区域设置和命名格式，字段将其提交文本回读为同一日期，覆盖 1900 至 2100 年的每一天。`chinese` 和 `dangi` 提交文本包含相关公历年，其月和日与同一环境中字段自身的运算一致。
- **FR14 — 自定义显示文本仅用于显示。** 自定义 `format` 函数产生且字段无法回读的文本，编辑后无法重新提交。

### 呈现

- **FR15 — 非公历日历使用 Astryx 选择器。** `<input type="date">` 按规范为公历，其选择器显示的日历来自设备而非应用。使用 `gregory` 或 `iso8601` 之外的日历时，每个显示原生控件的呈现都改用 Astryx 表面：精细指针上的弹出框和粗略指针上的底部面板。这涵盖 `adaptive-native`（默认）、已弃用的 `nativePicker` 值，以及显式 `presentation="native"`（后者在开发环境也警告）。
- **FR16 — 底部面板遵循日历。** 其滚动月历按 FR6 对 `Calendar` 的要求显示日历月、标签和数字，并保持固定面板高度。其月份滚轮列出所选日历年的月份，包括按名称的闰月。年份变化时，若新年有该月则保留，否则变为其平月对应。年号标签在日历有年号时包含年号，如 `令和8`（Reiwa 8）。年号变化分割的年份采用所选日期的年号，如原生 iOS 选择器：2019 年选一月读作 `平成31`（Heisei 31），选七月读作 `令和元`（Reiwa 1，年号第一年）。
- **FR17 — `DateRangeInput` 以日历显示范围。** 其触发文本和日历使用日历。它没有键入输入。

### 共享格式化器

- **FR18 — 内置日期格式遵循 Provider 的日历。** Timestamp 的含日期格式、PowerSearch 日期值和 Schedule 日期标签以其 `InternationalizationProvider` 的日历渲染（FR2，无组件 prop）。仅时间格式、相对格式和 `system_*` 格式不变。纯日期辅助函数保持公历默认并接受显式日历：`plainDateFormat` 通过其现有 `calendar` 格式选项，`formatSharedDate`、`shortDate` 和 `monthYear` 通过可选 `calendar` 参数。Astryx 调用点传递解析后的日历。
- **FR20 — 公开日历 Hook 接受显式日历。** `useCalendarDays` 和 `useCalendarNavigation` 接受可选的 `calendar`，默认 `gregory`；它们从不读取 Provider 的日历，因此应用添加 Provider 日历时调用方的数字保持含义。星期名称和 `monthYearLabel` 仍使用 Provider 的区域设置。给定日历时，`useCalendarDays` 将 `year` 和 `month` 读作日历的年份和该年中的月份位置，如 `Temporal.PlainDate` 编号（希伯来闰年为 1 至 13），每天保持其公历日期，而 `dayNumber` 是其在日历月中的日期。`useCalendarNavigation` 按日历月步进，以日历写入 `monthYearLabel`，并以日历数字报告可见月份，以便馈送给 `useCalendarDays`。无 `calendar` 时，两者行为与已发布版本一致。

### 阴阳历

- **FR19 — 阴阳历日期遵循运行时。** `chinese` 和 `dangi` 的月起始来自运行时的 `Temporal`。原生 `Temporal` 在 1900 至 2100 年与官方天文台数据一致。读取平台 `Intl` 数据的 polyfill 可能使该范围内九个 `chinese` 和四个 `dangi` 月起始相差一天，包括 2027 `chinese` 年的第一个月。当服务端和浏览器使用的实现月起始不一致时，浏览器对该月的渲染与服务端标记不同，React 在水合期间在客户端重新渲染该部分。国际化指南和日期组件文档说明这一点。它们建议仅在缺少 `Temporal` 时加载 polyfill，并在服务端渲染应用需要相同 `chinese` 或 `dangi` 标记时在服务端和浏览器加载相同实现。

### 可访问性契约

- **AR1 — 播报日期与可见日历一致。** 日期按钮名称、网格名称、选择和导航播报、错误消息以及错误消息中的边界使用日历。
- **AR2 — 字段描述其格式。** 使用 `gregory` 或 `iso8601` 之外的日历时，无调用方 `placeholder` 的空聚焦字段以日历数字模式显示示例日期：字段接受今天时用今天，否则用 `min`、`max` 和 `dateConstraints` 内它接受的最近日期。调用方的 `placeholder` 总是优先。字段描述仅在其日历与 Provider 日历不同时指明日历，因此单一日历页面不重复，混合日历页面说明哪个字段使用哪个日历。
- **AR3 — 每种拒绝有自己的消息。** 不可读文本、日历没有的日期和越界日期各自播报不同消息。

### 平台支持

- 支持的功能/引擎下限：公历对齐日历在每个支持的浏览器和渲染器中可用。其他日历需要 `Temporal`：运行时原生，或应用加载的 polyfill。国际化指南列出附带它的运行时。
- 不支持的行为：无 `Temporal` 时，FR5 的公历回退和开发警告。macOS 和 iOS 上的 Safari 缺少 `Temporal`，`spec:AST-013` FR3 明确点名 iOS Safari，因此每个解析日历的组件记录 `spec:AST-013` FR12 例外：`Calendar`、`DateInput`、`DateRangeInput`、`DateTimeInput