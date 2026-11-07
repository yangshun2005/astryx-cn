# Vibe Test 任务分类法

LLM 根据提示词生成 UI 时，其输出可拆分为若干独立任务。本分类法用于分析 token 成本并识别优化机会。

## 概要

### 输入 Token 来源

| 来源 | 说明 |
| ------------------- | ------------------------------------------- |
| **AGENTS.md** | 组件目录和 Astryx 指南 |
| **设计文档** | principles.md、tokens.md——样式模式 |
| **组件文档** | Button.md、TextInput.md 等 |
| **提示词开销** | 任务指令和角色设定 |

### 输出 Token 任务

| 任务 | 说明 | 典型占比 |
| ------------------------ | ------------------------------------------- | --------- |
| **组件路由** | Astryx 组件的 import 语句 | 3-10% |
| **组件配置** | Astryx 组件的 Props 和属性 | 7-30% |
| **补充样式** | 用于布局/间距缺口的 StyleX 块 | 10-27% |
| **内容编写** | HTML 结构、JSX 元素、文案 | 15-27% |
| **业务逻辑** | useState、处理函数、API 调用、条件判断 | 5-51% |
| **模板代码** | 类型定义、import、export | 5-10% |

## 任务（详细）

### 1. 组件路由（约占输出 token 的 3-10%）

**Astryx 组件的 import 语句**

```tsx
import {TextInput} from '@astryxdesign/core/TextInput';
import {Button} from '@astryxdesign/core/Button';
```

- 要求：理解组件目录，将意图匹配到组件
- 输入成本：阅读组件文档、AGENTS.md 索引
- 失败模式：选错组件、遗漏组件、幻觉组件

### 2. 组件配置（约占输出 token 的 7-30%）

**Astryx 组件的 Props 和属性**

```tsx
<TextInput
  label="Name"
  value={name}
  onChange={setName}
  isRequired
  placeholder="Enter your name"
/>
<Button variant="primary" disabled={!isFormValid}>
```

- 要求：理解组件 API、有效的 prop 值
- 输入成本：阅读具体组件文档（Button.md、TextInput.md 等）
- 失败模式：幻觉 prop、prop 类型错误、缺少必填 prop

### 3. 补充样式（约占输出 token 的 10-27%）

**用于布局/间距缺口的 StyleX 块**

```tsx
const styles = stylex.create({
  form: {
    display: 'flex',
    flexDirection: 'column',
    gap: 'var(--spacing-4)',
    padding: 'var(--spacing-6)',
    backgroundColor: 'var(--color-background-surface)',
  },
});
```

- 要求：理解设计 token、StyleX 模式
- 输入成本：阅读 tokens.md、principles.md
- 失败模式：硬编码值（反模式）、幻觉 CSS 变量、冗余 CSS

### 4. 内容编写（约占输出 token 的 15-27%）

**HTML 结构、JSX 元素、文案**

```tsx
return (
  <form onSubmit={handleSubmit} {...stylex.props(styles.form)}>
    {/* structure */}
    <div>Main content goes here</div>
  </form>
);
```

- 要求：理解 JSX、语义化 HTML
- 输入成本：极少（通用知识）
- 失败模式：可访问性问题、结构不佳

### 5. 业务逻辑（约占输出 token 的 5-51%）

**useState、处理函数、API 调用、条件判断**

```tsx
const [name, setName] = useState('');
const isFormValid = name.trim() !== '' && email.trim() !== '';

const handleSubmit = (e: React.FormEvent) => {
  e.preventDefault();
  if (isFormValid) {
    console.log('Form submitted:', {name, email});
  }
};
```

- 要求：理解 React 模式、状态管理
- 输入成本：极少（通用知识）
- 失败模式：逻辑 bug、遗漏边界情况

### 6. 模板代码（约占输出 token 的 5-10%）

**类型定义、import、export**

```tsx
import { useState } from 'react';
import * as stylex from '@stylexjs/stylex';

export function RequiredFieldsForm() {
```

- 要求：理解模块系统、TypeScript
- 输入成本：极少（通用知识）
- 失败模式：缺少 import、路径错误

## 按提示词类别的 Token 分布

| 类别 | 路由 | 配置 | 样式 | 内容 | 逻辑 | 模板代码 |
| ------------------------ | ------- | ------ | ------- | ------- | ----- | ----------- |
| **状态驱动** | 8% | 29% | 20% | 15% | 19% | 9% |
| **集成导向** | 2% | 7% | 10% | 24% | 51% | 5% |
| **页面搭建** | 6% | 26% | 27% | 27% | 5% | 10% |

### 关键洞察

1. **集成导向提示词逻辑繁重**（51% 业务逻辑）
   - 文件上传、API 调用、复杂状态
   - 组件配置极少（7%）

2. **页面搭建提示词较为均衡**
   - 样式、内容和配置比例相当
   - 业务逻辑极少（5%）

3. **状态驱动提示词侧重配置**
   - 表单校验需要组件 prop（29%）
   - 状态相关业务逻辑适中（19%）

## 各任务的输入 Token 成本

| 任务 | 所需文档 | 估计输入 Token |
| -------------------- | ------------------------ | ----------------- |
| 组件路由 | AGENTS.md 索引 | ~175 |
| 组件配置 | 组件文档（1-3 份） | 每份 300-2000 |
| 补充样式 | tokens.md、principles.md | ~1200 |
| 内容编写 | 无（通用知识） | 0 |
| 业务逻辑 | 无（通用知识） | 0 |
| 模板代码 | 无（通用知识） | 0 |

## 完整 Token 用量明细

aggregate 命令显示完整的 token 用量明细：

```
📊 Token Usage Breakdown:
  ┌─────────────────────────────────────────────────┐
  │ INPUT TOKENS (estimated from doc reading)       │
  ├─────────────────────────────────────────────────┤
  │   AGENTS.md:           173 tokens               │
  │   Design docs:         1200 tokens              │  (principles.md, tokens.md)
  │   Component docs:       800 tokens              │  (Button.md, TextInput.md, etc.)
  │   Prompt overhead:      375 tokens              │  (task instructions)
  │   ─────────────────────────────                 │
  │   Input subtotal:      2548 tokens              │
  ├─────────────────────────────────────────────────┤
  │ OUTPUT TOKENS (from job breakdown)              │
  ├─────────────────────────────────────────────────┤
  │   Component routing:       4%  (~48 tokens)     │
  │   Component config:       16%  (~195 tokens)    │
  │   Supplemental styling:   16%  (~188 tokens)    │
  │   Content authoring:      22%  (~263 tokens)    │
  │   Business logic:         34%  (~408 tokens)    │
  │   Boilerplate:             7%  (~86 tokens)     │
  │   ─────────────────────────────                 │
  │   Output subtotal:       1188 tokens            │
  ├─────────────────────────────────────────────────┤
  │ TOTAL:                   3736 tokens            │
  └─────────────────────────────────────────────────┘
```

### 输入 Token 类别

- **AGENTS.md**：始终读取（~173 token）——组件目录和 Astryx 指南
- **设计文档**：tokens.md（~897）和 principles.md（~283）——样式模式
- **组件文档**：因所用组件而异（每份 300-2000 token）
- **提示词开销**：任务指令和角色设定（~375 token）

### 输出 Token 类别

通过解析生成代码分析得出：

- import 语句 → 组件路由
- 带 props 的 Astryx 组件 JSX → 组件配置
- `stylex.create()` 块 → 补充样式
- HTML 元素和结构 → 内容编写
- 状态 Hook、处理函数、逻辑 → 业务逻辑
- 类型定义、export → 模板代码

## 计时

计时按测试和类别分别跟踪：

```
⏱️  Performance:
  Total time: 200.9s (avg 67.0s per test)

By Category:
  integration-oriented      100% (1/1) ⏱️70.1s
  page-setup                100% (1/1) ⏱️65.8s
  state-driven              100% (1/1) ⏱️65.0s
```

计时根据文件时间戳推断（任务创建 → 结果写入）。

## 优化机会

### 减少组件配置 Token

- 组件提供更好的默认值 = 所需 prop 更少
- 复合组件 = 配置更少

### 减少补充样式 Token

- 更多布局原语 = 自定义 CSS 更少
- 更好的 gap/spacing prop = StyleX 更少

### 减少业务逻辑 Token

- 提供 Hook（useForm、useUpload）= 自定义逻辑更少
- 内置模式 = 模板逻辑更少

## 衡量任务质量

每个任务可独立评估：

| 任务 | 成功标准 | 逃生舱类型 |
| -------------------- | ------------------------------ | -------------------------------------- |
| 组件路由 | 选对组件 | `wrong_component`、`hallucination` |
| 组件配置 | prop 有效、无幻觉 | `hallucination` |
| 补充样式 | 使用 token 而非硬编码 | `hardcoded_color`、`hardcoded_spacing` |
| 内容编写 | 语义化 HTML、可访问 | （未跟踪） |
| 业务逻辑 | 正确运行 | （未跟踪） |
| 模板代码 | import 正确 | （未跟踪） |