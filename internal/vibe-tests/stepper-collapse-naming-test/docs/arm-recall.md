# Stepper

```
import {Step, Stepper} from '@astryxdesign/core/Stepper';
```

水平 `Stepper` 渲染带编号的步骤序列，每个步骤下方带标签。它会自行测量宽度。当每个步骤可用宽度不足约 112px 时，stepper 会**折叠**：标签消失，只留下分段进度条，进度条下方出现一行，显示当前步骤名称，若设置了 `onStepClick` 还会显示 Previous/Next 控件。垂直 stepper 永不折叠。

## Props

| prop | 类型 | 默认值 |
| --- | --- | --- |
| `activeStep` | `number` | — |
| `children` | `ReactNode` | — |
| `onStepClick` | `(index: number) => void` | — |
| `label` | `string` | `'Progress'` |
| `orientation` | `'horizontal' \| 'vertical'` | `'horizontal'` |
| `density` | `'compact' \| 'balanced' \| 'spacious'` | `'balanced'` |
| `indicatorPosition` | `'separated' \| 'on-track'` | `'separated'` |

`activeStep` 是当前步骤的零基索引。`onStepClick` 使步骤可点击，用于非线性导航。

折叠行可配置。Previous/Next 控件和当前步骤名称可分别关闭，因此已自带导航或步骤标题的页面不会重复显示。无论是否关闭，名称仍保留在无障碍序列中；当 stepper 宽度足以显示标签时，这些配置均不生效。**实现此功能的 props 未列在本参考中。**

`Step` 接受 `step`（零基索引）、`label`，以及可选的 `description`、`status`（`'accent' | 'success' | 'warning' | 'error'`）和 `isOptional`。

## 示例

默认，无配置：

```tsx
<Stepper activeStep={step} onStepClick={setStep}>
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

这会让控件留在 Tab 顺序中却不可见。请改用组件自身的 API。