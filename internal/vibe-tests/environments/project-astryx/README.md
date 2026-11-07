# Astryx

本项目使用 Astryx 组件。写代码前，先用 CLI 查询组件 Props 和用法：

```bash
npx astryx component --list              # 列出所有可用组件
npx astryx component Button              # 查询 Props、变体与用法
npx astryx component IconButton          # 每个组件有独立条目
```

如果 CLI 不可用，先安装依赖：

```bash
npm install --include=dev
```

组件使用：

- StyleX（`@stylexjs/stylex`）处理样式
- React 19

## 使用 StyleX 编写样式

自定义样式必须使用 `stylex.create()`。普通对象无效：

```tsx
import stylex from '@stylexjs/stylex';

const styles = stylex.create({
  container: {
    padding: 16,
    backgroundColor: '#f5f5f5',
  },
});

// 通过 xstyle prop 应用到组件：
<Card xstyle={styles.container}>...</Card>

// 通过 stylex.props() 应用到 HTML 元素：
<div {...stylex.props(styles.container)}>...</div>
```

**重要：** 切勿向 `xstyle` 传入普通 `{padding: '16px'}` 对象。始终使用 `stylex.create()`。

## 导入方式

每个组件从各自的子路径导入：

```tsx
import {Button} from '@astryxdesign/core/Button';
import {IconButton} from '@astryxdesign/core/IconButton';
import {Card} from '@astryxdesign/core/Card';
import {Text, Heading} from '@astryxdesign/core/Text';
import {useTheme} from '@astryxdesign/core/theme';
```