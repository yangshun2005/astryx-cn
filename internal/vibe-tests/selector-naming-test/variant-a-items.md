# Selector — 下拉选择器组件

用于从列表中选择项的下拉选择器。支持简单字符串、带图标/描述富项、分组和分隔线。

## 导入

```tsx
import {Selector, SelectorItem} from '@astryxdesign/core';
```

## 基本用法

```tsx
// 简单字符串项
<Selector
  label="Fruit"
  items={['Apple', 'Banana', 'Orange']}
  value={fruit}
  onChange={setFruit}
  placeholder="Select a fruit..."
/>

// 带图标和描述的富项
<Selector
  label="Status"
  items={[
    { value: 'active', label: 'Active', icon: 'checkCircle' },
    { value: 'paused', label: 'Paused', icon: 'pause' },
    { value: 'archived', label: 'Archived', icon: 'archive' },
  ]}
  value={status}
  onChange={setStatus}
/>

// 分组和分隔线
<Selector
  label="Action"
  items={[
    { type: 'section', title: 'Edit', items: [
      { value: 'cut', label: 'Cut' },
      { value: 'copy', label: 'Copy' },
    ]},
    { type: 'divider' },
    { type: 'section', title: 'View', items: [
      { value: 'zoom-in', label: 'Zoom In' },
      { value: 'zoom-out', label: 'Zoom Out' },
    ]},
  ]}
  value={action}
  onChange={setAction}
/>
```

## 自定义项渲染

```tsx
<Selector
  label="User"
  items={users.map(u => ({value: u.id, label: u.name, icon: 'user'}))}
  value={selectedUser}
  onChange={setSelectedUser}
  renderOption={item => (
    <SelectorItem
      icon={item.icon}
      label={item.label}
      description={item.email}
    />
  )}
/>
```

## Props

### Selector

| Prop          | Type                                                          | Default     | Description                           |
| ------------- | ------------------------------------------------------------- | ----------- | ------------------------------------- |
| label         | string                                                        | required    | 标签文本（为无障碍始终渲染） |
| items         | Array<string \| ItemData \| Divider \| Section>               | required    | 要显示的项                  |
| value         | string                                                        | —           | 当前选中的值              |
| onChange      | (value: string) => void                                       | —           | 选择变更回调             |
| changeAction  | (value: string) => Promise<void>                              | —           | 变更时执行的异步操作                |
| placeholder   | string                                                        | 'Select...' | 未选中时的占位文本     |
| size          | 'sm' \| 'md' \| 'lg'                                          | 'md'        | 触发器尺寸                          |
| isDisabled    | boolean                                                       | false       | 禁用状态                        |
| isLabelHidden | boolean                                                       | false       | 视觉上隐藏标签                   |
| description   | string                                                        | —           | 标签下方的帮助文本                 |
| isOptional    | boolean                                                       | false       | 显示可选指示               |
| isRequired    | boolean                                                       | false       | 显示必填指示               |
| status        | { type: 'warning' \| 'error' \| 'success', message?: string } | —           | 校验状态                     |
| renderOption  | (item: ItemData) => ReactNode                                 | —           | 自定义项渲染器                  |

### SelectorItem（用于自定义渲染）

| Prop        | Type      | Description        |
| ----------- | --------- | ------------------ |
| icon        | IconType  | 前置图标       |
| label       | ReactNode | 主文本       |
| description | ReactNode | 次要文本     |
| children    | ReactNode | 附加内容 |

### 项类型

```ts
type ItemData = {
  value: string;
  label?: string;
  disabled?: boolean;
  icon?: IconType;
};
type Divider = {type: 'divider'};
type Section = {type: 'section'; title?: string; items: ItemData[]};
```