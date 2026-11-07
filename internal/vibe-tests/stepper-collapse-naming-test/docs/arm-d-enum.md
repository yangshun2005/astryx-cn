# Stepper

```
import {Step, Stepper} from '@astryxdesign/core/Stepper';
```

水平 `Stepper` 渲染带编号的步骤序列，每个步骤下方带标签。它会测量自身宽度。当每个步骤可用宽度不足约 112px 时，stepper 会**折叠**：标签消失，只留下分段进度轨道，轨道下方出现一行，显示当前步骤名称；若设置了 `onStepClick`，还显示 Previous/Next 控件。垂直 stepper 永不折叠。

## Props

| prop | type | default |
| --- | --- | --- |
| `activeStep` | `number` | — |
| `children` | `ReactNode` | — |
| `onStepClick` | `(index: number) => void` | — |
| `label` | `string` | `'Progress'` |
| `orientation` | `'horizontal' \| 'vertical'` | `'horizontal'` |
| `density` | `'compact' \| 'balanced' \| 'spacious'` | `'balanced'` |
| `indicatorPosition` | `'separated' \| 'on-track'` | `'separated'` |
| `collapsedSummary` | `'auto' \| 'label' \| 'controls' \| 'none'` | `'auto'` |

`activeStep` 是当前步骤的从零开始的索引。`onStepClick` 使步骤可点击，用于非线性导航。

`collapsedSummary` 设置折叠后的 stepper 在轨道下方显示的内容。`'auto'` 显示当前步骤名称，并在设置了 `onStepClick` 时显示 Previous/Next 控件。`'label'` 仅显示名称。`'controls'` 仅显示控件。`'none'` 不显示任何内容，只留裸轨道。无论取何值，名称都保留在无障碍序列中。当 stepper 宽度足以显示标签时，此属性无效。

`Step` 接受 `step`（从零开始的索引）、`label`，以及可选的 `description`、`status`（`'accent' | 'success' | 'warning' | 'error'`）和 `isOptional`。

## 示例

默认，无配置：

```tsx
<Stepper activeStep={step} onStepClick={setStep}>
  <Step step={0} label="Cart" />
  <Step step={1} label="Address" />
  <Step step={2} label="Payment" />
</Stepper>
```

关闭折叠行的一半内容：

```tsx
<Stepper activeStep={step} onStepClick={setStep} collapsedSummary="label">
  <Step step={0} label="Cart" />
  <Step step={1} label="Address" />
  <Step step={2} label="Payment" />
</Stepper>
```

## 反模式

不要用 CSS 隐藏折叠行：

```css
/* 不要这样做 */
[data-astryx-stepper-summary] { display: none; }
```

这会让控件留在 Tab 顺序中，却没有任何可见内容。请改用 `collapsedSummary`。