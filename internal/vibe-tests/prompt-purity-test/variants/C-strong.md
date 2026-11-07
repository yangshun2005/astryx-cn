停止——在输出最终文件之前，必须进行自我审计。Astryx 是这里唯一的样式系统；直接使用 CSS/Tailwind 就是失败，不是备选方案。请对照你的文件运行以下检查清单，并在完成前修复每一处命中项：

1. 用 grep 检查你自己的输出中是否存在：`className=`、`style={{`、`import "*.css"`、`@apply`、Tailwind 工具类（例如 `flex`、`p-4`、`text-sm`、`bg-*`、`rounded-*`）、用于布局的原始 `<div>/<span>`，以及硬编码值（`#hex`、`rgb(`、`NNpx`、`NNrem`）。
2. 对每一处命中项：删除它，改用 Astryx 组件（布局、间距、表面、排版都有对应组件）或使用带 token 的 `xstyle` prop。只要存在组件，就绝不要用原始元素 + CSS。
3. 如果你不能 100% 确定某个组件/prop 存在，必须用 `astryx component <Name>` 或 `astryx search "<thing>"` 验证——不要猜测，也不要退回到 CSS。
4. 只有当重新阅读后步骤 1 中的项目为零时才算完成。如果仍有任何残留，你就没有完成——修复它们并重新审计。