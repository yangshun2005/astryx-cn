

> **[中文化] astryx**
>
> 此项目是 [meta/astryx](https://github.com/meta/astryx) 的中文翻译版本。
> - 原项目 Stars: 14331
> - 主语言: TypeScript
> - 许可证: MIT
> - 翻译日期: 2026-11-08
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

Astryx 是 Meta 开源的一套设计系统（design system），在 Meta 内部发展了八年，据称支撑了 13,000+ 应用。它提供 150+ 个可访问（accessible）组件、品牌级主题定制、暗色模式、现成模板和一个 CLI，打包成一个整体系统。使用方式是导入预构建的 CSS 加类型化的 React 组件，不需要构建插件，也不需要额外采用某个样式库。项目目前处于 **Beta** 阶段。

### 快速上手（最短路径）

1. 确认你的项目使用 **React 19** 或更高版本（`react` 和 `react-dom` 是 `@astryxdesign/core` 的 peer dependencies）。
2. 安装核心包、一个主题和 peer 依赖（以 npm 为例，原文也给了 pnpm / yarn 版本）：
   ```bash
   npm install @astryxdesign/core @astryxdesign/theme-neutral @stylexjs/stylex
   npm install -D @astryxdesign/cli
   ```
3. （可选）全局安装 CLI，让 `astryx` 命令在任意目录可用：
   ```bash
   npm i -g @astryxdesign/cli
   ```
4. 在 `package.json` 里加一个脚本，避免 AI 助手或新同事直接调用 CLI 时出现路径错误：
   ```json
   "scripts": {
     "astryx": "node node_modules/@astryxdesign/cli/clients/cli/bin/astryx.mjs"
   }
   ```
   之后用 `npm run astryx -- component --list` 调用。
5. 完成最简单的接入：几个 CSS 导入加一个 theme provider，不需要构建插件、PostCSS 或 Babel 配置。完整指南（含 Next.js、Tailwind、Vite、CDN）见 `packages/core/README.md#quick-start`——**原文未在本 README 中给出具体代码**。

### 环境要求

- **操作系统**：原文未说明。
- **硬件**：原文未说明。
- **依赖**：React 19 或更高版本（`react`、`react-dom` 为 peer dependencies）；需安装 `@stylexjs/stylex`；CLI 需 Node.js（原文未给出具体 Node 版本）。
- **版本要求**：项目自身处于 Beta；`@astryxdesign/lab` 不发布到 npm；`@astryxdesign/vega` 和 `@astryxdesign/charts` 仅在 `@canary` dist-tag 下发布，暂无稳定版。
- **国内网络提示**：原文未提及镜像源，但 npm 包安装在国内可能需要更换镜像源或加速下载。

### 常见坑与注意事项

- **必须用 React 19+**：这是硬性 peer dependency，低版本 React 项目无法直接使用。
- **CLI 调用路径问题**：原文明确提醒，AI 助手或新开发者直接调用 CLI 可能出现路径错误，建议通过 `package.json` 脚本调用。
- **部分包没有稳定版**：`@astryxdesign/vega`、`@astryxdesign/charts` 只有 `@canary` 标签；`@astryxdesign/lab` 根本不发布到 npm，不要指望从 npm 安装。
- **Beta 阶段**：项目自述「Currently in Beta」，API 可能变动，生产使用需自行评估。

---



---

<!-- SYNC CONTRACT: 架构变更需要同步更新文档。 -->

<div align="center">

<img src="https://lookaside.facebook.com/assets/astryx/Astryx-Banner.png" alt="Astryx" width="100%" />

# Astryx

一个完全可定制的开源设计系统，为当下的构建方式而生：<br/> 由人以及与之协作的智能体共同构建。

**目前处于 Beta 阶段** · 基于 [React 19+](https://react.dev) 和 [StyleX](https://stylexjs.com) 构建

[![npm version](https://img.shields.io/npm/v/@astryxdesign/core?label=npm&color=cb3837&logo=npm)](https://www.npmjs.com/package/@astryxdesign/core)
[![license MIT](https://img.shields.io/npm/l/@astryxdesign/core?color=blue)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/meta/astryx/wiki/Contributing)

[![Docs](https://img.shields.io/badge/Docs-astryx.atmeta.com-6741d9?logo=readthedocs&logoColor=white)](https://astryx.atmeta.com)
[![Storybook](https://img.shields.io/badge/Storybook-live-ff4785?logo=storybook&logoColor=white)](https://astryx.atmeta.com/storybook/)
[![Sandbox](https://img.shields.io/badge/Sandbox-playground-A9792B)](https://astryx.atmeta.com/sandbox/)

**[文档](https://astryx.atmeta.com)** · **[Storybook](https://astryx.atmeta.com/storybook/)** · **[沙盒](https://astryx.atmeta.com/sandbox/)** · **[贡献指南](https://github.com/meta/astryx/wiki/Contributing)** · **[Discord](https://discord.com/invite/XnsUcFykEP)**

</div>

## 概述

Astryx 是一个开源设计系统，过去八年在 Meta 内部成长，成为公司内使用最广、规模最大的设计系统——支撑着 13,000+ 个应用，由每天依赖它的工程师、设计师和产品团队共同塑造。

它以一套连贯的系统提供 150+ 个无障碍组件、品牌级主题、深色模式、开箱即用的模板和 CLI。你只需导入预构建的 CSS 并使用带类型的 React 组件——无需构建插件，无需引入样式库——人和 AI 助手使用同一套工具构建。

**Astryx 的不同之处：**

- **开放的内部实现。** 组件可在任意层级组合，不受封闭的顶层 API 限制。你需要的构建块直接导出；需要更深入时，swizzle 可将组件完整源码弹出到你的项目中，由你掌控。
- **无样式锁定。** Astryx 用 StyleX 编写样式，但这对使用者不可见。可用 `className` 配合 Tailwind、CSS modules 或纯 CSS 覆盖——用你项目已有的方式即可。
- **无需包装即可定制。** 主题是一组 CSS 自定义属性覆盖，设计师无需 Fork 或包装组件源码，就能让 Astryx 带上鲜明的个人风格。
- **为人与智能体而构建。** API、文档和 CLI 协同设计，让人和 AI 助手基于同一份参考、以相同方式构建。

## 快速开始

Astryx 需要 **React 19** 或更高版本（`react` 和 `react-dom` 是 `@astryxdesign/core` 的 peer 依赖）。

安装 Astryx、一个主题及其 peer 依赖：

```bash
# npm
npm install @astryxdesign/core @astryxdesign/theme-neutral @stylexjs/stylex
npm install -D @astryxdesign/cli

# pnpm
pnpm add @astryxdesign/core @astryxdesign/theme-neutral @stylexjs/stylex
pnpm add -D @astryxdesign/cli

# yarn
yarn add @astryxdesign/core @astryxdesign/theme-neutral @stylexjs/stylex
yarn add -D @astryxdesign/cli
```

你也可以在电脑上全局安装一次 CLI，这样 `astryx` 在任何文件夹都能用，每个编码智能体也都认识 Astryx；在项目中，它仍会运行该项目自己的版本：

```bash
npm i -g @astryxdesign/cli
```

项目安装、全局安装，还是两者都要？`astryx docs global/project-global-or-both` 会比较它们（推荐两者都要），`astryx docs global` 解释全局模式如何工作。

最简单的配置是几个 CSS 导入加一个主题 Provider——无需构建插件，无需 PostCSS 或 Babel 配置。完整指南（Next.js、Tailwind、Vite 和 CDN）见 **[`@astryxdesign/core` README](packages/core/README.md#quick-start)**。

为可靠地使用 CLI，在 `package.json` 中添加一个脚本：

```json
"scripts": {
  "astryx": "node node_modules/@astryxdesign/cli/clients/cli/bin/astryx.mjs"
}
```

然后以 `npm run astryx -- component --list` 使用。这样可避免 AI 助手或新开发者直接调用 CLI 时的路径错误。

## 包

| 包                                         | 描述                                                                                                 | README                             |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------- | ---------------------------------- |
| [`@astryxdesign/core`](packages/core)      | 组件、主题系统和工具                                                                                 | [README](packages/core/README.md)  |
| [`@astryxdesign/cli`](packages/cli)        | CLI 工具：组件文档、模板、脚手架、主题和 codemod                                                      | [README](packages/cli/README.md)   |
| [`@astryxdesign/build`](packages/build)    | StyleX 源码构建的构建插件                                                                            | [README](packages/build/README.md) |
| [`@astryxdesign/theme-*`](packages/themes) | 七个现成、完全可定制的主题（neutral、butter、chocolate、matcha、stone、gothic、y2k）                  | [README](packages/themes)          |

> `@astryxdesign/lab`（实验性组件）供 Storybook 和沙盒内部使用，未发布到 npm。`@astryxdesign/vega`（Vega/Vega-Lite 图表封装）和 `@astryxdesign/charts`（图表组件）仅以 `@canary` dist-tag 发布到 npm——尚无稳定版本。

## 原则

这些是 Astryx 对使用者的承诺。

- **引导而非强制。** 组件提供能力，而不是与你对抗的护栏。设计观点体现在文档和示例中——你传什么值，组件就渲染什么。
- **强大且有文档的约定。** 每个组件遵循相同的命名、Props 和组合规则，且都有详尽文档——学会几个之后，其余的都似曾相识，人和 AI 都能预测陌生组件的行为。
- **人与 AI 共用一套系统。** API、约定、文档和 CLI 协同设计，让人和 AI 助手以相同方式构建。每一次让 Astryx 对 AI 更友好的改动，也让人用起来更顺手。
- **以度量赢得。** 我们测试约定而非断言约定，对结果保持开放，遇到新情况证明其错误时重新审视。

## 架构

### 基础

视觉统一且无障碍的界面构建块：排版、颜色、布局和无障碍。

### 组件

150+ 个可复用 UI 构建块的库，完整支持 TypeScript。

### 模式

经过实战检验的设计方案，覆盖常见交互和工作流：表格页、详情页布局、表单向导、导航模式、数据录入流程。

## 项目结构

| 目录        | 用途                                                     |
| ----------- | -------------------------------------------------------- |
| `apps/`     | 示例应用、文档站点和 Storybook                           |
| `packages/` | 已发布的包：core、cli、build、themes                     |
| `internal/` | 内部工具：测试工具、eslint 插件、vibe 测试               |

## 贡献

欢迎贡献！完整指南见 **[CONTRIBUTING.md](CONTRIBUTING.md)**。

有问题，或需要 PR 方面的帮助？来
**[Discord](https://discord.com/invite/XnsUcFykEP)** 找我们。

我们的约定和评审标准位于
[贡献 wiki](https://github.com/meta/astryx/wiki/Contributing)——
包括 [API 约定](https://github.com/meta/astryx/wiki/API-Conventions)、
[设计约定](https://github.com/meta/astryx/wiki/Design-Conventions)、
[组件生命周期](https://github.com/meta/astryx/wiki/Component-Lifecycle)、
[贡献模板](https://github.com/meta/astryx/wiki/Contributing-Templates)
和 [博客评审标准](https://github.com/meta/astryx/wiki/Blog-Review-Rubric#three-required-scored-criteria)。提交 PR 前请阅读相关标准。

贡献者快速开始：本仓库使用 **Node 22+ 的活跃 LTS 版本**
和 **pnpm 11**。直接安装 pnpm，或启用一次
[Corepack](https://nodejs.org/api/corepack.html)，让锁定的 pnpm 版本
自动安装：

```bash
corepack enable
pnpm install
```

如果缺少 `corepack`，直接安装 pnpm 或手动安装 Corepack；
参见 [CONTRIBUTING.md](CONTRIBUTING.md#troubleshooting) 中的故障排除说明。

Theme Showcase DOM 测试使用根 `ui` 项目进行共享的 DOM 和
StyleX 测试配置。在仓库根目录运行
`pnpm exec vitest run --project ui apps/docsite/src/__tests__/theme-showcase/`。

## 许可证

MIT