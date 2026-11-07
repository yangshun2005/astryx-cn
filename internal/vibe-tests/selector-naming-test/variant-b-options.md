# Selector — 下拉选择器组件

用于从选项列表中选择的下拉选择器。支持简单字符串、带图标/描述富选项、分组和分隔线。

## 导入

```tsx
import {Selector, SelectorOption} from '@astryxdesign/core';
```

## 基本用法

```tsx
// 简单字符串选项
<Selector
  label="Fruit"
  options={['Apple', 'Banana', 'Orange']}
  value={fruit}
  onChange={setFruit}
  placeholder="Select a fruit..."
/>

// 带图标和描述的富选项
<Selector
  label="Status"
  options={[
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
  options={[
    { type: 'section', title: 'Edit', options: [
      { value: 'cut', label: 'Cut' },
      { value: 'copy', label: 'Copy' },
    ]},
    { type: 'divider' },
    { type: 'section', title: 'View', options: [
      { value: 'zoom-in', label: 'Zoom In' },
      { value: 'zoom-out', label: 'Zoom Out' },
    ]},
  ]}
  value={action}
  onChange={setAction}
/>
```

## 自定义选项渲染

```tsx
<Selector
  label="User"
  options={users.map(u => ({value: u.id, label: u.name, icon: 'user'}))}
  value={selectedUser}
  onChange={setSelectedUser}
  renderOption={option => (
    <SelectorOption
      icon={option.icon}
      label={option.label}
      description={option.email}
    />
  )}
/>
```

## Props

### Selector

| Prop          | 类型                                                          | 默认值      | 描述                           |
| ------------- | ------------------------------------------------------------- | ----------- | ------------------------------------- |
| label         | string                                                        | 必填        | 标签文本（为无障碍始终渲染） |
| options       | Array<string \| OptionData \| Divider \| Section>             | 必填        | 要显示的选项                |
| value         | string                                                        | —           | 当前选中的值              |
| onChange      | (value: string) => void                                       | —           | 选择变更回调             |
| changeAction  | (value: string) => Promise<void>                              | —           | 变更时执行的异步操作                |
| placeholder   | string                                                        | 'Select...' | 未选中时的占位符     |
| size          | 'sm' \| 'md' \| 'lg'                                          | 'md'        | 触发器尺寸                          |
| isDisabled    | boolean                                                       | false       | 禁用状态                        |
| isLabelHidden | boolean                                                       | false       | 视觉上隐藏标签                   |
| description   | string                                                        | —           | 标签下方的帮助文本                 |
| isOptional    | boolean                                                       | false       | 显示可选指示器               |
| isRequired    | boolean                                                       | false       | 显示必填指示器               |
| status        | { type: 'warning' \| 'error' \| 'success', message?: string } | —           | 校验状态                     |
| renderOption  | (option: OptionData) => ReactNode                             | —           | 自定义选项渲染器                |

### SelectorOption（用于自定义渲染）

| Prop        | 类型      | 描述        |
| ----------- | --------- | ------------------ |
| icon        | IconType  | 前置图标       |
| label       | ReactNode | 主文本       |
| description | ReactNode | 次要文本     |
| children    | ReactNode | 附加内容 |

### 选项类型

```ts
type OptionData = {
  value: string;
  label?: string;
  disabled?: boolean;
  icon?: IconType;
};
type Divider = {type: 'divider'};
type Section = {type: 'section'; title?: string; options: OptionData[]};
```