---
schema_version: 1
template_version: 1
kind: design
id: design:user-states
authority: current
archive_reason: null
superseded_by: null
approved_by: cixzhang
approved_at: 2026-09-19
owners: [ernestt, cixzhang]
review_triggers: [visual, interaction, accessibility]
verified_by:
  [
    packages/core/src/__tests__/PressedState.a11y.chromium.spec.ts,
    packages/core/src/CheckboxInput/CheckboxInput.test.tsx,
    packages/core/src/Collapsible/Collapsible.test.tsx,
    packages/core/src/Link/Link.test.tsx,
    packages/core/src/RadioList/RadioList.test.tsx,
    packages/core/src/SegmentedControl/SegmentedControl.test.tsx,
    packages/core/src/Slider/Slider.test.tsx,
    packages/core/src/Switch/Switch.test.tsx,
    packages/core/src/TabList/TabList.test.tsx,
  ]
architecture: [architecture:interaction-modality, architecture:theme-tokens]
components:
  [
    component:CheckboxInput,
    component:Collapsible,
    component:Link,
    component:Slider,
    component:Switch,
    component:TabList,
    component:RadioList,
    component:SegmentedControl,
  ]
families: []
deciding_specs: []
---

# 用户状态设计规范

## 用户意图

指针用户按住或拖拽已启用的控件时，应立即在被操作的表面上获得一致的反馈。该反馈不得暗示已禁用的控件有响应，也不得抹除持久选中的表面。

## 设计原则

- **DR1 — 按压被操作的表面。** 指针按住或拖拽期间，下表中每个已启用的表面必须在指定的视觉部分绘制 `--color-overlay-pressed`。该叠加层是瞬态的，按压或拖拽结束时必须清除。在任何指针下，叠加层的时序遵循 `spec:AST-094` FR11 的按压配置。
- **DR2 — 禁用即无响应。** 表中的禁用控件必须保持不可操作，且不得获得按压视觉处理。
- **DR3 — 持久选中优先。** 选中的 SegmentedControl 项在指针按住期间必须保留其凸起的选中表面；只有未选中项获得按压叠加层。在任何指针下，该项都保留该表面，其按压时序遵循 `spec:AST-094` FR11。

## 结构与层级

| 角色             | 用途                     | 必需关系                                             |
| ---------------- | ------------------------ | ---------------------------------------------------- |
| 交互目标         | 接收指针按住或拖拽       | 拥有或控制表中指定的绘制表面                         |
| 按压叠加层       | 确认当前操作             | 叠加在现有填充之上，不替换内容或持久状态             |
| 持久选中表面     | 传达已选中的分段         | 指针按住期间视觉上保持不变                           |

## 状态表示

| 组件             | 已启用指针按住或拖拽                                                                                      | 禁用指针按住                                                   |
| ---------------- | --------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| CheckboxInput    | 指示器绘制 `--color-overlay-pressed`                                                                      | 指示器保持不变                                                 |
| Collapsible      | 触发行绘制 `--color-overlay-pressed`                                                                      | 触发行保持不变，且不展开或折叠                                 |
| Link             | 文本表面绘制 `--color-overlay-pressed`                                                                    | 文本表面保持不变，且不激活                                     |
| Slider           | 仅被拖拽的滑块绘制 `--color-overlay-pressed`                                                              | 滑块保持不变，且值不变                                         |
| Switch           | 轨道和滑块均绘制 `--color-overlay-pressed`                                                                | 轨道和滑块保持不变，且值不变                                   |
| TabList Tab      | 标签表面绘制 `--color-overlay-pressed`                                                                    | 标签表面保持不变，且选中项不变                                 |
| RadioList        | 指示器绘制 `--color-overlay-pressed`                                                                      | 指示器保持不变，且选中项不变                                   |
| SegmentedControl | 未选中项绘制 `--color-overlay-pressed`；选中项保留其凸起表面不变                                          | 项表面保持不变，且选中项不变                                   |

此表是当前完整的声明。它不批准其他组件的按压处理，也不决定它们的状态设计。

## 响应式与输入行为

- **DR4 — 跟随真实交互。** 按住驱动的控件仅在指针按下期间保留叠加层。Slider 跟随其现有拖拽状态，因此叠加层停留在被拖拽的滑块上，包括拖拽从轨道开始的情况。
- **DR5 — 保留静止结果。** 释放或取消后，组件必须恢复到原本应有的相同视觉状态；按压反馈不产生持久状态。

## 无障碍意图

按压反馈补充而非替代控件的语义状态、标签、焦点指示或禁用行为。以禁用状态暴露的控件必须保持不可操作，即使其实现为解释不可用原因而保持可聚焦。

## 代表性示例

- 按住已启用的 CheckboxInput 行会加深其指示器；在外部释放且未激活时，恢复原始指示器像素。
- 拖拽范围 Slider 仅绘制当前被移动的滑块。
- 按住选中的 SegmentedControl 项时，其凸起表面保持不变；按住未选中项时绘制按压叠加层。

## 视觉参考

由 `packages/core/src/__tests__/PressedState.a11y.chromium.spec.ts` 生成的真实 Chromium 证据，捕获了八个组件的匹配静止、按住指针、释放、禁用和选中例外表面。CI 在 `pressed-state-evidence` 构建产物中发布这些 PNG、像素差异、Chromium 版本、视口、story ID、哈希和确切的 PR-head SHA。生成的证据验证此记录，但不是规范性设计资产。

## 组件契约链接

`components` frontmatter 列表是 DR1–DR5 的机器可读采用矩阵。组件记录不重复此跨组件决策。

## 决策日志

### DEC-1 — 在限定的八组件矩阵上使用系统按压叠加层

**参考：** `design:user-states/DEC-1`
**决策者：** `cixzhang`，2026-09-19

一致的反馈帮助指针用户理解哪个已启用的表面正在响应。该决策有意限定于矩阵中的表面，保留禁用行为，并保持已选中分段的凸起表面稳定。

## 待解决问题

此限定的按压状态决策无待解决问题。悬停、焦点、选中分段例外之外的选中处理，以及其他组件的采用，均不属于此记录当前声明的范围。

## 内容边界

此文件定义了已批准的跨组件按压状态意图及其限定的采用矩阵。组件实现拥有选择器和事件机制；架构记录拥有交互模态和 token 行为；测试和 CI 构建产物拥有当前一致性证据。