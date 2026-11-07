## 将 Astryx 添加到已有的 Tailwind 应用

公开的全新项目方案不能替代已有的样式表。
保留宿主应用的 CSS，在其周围添加 Astryx：

- 保留 `@import 'tailwindcss';`，并将 layer 声明放在**它上面**：

  ```css
  @layer reset, theme, base, astryx-base, astryx-theme, components, utilities;

  @import 'tailwindcss';
  @import '@astryxdesign/core/reset.css';
  @import '@astryxdesign/core/astryx.css';
  @import '@astryxdesign/theme-neutral/theme.css';
  ```

- 不要导入 `@astryxdesign/core/tailwind-theme.css`，除非宿主希望
  Astryx 接管其 Tailwind 词汇。该文件会刻意映射常见的颜色、
  圆角、阴影、字体和间距名称。
- 向 `<Theme>` 传入与宿主初始模式一致的显式 `mode`。
  保留应用自有的明暗切换，不要替换它。
- 如果两套系统占用同一个自定义属性名，在主题边界处重新声明宿主的值，
  而不是删除任一方的词汇。
- 除了构建，还要验证渲染后的应用。级联和继承颜色的变化可能编译通过却出问题。