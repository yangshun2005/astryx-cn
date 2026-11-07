## 具体编辑

按顺序执行：

1. 运行 `pnpm add @astryxdesign/core @astryxdesign/theme-neutral`。
2. 在 `src/index.css` 中，将 layer 声明插入到现有 `@import 'tailwindcss';` **上方**，然后在 Tailwind 导入之后紧接着添加三个 Astryx 样式表。保持宿主样式表的其余部分不变。
3. 在 `src/main.tsx` 中，用 `<Theme theme={neutralTheme} mode="light">` 包裹 `<App />`。保留应用自身的模式控制，仅当宿主的初始模式不同时才更改显式模式。
4. 构建应用，并在完成前检查渲染后的页面。