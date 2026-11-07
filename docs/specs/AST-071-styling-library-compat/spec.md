---
schema_version: 4
template_version: 2
kind: system-spec
id: spec:AST-071
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-10-11
phase: accepted
owners: [cixzhang]
affects_architecture: [architecture:public-component-api]
affects_families: []
affects_contributing: []
affects_consumer_docs: []
---

# 样式库兼容性系统规范

<!-- path-applicability:v1 -->

```json
{
  "paths": [
    "packages/core/src/**",
    "packages/lab/src/**",
    "packages/charts/src/**",
    "packages/richtext/src/**",
    "packages/vega/src/**"
  ]
}
```

## 意图

产品用其代码库已有的方案为 Astryx 组件设置样式：StyleX、类名 CSS（CSS modules、工具类、全局样式表）或内联样式。组件接受的每一种样式输入都必须能从上述任一系统到达其元素，因此组件的任何部分都不能仅靠 StyleX 设置样式。

## 非目标

- 组件是否为某个部分暴露样式。部分输入是新的公开 prop，按 `spec:AST-002/DEC-1` 准入；本规范约束的是部分输入准入之后的形态。
- 多个样式输入在同一元素上的组合顺序，由 `architecture:public-component-api` INV6 负责。
- 主题令牌与主题目标，由 `architecture:theme-tokens` 及主题相关记录负责。
- 满足本契约的等价内部实现仍然有效。

## 需求

- **FR1 — 部分的样式输入必须成套提供。** 组件若为非根元素接受样式输入，则必须同时接受该元素的 `<part>Xstyle`、`<part>ClassName` 和 `<part>Style`，且三者使用相同的 `<part>` 词。三者必须到达同一元素，并按根元素的 `xstyle`、`className`、`style` 的方式组合。
- **FR2 — 不允许仅支持 StyleX 的样式输入。** 为元素设置样式的公开 prop 不得只接受 StyleX 样式，除非同一元素也通过公开 prop 接受类名和内联样式。

### 平台支持

- 支持的 feature/engine 下限：不变；本契约不新增任何 CSS 特性。
- 不支持的行为：无；类名和内联样式在所有支持的引擎中均可用。
- 浏览器证据：不需要；FR1 和 FR2 在声明层和渲染层验证。

## 现状影响

- `architecture:public-component-api` 新增一条不变量：部分的样式输入必须成套提供，由 `spec:AST-071/DEC-1` 决定。
- 不符合 FR1 和 FR2 的已发布公开 prop：`ComplexSelector.contentXstyle`、`ContextMenu.triggerXstyle` 和 `BaseTypeahead.inputXStyle`。它们是已知缺口，不是先例：不得援引它们来准入新的部分输入。

## 验证

| 契约 | 验证                                                                     | 代表性状态                                     | 变更或失败预期                                                               |
| ---- | ------------------------------------------------------------------------ | ---------------------------------------------- | ---------------------------------------------------------------------------- |
| FR1  | 对生成的声明清单进行公开 API 审查；组件渲染测试                          | 部分输入分别由 StyleX、类名和内联样式设置     | 只有 `<part>Xstyle` 而缺少 `<part>ClassName` 和 `<part>Style`，或未落到同一元素，即失败 |
| FR2  | 对每个以 StyleX 样式为类型的导出 prop 进行公开 API 审查                  | 导出组件的根输入和部分输入                     | 以 StyleX 为类型的 prop 在其元素上没有对应的类名和内联样式，即失败           |

已知验证缺口：没有自动化检查强制执行 FR1 或 FR2；由公开 API 审查执行。

## 决策日志

### DEC-1 — 每种样式输入都能从每种样式系统生效

**引用：** `spec:AST-071/DEC-1`
**决策者：** `cixzhang`，2026-10-09

Astryx 被不使用 StyleX 的产品采用。只能通过 `xstyle` 到达的部分，会让用类名或内联样式设置样式的代码库无法触达该部分，而每个这样的 prop 都会成为下一个部分照抄的先例。因此，部分的样式输入必须是根元素已接受的完整集合，否则不予准入。

已否决：只准入部分的 `xstyle`，按需再补对应项——非 StyleX 调用方在有人提出请求前一直被排除在外。

## 待解决问题

- **OQ1 — 已知不符合的 prop 如何达到合规**（`human-api`）：`ComplexSelector.contentXstyle`、`ContextMenu.triggerXstyle` 和 `BaseTypeahead.inputXStyle`。