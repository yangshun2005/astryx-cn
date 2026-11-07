<!--
对照组：不追加自检块。
本文件刻意只含注释。setup-purity.mjs 会剥离 HTML 注释，发现无剩余文本后不注入任何内容——因此 agent 看到的 astryx-init 提示词与 `generateCompressedIndex()` 当前输出的完全一致。这是候选变体（B、C）的对照基线。
-->