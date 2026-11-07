# 理想参考图

Night Watch 设计维度的参考设计。生成的截图会与这些"目标"图进行对比。

## 命名约定

用 `__`（双下划线）分隔 promptId 与视口/主题。这样可支持 `feature-card-1` 这类多段 prompt ID。

```
{promptId}__{viewport}__{theme}.png — 特定（优先）
{promptId}__{viewport}.png          — 仅视口（两种主题共用）
{promptId}.png                      — 通用回退（所有组合共用）
```

示例：

- `cwm-1__desktop__light.png`：特定于桌面 + 浅色
- `ty-3__desktop.png`：仅桌面，浅色和深色共用
- `dd-3.png`：所有视口/主题组合的回退

## 要求

- **格式：** PNG
- **视口：** 1920×1200（Figma 导出分辨率）
- **来源：** Figma 设计稿或手工构建的 HTML，不得由 3 个目标（Astryx、baseline、HTML）中的任何一个生成
- **评审：** 提交前至少一名设计师评审理想图

## 当前理想图

覆盖 55 个 prompt ID（共 62 张图）。背景见 [issue #733](https://github.com/meta/astryx/issues/733)。

多图 prompt 用 `__` 后缀区分不同状态/视图：

- `rc-1.png` + `rc-1__menu.png`：折叠和展开菜单
- `rc-3.png` + `rc-3__desktop.png`：移动端和桌面端
- `wd-1.png` + `wd-1__cart.png`、`wd-1__shipping.png`、`wd-1__payment.png`、`wd-1__confirm.png`：结账流程各步骤
- `wd-5.png` + `wd-5__dialog.png`：页面和对话框状态