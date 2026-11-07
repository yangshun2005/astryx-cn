# Stepper

```
import {Step, Stepper} from '@astryxdesign/core/Stepper';
```

水平 `Stepper` 渲染带编号的步骤序列，每个步骤下方带标签。它会自行测量宽度。当每个步骤可用宽度不足约 112px 时，stepper 会**折叠**：标签消失，只留下分段进度条，进度条下方出现一行，显示当前步骤名称，并在设置了 `onStepClick` 时显示 Previous/Next 控件。垂直 stepper 永不折叠。

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
| `hasControls` | `boolean` | `true` |
| `hasLabel` | `boolean` | `true` |

`activeStep` 是当前步骤的索引（从 0 开始）。`onStepClick` 使步骤可点击，用于非线性导航。

`hasControls` 设置折叠后的 stepper 是否在进度条下方显示 Previous/Next 控件；仅当设置了 `onStepClick` 时才会出现。`hasLabel` 设置折叠后的 stepper 是否在进度条下方显示当前步骤名称。无论哪种情况，名称都保留在无障碍序列中。stepper 宽度足够显示标签时，两者均不生效。

`Step` 接受 `step`（从 0 开始的索引）、`label`，以及可选的 `description`、`status`（`'accent' | 'success' | 'warning' | 'error'`）和 `isOptional`。

## 示例

默认，无配置：

```tsx
<Stepper activeStep={step} onStepClick={setStep}>
  <Step step={0} label="Cart" />
  <Step step={1} label="Address" />
  <Step step={2} label="Payment" />
</Stepper>
```

关闭折叠行的一半：

```tsx
<Stepper activeStep={step} onStepClick={setStep} hasControls={false}>
  <Step step={0} label="Cart" />
  <Step step={1} label="Address" />
  <Step step={2} label="Payment" />
</Stepper>
```

## 反模式

不要用 CSS 隐藏折叠行：

```css
/* Don't do this */
[data-astryx-stepper-summary] { display: none; }
```

这会让控件留在 Tab 顺序中，却没有任何可见内容。请改用 `hasControls` / `hasLabel`。