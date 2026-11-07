# Prompt Purity Vibe Test

**状态：** 已实现（测试框架）；正在迭代变体
**相关：** `../cli-discovery-test/PLAN.md`（蓝图）、`../src/universal-eval.ts`（复用的检测器）、`packages/cli/src/commands/agent-docs.mjs`（被测提示词）

---

## 1. 问题

Agent 已经能较好地使用 Astryx——但在困难任务上会**偏离**，转而使用原生 CSS / Tailwind / 内联样式 / 硬编码值，而不是 Astryx 组件 + `xstyle`/tokens。我们希望修改**注入的初始化提示词**，让 Agent **自检**其输出，并在完成前**自我纠正**。

> 在 `astryx init` 提示词中加入生成后自检，能否让 Agent 可靠地 (a) _不_在最终文件中留下原生 CSS/Tailwind，并且 (b) 在开始偏离时**发现并纠正**——从实际运行轨迹中测量？

## 2. 变量（自变量）

`astryx init --features agents` 在 `<!-- ASTRYX:START -->` 标记之间拼接的文本，由 `packages/cli/src/commands/agent-docs.mjs` 中的 `generateCompressedIndex()` 生成。实验期间我们**不**修改该源文件。每个条件 = 一个真实的 `astryx init` AGENTS.md + 一个拼接在标记内的**变体**自检块（`variants/*.md`、`conditions.json`）。只有注入的提示词变化；环境、提示词和模型保持不变（Checker Protocol §2）。获胜变体仅在最后固化进 `generateCompressedIndex()`。

## 3. 测量指标（因变量）

每次运行，基于**纯度时间线**——运行期间实时捕获的输出 `.tsx` 有序快照（`run-purity.ts` 通过 `fs.watch` + 轮询兜底），在 `purity-eval.ts` 中分类：

- `neverVeered` — 从未出现非 Astryx 样式标记。（最佳）
- `caught` — 某一步出现了标记，但**最终**文件干净。（自检生效：偏离后被捕获）
- `veeredUncaught` — 结束时标记仍存在。（需最小化的坏结果）
- `finalPurity` — 最终文件的 0–100 评分。

**偏离标记**（见 `purity-eval.ts`，可在一处调整）：`className=`、`style={{…}}`、`.css` 导入 / `@apply`、Tailwind 工具类、硬编码 `#hex`/`rgb()` 颜色（硬标记 → 驱动二元"偏离"判定）；硬编码 `px`/`rem`、样式化原生元素（软标记 → 仅影响评分）。

次要信号：`ranAstryxSelfCheck`（是否运行 `astryx search/component/docs` 进行验证——来自日志 shim），以及转录扫描"推理中捕获"（先提出 CSS 后在写入前重新考虑）。

## 4. 测试框架

- `setup-purity.mjs` — 为每个 `condition × prompt × rep` 构建一个隔离项目，**位于工作区外的沙箱**（默认 `os.tmpdir()/astryx-purity/<expId>`；`--out` 可覆盖）。裸基础 = `@astryxdesign/core` + 主题符号链接，**`package.json` 中列出 StyleX 编译器插件**以便 CLI 检测到 `stylex` 样式系统（Astryx 路径），一个记录日志的 `astryx` shim，然后执行真实的 `astryx init` + 变体拼接。写入 `tasks/<id>.json` + `purity-config-<expId>.json`。
- `run-purity.ts` — 运行器。以无头模式调用 `cursor-agent` CLI（`--print
--output-format stream-json --model claude-opus-4-8-max-fast --force --trust
--workspace <sandbox project>`），每次变更时快照 `.tsx`，将流式事件持久化到 `transcript.jsonl`，将运行时事实记录到 `run.json`。有界并发（`--concurrency`，默认 3）+ 每次运行超时，确保不会耗尽交互式 Cursor 使用。
- `purity-eval.ts` — 偏离标记检测器 + 时间线分类器（还复用 `../src/universal-eval` 的 `evaluate()` 作为补充评分）。
- `purity-aggregate.ts` — 每条件 Wilson 95% CI 比率 + 与对照的比较；写入 `purity-summary-<expId>.json`。
- `run-purity.mjs` — 一键编排器（setup → run → aggregate）；`/loop` 驱动的单元。
- `smoke-test.ts` — **无 LLM** 验证分类器 + setup 接线。

## 5. 隔离（绝不干扰并行 Cursor 工作）

- 所有运行时产物位于**工作区外的沙箱**（临时目录 / worktree），绝不在打开的仓库中，因此文件监视器、其他 Cursor Agent 和 `git status` 不受影响。
- 每个 Agent 以 `cwd` = 其沙箱项目、`settingSources: []` 运行；仓库包仅通过符号链接**读取**。`fs.watch` 仅监视沙箱。
- 使用 `cursor-agent` CLI，通过 `--workspace` 限定到沙箱；有界并发 + 每次运行超时保持交互式 Cursor 使用响应。无需单独的 API 密钥——使用你现有的 `cursor-agent` 登录。

## 6. 流程

LIVE 运行的一个前提（无 LLM 冒烟测试和 `--dry-run` 无需任何前提）：`cursor-agent` CLI 必须已登录。

```bash
cursor-agent status     # 应显示 "Logged in as ..."；否则：cursor-agent login
```

```bash
# 无 LLM 框架检查（零模型调用）——始终可安全运行
npx tsx prompt-purity-test/smoke-test.ts

# 一键：setup -> run -> aggregate（试点默认：dd-1, tc-6；A vs B；K=3）
node prompt-purity-test/run-purity.mjs --conditions A-control,B-selfcheck --prompts dd-1,tc-6 --reps 3

# 或分步执行
node prompt-purity-test/setup-purity.mjs --exp <id> --conditions A-control,B-selfcheck --prompts dd-1,tc-6 --reps 3
npx tsx prompt-purity-test/run-purity.ts --experiment <id> --concurrency 3
npx tsx prompt-purity-test/purity-aggregate.ts --experiment <id>
```

## 7. 判定规则

当变体的 `veeredUncaught` 比率显著低于对照（其上 Wilson 界低于对照的下界）和/或 `caught | veered` 与平均 `finalPurity` 显著更高，且在精选困难提示词上 K ≥ 5 时保持稳定，则该变体**获胜**。然后将其固化进 `generateCompressedIndex()`。

## 8. Checker Protocol 合规

- §2 仅 SUT 变化——一个裸基础、单变体拼接、相同的提示词/环境/模型。
- §3 绝不泄露答案——`expectedComponents` 仅用于评估，绝不出现在任务提示词中。
- §4 代表性环境——真实的"已安装 @astryxdesign/core"项目。
- §5 无上下文 Agent——每次运行全新 SDK Agent，`settingSources: []`，K 个独立重复。

## 9. 自主迭代（`/loop`）

前提就绪后，整个搜索自行运行。在实时 Cursor Agent 会话中：

```
/loop follow internal/vibe-tests/prompt-purity-test/ITERATE.md
```

动态（自定节奏）模式：现在运行一次迭代，然后在每次框架运行完成时唤醒（以编排器完成行为事件门控）并运行下一次。每轮它编写一个针对上次运行失败模式的新挑战者变体，廉价地比较对照 vs 冠军 vs 挑战者（在 `dd-1,tc-6` 上 K=3），晋升最佳者，然后在完整精选集上以 K>=5 确认领先者。完全自主，**无迭代上限**——仅在确认获胜者（然后将措辞固化进 `generateCompressedIndex()` 并更新 CLI 测试——阶段 4）或真正停滞时停止。跨轮记忆存于 `"$TMPDIR/astryx-purity/_loop/iterations-log.json"`；所有运行留在 tmpdir 沙箱中，因此你并行的 Cursor 工作绝不受影响。完整每轮流程和停止条件见 `ITERATE.md`。