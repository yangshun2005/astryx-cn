## 策略：访客自包含式接入

该应用已有自己的全局样式表、CSS reset 和 token 词汇。当设计系统必须只存在于某个新增或迁移的区域、且不影响页面其余部分时，采用此策略。

这是例外策略，不是既有应用的默认做法。此类应用的常规目标是宿主对齐式接入——宿主保持对排版和颜色的权威，设计系统通过受支持的公共 API 向其主题化。自包含适用于更窄的场景：工作硬性要求不动页面其余部分，即应用内某个嵌套或子作用域区域，而其全局归属无法迁移。

这也是目前没有完全受支持路径的策略。将设计系统限定到子树意味着要抑制下文所述的根属性同步，而目前没有公共 API 可做到；渲染在子树之外的 portal 内容会落在所有作用域根之外，从而丢失 token。这两者都是产品缺口，而非本文档能妥善做出的选择；遵循此策略会有意触发它们——它产出的既是一份配方，也是关于这些缺口的证据。

### 宿主保留全局归属

- **不要**导入 `@astryxdesign/core/reset.css`。宿主已拥有 preflight 和元素默认值；第二个 reset 会改变宿主元素。
- **不要**导入 `@astryxdesign/core/tailwind-theme.css`，也不要修改宿主现有的 token 定义。
- 保持 `@import 'tailwindcss';` 原位不动。在其上方声明层顺序，使设计系统的层位于宿主的 base 与宿主的 components 之间：

  ```css
  @layer reset, theme, base, astryx-base, astryx-theme, components, utilities;
  ```

- 在**产出的** CSS 中验证层顺序，而非源码顺序。

### 安装包——批准构建脚本

`@astryxdesign/core` 附带 `postinstall` 脚本，而 pnpm 11 在该依赖被批准前不会运行其安装脚本。它会直接让安装失败，而非静默跳过：

```
[ERR_PNPM_IGNORED_BUILDS] Ignored build scripts: @astryxdesign/core@0.5.2
Run "pnpm approve-builds" to pick which dependencies should be allowed to run scripts.
```

按错误提示操作：

```bash
pnpm install          # fails with ERR_PNPM_IGNORED_BUILDS
pnpm approve-builds   # --all to accept every pending package without prompting
```

这会把批准写入 **`pnpm-workspace.yaml`**，形式为 `allowBuilds` 映射：

```yaml
allowBuilds:
  '@astryxdesign/core': true
```

在手动编写任何内容之前，有两点值得了解：

- **pnpm 11 只从 `pnpm-workspace.yaml` 读取 `allowBuilds`。** 同一键放在 `package.json` 的 `pnpm` 字段下会被忽略，安装会像之前一样失败——这种失败令人困惑，因为设置就在你编辑的文件里。若手动编写而非运行 `approve-builds`，请写在 `pnpm-workspace.yaml` 中；项目没有该文件就创建它。
- **`ignoredBuiltDependencies` 和 `onlyBuiltDependencies` 不是这里的解法。** 在 pnpm 11 中，这两个键无论放在哪个文件都无法消除此错误。请使用 `approve-builds`，或它写入的 `allowBuilds` 映射。

将包设为 `false` 也能满足 pnpm——它记录的是不构建的决定——但 `@astryxdesign/core` 期望其 `postinstall` 运行，因此应批准而非抑制。

优先使用 `pnpm-workspace.yaml` 而非 `package.json` 还有另一个原因：它是 pnpm 自己的文件，npm 和 yarn 会忽略它，因此批准 pnpm 构建不会在这些工具读取的 manifest 中留下 pnpm 专属配置。

### 仅在访客子树周围挂载 Provider

只包裹新增或迁移的区域：

```tsx
<Theme theme={neutralTheme} mode={hostMode}>
  <NewRegion />
</Theme>
```

不要包裹应用根节点，也不要包裹任何既有宿主 UI。`mode` 跟随宿主自身的明暗状态；宿主保留其模式控制。

### 抑制根属性同步——此事的真实状况

上方没有父 Provider 的 Provider 是**根** Provider，根 Provider 会把 `data-theme` 和 `data-astryx-theme` 同步到 document 元素上。主题 CSS 以 `@scope ([data-astryx-theme="<name>"]) to (...)` 形式产出，因此 document 元素上的该属性会让整个文档成为作用域根——这恰好破坏了自包含。

**没有受支持的公共 API 可禁用该同步。** Provider 的公共 props 是 `theme`、`mode` 和 `children`；它们都无法退出，也没有嵌套模式或作用域根的逃生通道。不要臆造 prop 或深入内部实现。

目前唯一可用的机制是在挂载后从 document 元素移除该属性，并用 `MutationObserver` 保持移除：

```tsx
useEffect(() => {
  const root = document.documentElement;
  const strip = () => root.removeAttribute('data-astryx-theme');
  strip();
  const observer = new MutationObserver(strip);
  observer.observe(root, {
    attributes: true,
    attributeFilter: ['data-astryx-theme'],
  });
  return () => observer.disconnect();
}, []);
```

在代码注释中明确说明：这是对缺失能力的变通，而非受支持的配置。若宿主依赖 `data-theme` 实现 `color-scheme`，则不要动它；只移除主题名属性。

### Portal 的取舍——测试并记录

document 元素上的 `data-astryx-theme` 正是让 `@scope` 主题 CSS 能触及渲染在 Provider DOM 子树**之外**的内容：portal 覆盖层、对话框、弹出层、工具提示和回退 toast 视口。因此移除它并非没有代价：

- Provider 子树内的内容保持主题化；
- 渲染到 `document.body` 的 portal 设计系统内容会落在所有作用域根之外，丢失主题 token。

你必须直接验证：在两种模式下从访客子树内打开一个 portal 设计系统覆盖层，观察结果，并在变通方案旁的注释中记录发生了什么。若该区域需要 portal 覆盖层，请明确说明——对该区域而言，自包含与主题化 portal 目前无法兼得，选择权属于应用。

不要通过以其他名称重新添加全局属性、把主题变量复制到 `body`，或添加 `!important` 覆盖来解决此问题。