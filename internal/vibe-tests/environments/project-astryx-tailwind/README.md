# Astryx + Tailwind CSS

本项目将 Astryx 组件与 Tailwind CSS 搭配使用。写代码前，先用 CLI 查询组件 Props 和用法：

```bash
npx astryx component --list              # 列出所有可用组件
npx astryx component Button              # 查询 Props、变体与用法
npx astryx component IconButton          # 每个组件有独立条目
```

组件使用：

- Tailwind CSS 工具类处理布局和自定义样式
- Astryx Tailwind 桥接令牌（`bg-surface`、`text-primary`）处理设计令牌
- React 19

## 导入方式

每个组件从各自的子路径导入：

```tsx
import {Button} from '@astryxdesign/core/Button';
import {IconButton} from '@astryxdesign/core/IconButton';
import {Card} from '@astryxdesign/core/Card';
import {Text, Heading} from '@astryxdesign/core/Text';
import {ToggleButton, ToggleButtonGroup} from '@astryxdesign/core/ToggleButton';
import {Theme} from '@astryxdesign/core/theme';
```

## 事件处理器

Astryx 是 React DOM 库。使用标准 React DOM 事件处理器 Props，如
`onClick`、`onChange`、`onKeyDown`。按钮激活用 `onClick`：

```tsx
<Button label="Save" onClick={() => handleSave()} />
```

除非组件明确说明，否则不要使用 `onPress` 这类跨平台激活 Props。