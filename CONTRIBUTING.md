# 为 Astryx 做贡献

完整的贡献流程——我们接受什么、如何提议新组件、API 决策如何做出——请阅读 **[贡献 wiki](https://github.com/meta/astryx/wiki/Contributing)**。

关键页面：

- **[Pull Request 意图](docs/contributing/pull-requests.md)** — 选择一个主要意图、其证据标准，以及匹配的 PR 模板
- **[API 约定指南](docs/contributing/api-conventions.md)** — 实用的命名、组合、样式、提案和审查指导，链接到当前负责人记录
- **[设计约定](https://github.com/meta/astryx/wiki/Design-Conventions)** — 设计侧标准：token、间距、圆角、高度、字体、颜色、动效和状态表示
- **[规范协议](https://github.com/meta/astryx/wiki/Component-Specification-Protocol)** — 新组件的 9 阶段流程
- **[组件生命周期](https://github.com/meta/astryx/wiki/Component-Lifecycle)** — 组件如何从 lab → core，模板如何从隐藏 → 可见
- **[API 仲裁](https://github.com/meta/astryx/wiki/API-Arbitration)** — 我们如何解决 API 设计问题
- **[贡献模板](https://github.com/meta/astryx/wiki/Contributing-Templates)** — 构建模板/区块以及模板评分标准
- **[博客审查标准](https://github.com/meta/astryx/wiki/Blog-Review-Rubric#three-required-scored-criteria)** — 文档站博客文章如何审查
- **[借助 AI 贡献](https://github.com/meta/astryx/wiki/Contributing-with-AI-Assistants)** — 安全区域、规范协议以及与 AI 工具协作

本文件涵盖本地开发环境搭建。

---

## 前置条件

### Node.js

Node 版本存放在 `.nvmrc` 中（当前为 24.x 系列）。CI 通过 `node-version-file` 读取同一文件，因此本地和 CI 永远不会出现偏差。不要在其他地方声明版本。

**通过 nvm（推荐）：**

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.zshrc
nvm install   # 无参数——读取 .nvmrc
```

`fnm` 和 `mise` 也会读取 `.nvmrc`。`asdf` 不会；它的 `.tool-versions` 被 git 忽略，正是为了防止它成为竞争的真相来源。

**通过 nodejs.org：**
从 https://nodejs.org 下载并安装

### pnpm

Astryx 使用 [pnpm](https://pnpm.io/) 作为包管理器（在 `package.json` 的 `packageManager` 字段中声明）。你可以直接安装 pnpm：

```bash
# 通过 npm
npm install -g pnpm@11

# 通过 Homebrew（macOS）
brew install pnpm

# 通过独立安装器（无需 npm 或 Node.js）
curl -fsSL https://get.pnpm.io/install.sh | env PNPM_VERSION=11.10.0 sh -

# 通过 GitHub releases（单一二进制文件，无依赖）
# https://github.com/pnpm/pnpm/releases/latest
```

或使用 [Corepack](https://nodejs.org/api/corepack.html) 安装 Astryx 锁定的确切 pnpm 版本：

```bash
corepack enable
```

Corepack 随 Node.js 22 和 24 一起提供，但当前的 Node.js 25+ 版本不再捆绑它。如果 `corepack` 缺失且你想要自动锁定路径，请先手动安装 Corepack：

```bash
npm install -g corepack
corepack enable
```

验证安装：

```bash
node --version   # v24.x.x
pnpm --version   # 11.x.x
```

## 入门

```bash
# 克隆时不预先下载历史文件内容
git clone --filter=blob:none https://github.com/meta/astryx.git
cd astryx

# 安装依赖
pnpm install

# 启动 Storybook 进行组件开发
cd apps/storybook
pnpm dev
```

### 运行 Storybook

Storybook 将每个工作区包解析到其 `src/` 目录并自行编译，因此全新克隆无需先构建——`pnpm install` 然后 `pnpm dev` 就够了。

```bash
cd apps/storybook
pnpm dev
```

Storybook 将在 http://localhost:6006 打开，包含：

- **主题切换器** - 在基础 token 与 Neutral、Stone 和 Y2K 主题之间切换
- **模式切换器** - 在浅色和深色模式之间切换
- **组件故事** - 交互式组件示例

**如果你修改了 `@astryxdesign/core`：** 源码更改无需重新构建或重启即可更新。Storybook 和文档站还会监视 `packages/core/locales/*.json`，重新生成紧凑的运行时目录，并在语言文件编辑后重新加载导入的消息。

### 运行文档站

文档站（`apps/docsite/`）是一个 Next.js 应用，在 https://astryx.atmeta.com 渲染组件文档。本地运行：

```bash
# 仅首次——构建它依赖的工作区包
pnpm build

# 启动文档站（Next 开发服务器，默认 localhost:3000）
pnpm docsite
```

`pnpm docsite` 是 `pnpm -F @astryxdesign/docsite dev` 的薄别名，它在启动 Next 之前运行文档站的 `generate` 步骤（主题 CSS、注册表、playground 作用域）。

> **注意：** `pnpm docs` 与 `npm docs` 内置命令冲突，后者会尝试在浏览器中打开包的 npm 页面。请改用 `pnpm docsite`。

## 项目结构

```
astryx/
├── apps/
│   ├── storybook/      # 组件演练场 (localhost:6006)
│   ├── docsite/        # 文档站 (localhost:3000)
│   └── sandbox/        # 开发测试
│
├── packages/
│   ├── core/           # 核心组件 (Button, Input 等)
│   ├── cli/            # CLI 工具 (astryx)
│   ├── lab/            # 实验性组件（尚未稳定）
│   └── themes/         # 主题预设 (neutral, stone, y2k 等)
│
└── internal/           # 内部工具（不发布）
    └── test-utils/     # 共享测试辅助
```

## 开发工作流

### 常用命令

| 命令              | 描述                                  |
| ----------------- | -------------------------------------------- |
| `pnpm install`    | 安装所有依赖                     |
| `pnpm dev`        | 启动 Storybook（`pnpm storybook` 的别名） |
| `pnpm build`      | 构建所有包                           |
| `pnpm test`       | 运行所有测试                                |
| `pnpm test:watch` | 以监视模式运行测试                      |
| `pnpm typecheck`  | 对每个包进行类型检查（在 `pnpm build` 之后） |
| `pnpm storybook`  | 在 localhost:6006 启动 Storybook            |
| `pnpm lint`       | 对所有包进行 lint                            |

## 添加新组件

组件使用**同目录测试**——测试文件与组件放在一起。

### 1. 创建组件目录

```bash
mkdir -p packages/core/src/MyComponent
```

### 2. 创建组件文件

```
packages/core/src/MyComponent/
├── MyComponent.tsx        # 组件实现
├── MyComponent.test.tsx   # 单元测试（同目录）
├── MyComponent.doc.mjs    # 组件文档（props、特性、示例）
└── index.ts               # 公共导出
```

故事**不**同目录——它们位于 Storybook 应用中：

```
apps/storybook/stories/MyComponent.stories.tsx
```

### 3. 组件模板

````tsx
// MyComponent.tsx
import type {HTMLAttributes, ReactNode, Ref} from 'react';

export interface MyComponentProps extends HTMLAttributes<HTMLDivElement> {
  /** Ref forwarded to the root element */
  ref?: Ref<HTMLDivElement>;
  /** Description for AI-assisted development */
  children: ReactNode;
}

/**
 * Brief description of the component.
 *
 * @example
 * ```
 * <MyComponent>Hello</MyComponent>
 * ```
 */
export function MyComponent({children, ref, ...props}: MyComponentProps) {
  return (
    <div ref={ref} {...props}>
      {children}
    </div>
  );
}

MyComponent.displayName = 'MyComponent';
````

### 4. 测试模板

```tsx
// MyComponent.test.tsx
import {describe, it, expect} from 'vitest';
import {render, screen} from '@testing-library/react';
import {MyComponent} from './MyComponent';

describe('MyComponent', () => {
  it('renders children', () => {
    render(<MyComponent>Hello</MyComponent>);
    expect(screen.getByText('Hello')).toBeInTheDocument();
  });
});
```

### 5. 故事模板

故事位于 Storybook 应用中，而非组件旁边——`apps/storybook/.storybook/main.ts` 通过 `'../stories/**/*.stories.@(js|jsx|mjs|ts|tsx)'` 发现它们，因此 `packages/` 下任何位置的故事都不会被拾取。通过组件的已发布入口点导入它，并将其标题置于 `Core/` 下（`@astryxdesign/lab` 则置于 `Lab/` 下）。

```tsx
// apps/storybook/stories/MyComponent.stories.tsx
import type {Meta, StoryObj} from '@storybook/react';
import {MyComponent} from '@astryxdesign/core/MyComponent';

const meta = {
  title: 'Core/MyComponent',
  component: MyComponent,
  tags: ['autodocs'],
} satisfies Meta<typeof MyComponent>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    children: 'Hello World',
  },
};
```

### 6. 从包中导出

```ts
// packages/core/src/index.ts
export * from './MyComponent';
```

> **注意：** 不要手动编辑 `packages/core/package.json` 中的 `"exports"` 字段。它由 `scripts/sync-exports.js` 从 `src/` 目录自动生成，并在更改落地 `main` 时自动提交。如果你需要验证组件会被包含，运行 `pnpm sync:exports:check`。

### 面向用户的字符串和文本方向

组件使用 `useTranslator()` 读取面向用户的字符串，而非硬编码：

```tsx
import {useTranslator} from '@astryxdesign/core/i18n';

function SaveButton() {
  const t = useTranslator();
  return <button>{t('@astryx.actions.save')}</button>;
}
```

Astryx 自身的字符串位于 `packages/core/locales/en.json`。`@astryx/no-hardcoded-i18n-string` ESLint 规则会拒绝硬编码字符串。对于默认值为翻译字符串的 prop，为 prop 起别名并在函数体中解析：解构 `label: labelFromProps`，然后 `const label = labelFromProps ?? t('@astryx.<component>.<key>')`。

响应文本方向的组件从 DOM 解析方向，而非在渲染时通过 JavaScript 读取。按以下顺序使用能起作用的最轻量工具：

1. **CSS 逻辑属性。** 使用 `insetInlineStart`、`paddingInlineEnd`、`marginInline` 等，而非物理的 `left`/`right`。大多数镜像需求仅此即可：浏览器会根据环境 `dir` 翻转它。`@astryx/no-physical-properties` ESLint 规则强制执行此点。
2. **方向性图标：用 CSS 镜像，而非交换名称。** 渲染一个固定字形，并将其包裹在共享的 `rtlStyles.mirror` 中（一个仅在 `[dir="rtl"]` 下应用的 `scaleX(-1)`）。它通过级联从祖先 `dir` 翻转，因此在服务器上也能工作，无水合闪烁。不要在 JavaScript 中选择 `chevronLeft` 或 `chevronRight`。Pagination、Calendar 和 Carousel 就是这样处理它们的箭头图标的。

   ```tsx
   import * as stylex from '@stylexjs/stylex';
   import {rtlStyles} from '@astryxdesign/core';

   function NextButton() {
     // One glyph; CSS flips it under RTL. No direction read.
     return (
       <span {...stylex.props(rtlStyles.mirror)}>
         <Icon icon="chevronRight" />
       </span>
     );
   }
   ```

3. **行为：在事件发生时惰性读取 DOM。** 对于 CSS 无法表达的（方向键映射、拖拽和滚动计算），在交互时用 `isRtlElement(el)`（一个 `getComputedStyle().direction` 检查）读取方向，绝不在渲染期间读取。焦点 Hook（`useListFocus`、`useGridFocus`、`useTreeFocus`）已从其容器检测方向，因此方向键会自动翻转；不要给它们传方向标志。
4. **`useDirection()` 上下文，作为最后手段。** 仅当你需要在渲染期间获取方向值且以上都不适用时使用。它是 SSR 安全的，在 provider 外返回 `'ltr'`（类似 `useTranslator` 的静默回退），但它是唯一在 provider 方向与 `<html dir>` 不一致时可能在水合时不匹配的路径。没有 Astryx 组件在渲染时从上下文读取方向。

### 添加语义图标

仅为整个系统共享的字形添加语义图标名称：

1. 将名称添加到 `packages/core/src/Icon/globalIconRegistry.tsx` 中的 `IconName` 类型。
2. 将默认 SVG 添加到 `packages/core/src/Icon/defaultIcons.tsx`。
3. 在 `packages/cli/assets/docs/icons.doc.mjs` 的 Available Names 表中添加一行。

属于单个组件的字形使用命名空间键（`numberInput:stepperDown`），这样共享的 `IconName` 列表不会增长。在 `defaultIcons` 中以相同键提供其回退，以便在没有主题时字形仍能渲染。

## 无障碍检查清单

每个新组件——以及对交互式组件的任何更改——必须在审查前通过 **[无障碍检查清单](https://github.com/meta/astryx/wiki/Accessibility-Checklist)**（wiki）。检查清单位于 wiki 上，以便无障碍专家无需代码 PR 即可完善它；审查者会以此阻塞（它是 [组件审计标准](https://github.com/meta/astryx/wiki/Component-Audit-Rubric) 上的 `A1`–`A16`），并且它是 lab → core 晋升的硬性要求（见 `packages/lab/README.md`）。

此处值得重申两条仓库侧规则：

- 组合共享原语——`VisuallyHidden`、`useAnnounce`、`useFocusTrap` 以及焦点 Hook（`useListFocus`、`useGridFocus`、`useTreeFocus`）——而非手工编写等价物。它们实现了 WAI-ARIA APG 模式并经过一次测试；对其中一个的定制重新实现会被审查拒绝。
- CI 是执行层，而非检查清单的替代品：`ci.yml` 中的 `pr-a11y` 作业在合并到 `main` 后和每次发布前（以及在标记为 `ci:run-a11y` 的 Pull Request 上）运行 axe 审计，覆盖组件表面的完整范围，而 `useAnnounce` lint 规则拒绝手工接线的 `aria-live` 区域。axe 只捕获静态的 DOM 级问题——键盘行为、焦点管理和播报时机正是检查清单和组件单元测试所覆盖的内容。

## 开发 `astryx` CLI

CLI（`packages/cli/`）是分层的，以便行为、呈现和契约保持可分离：

- **`clients/cli/`** — Commander 程序和每命令处理器。处理器是_薄包装_：解析标志 → 调用匹配的 `api/` 函数 → 渲染（通过 `jsonOut` 输出 JSON，或通过 `clients/cli/formatters/` 中的格式化工具包输出文本）。
- **`api/`** — 编程 API（`@astryxdesign/cli/api`）。每个命令映射到 `api/<name>/`，其函数返回类型化的 `{ type, data }` 信封。这是行为的真相来源，因此 `astryx --json` 和导入的函数返回相同的数据。
- **`authoring/`** — 纯数据契约（`@astryxdesign/cli/authoring`）：你编写对象所依据的 TypeScript 类型（config、integration、codemod 和 doc-types）以及 CLI 在加载边界运行的密封 zod 解析器。
- **`foundation/`** — 底层：所有上层构建所依赖的横切基础设施——`{ type, data }` JSON 契约、稳定的 `ERROR_CODES`、发现（组件、模板）、集成贡献验证器和路径安全。它从不导入 `api/` 或 `clients/`；如果 foundation 需要某物，那物就属于 foundation。

### CLI 自我文档化

每个 CLI 表面在其描述对象旁边都有一个同目录的、类型化的 `.doc.mjs`，并用来自 `@astryxdesign/cli/authoring` 的 `@type` 注解：

| 表面                                                                                 | Doc-type                                             | 位于                                              |
| --------------------------------------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------- |
| 一个 API 函数（一个 Hook 或一个 `api/` 导出）                                            | `FunctionDoc`                                        | `api/<name>/<fn>.doc.mjs`                                  |
| 一个 CLI 命令                                                                           | `CommandDoc`（通过 `fn` 引用其 `FunctionDoc`） | `clients/cli/commands/<name>.doc.mjs`                      |
| 一个编写对象（config、integration、codemod、doc-types、响应