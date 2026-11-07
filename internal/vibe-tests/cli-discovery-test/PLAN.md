# CLI 发现 Vibe 测试

**状态：** 提案 / 设计（尚未实现）
**负责人：** 待定
**相关：** `../README.md`（Checker Protocol）、`../src/setup-environment.mjs`、`../src/universal-eval.ts`

---

## 1. 问题

现有每个 vibe 测试都在问：_"假设 agent 已经拿到文档（AGENTS.md，或写着'使用 CLI'的 README），它写的代码有多好？"_

本实验问的是**相反的、上游的**问题：

> **当 agent 进入一个刚跑过 `npm install @astryxdesign/core` 的项目——没人跑过 `astryx init`——它会不会自己发现 CLI？哪条"防呆"通道能让这件事_稳定_发生？**

这就是**先有鸡还是先有蛋的问题**。让 agent 可靠使用 CLI 的，是 `astryx init` 注入到 `AGENTS.md`/`CLAUDE.md`/`.cursorrules` 的 `<!-- ASTRYX:START -->` 速查表。但这份速查表是_由 CLI_ 安装的。只装包、从不跑 `init` 的用户，agent 永远不知道 CLI 存在。我们想知道哪些被动通道能补上这个缺口。

**注意：** 当前 `environments/project-astryx/README.md` 已包含"使用 CLI 查询组件 props"。这使它不适合作为_本_实验的对照组——它预先埋好了答案。这里，README 提示是**被测条件**之一，不是常量。

---

## 2. "有效"的含义

用户的措辞是_"什么能**稳定**有效"_。所以主要输出不是单一分数，而是**每条通道的可靠性率**：在多次全新独立运行中，agent 发现并使用 CLI 的比例及其方差。10 次中 9 次有效的通道，胜过 10 次中 5 次且方差大的通道，即使后者偶尔产出更漂亮的代码。

---

## 3. 假设

| #   | 通道                         | 假设                                                                                     | 能触达 agent？ |
| --- | ---------------------------- | ---------------------------------------------------------------------------------------- | -------------- |
| H1  | 裸安装（对照）               | agent 几乎从不发现 CLI；它们猜 API 并产生幻觉                                            | —              |
| H2  | README 顶部提示              | 中等提升；取决于 agent 是否打开 `node_modules/@astryxdesign/core/README.md`              | 有时           |
| H3  | 类型/JSDoc `@see` 横幅       | 高且稳定的提升；agent 会本能地读 `.d.ts`/悬停提示，且不受 pnpm 脚本拦截影响              | 通常           |
| H4  | Postinstall 横幅             | 低/不可靠；仅当 agent 自己跑 install 并读 stdout 才可见；pnpm 10 跳过依赖脚本            | 很少           |
| H5  | CLI 首次运行提示             | 把_感知 → init_，但只在 agent 已运行 `astryx` 后才触发                                   | 有条件         |
| H6  | 存在 AGENTS.md（天花板）     | 接近 100%；这是 `init` 产出的已知良好状态                                                | 是             |
| H7  | README + 类型横幅（组合）    | 仅用被动、随包发布的通道就接近天花板                                                     | 通常           |

我们要做的决策：**找出最小的被动通道集合（能随 npm 包发布、无需用户操作），使其发现率接近 H6 天花板。**

---

## 4. 变量

### 自变量——发现通道（"条件"）

定义在 [`conditions.json`](conditions.json)。每个条件 = 裸的真实项目 **加上** 零个或多个通道补丁。只有通道变化；提示词、人设、环境、包、评估保持不变（Checker Protocol §2）。

### 因变量——发现漏斗（新增）+ 下游质量（复用）

**发现漏斗**（本实验重点；每次运行测量）：

1. **感知**——agent 是否提到过 CLI？（转录）
2. **调用**——它是否真的运行了任何 `npx astryx …`？（**ground truth**，见 §6.2）
3. **Init**——它是否运行了 `astryx init`？
4. **文档检索**——它是否运行了 `astryx component` / `build` / `search` / `template`？

**下游质量**（复用 `universal-eval.ts` 不变，以便与既往运行可比）：

5. 正确性、无障碍、代码质量、效率、可维护性
6. 逃生舱 / 幻觉率（按 `prompts/evaluator.md`：幻觉 props、无效 `--xds-*`/`--space-*` token、幽灵组件）

漏斗是独立贡献；质量维度回答后续问题"……发现是否真的让输出更好？"

---

## 5. 条件（矩阵）

机器可读版本见 [`conditions.json`](conditions.json)。摘要：

| id                | 存在的通道                                                                      | 角色                |
| ----------------- | ------------------------------------------------------------------------------- | ------------------- |
| `c0-bare`         | 无。`package.json` + 符号链接包。任何地方都不提 CLI。                           | **对照（地板）**    |
| `c1-readme`       | Core `README.md` 顶部加"使用 AI agent？运行 `npx astryx init`"提示              | 候选                |
| `c2-types`        | 生成的 `@see npx astryx …` 横幅注入每个组件的 `.d.ts`                           | 候选                |
| `c3-postinstall`  | 模拟安装时横幅（见 §9 真实性说明）                                              | 候选（探针）        |
| `c4-firstrun`     | CLI 存在；任何 `astryx` 调用都打印"运行 init"提示                               | 候选                |
| `c5-agents-md`    | 完全初始化：`AGENTS.md` 含真实 `<!-- ASTRYX:START -->` 块                       | **天花板**          |
| `c6-readme+types` | c1 + c2                                                                         | 候选组合            |
| `c7-all-passive`  | c1 + c2 + c4（所有无需用户操作即可发布的）                                      | 候选组合            |

每个候选都与 **`c0-bare`**（相对什么都不做的提升）和 **`c5-agents-md`**（离天花板多近）比较。

---

## 6. 测试框架设计

复用机制 B（`setup-environment.mjs`，符号链接隔离项目）——这是 Checker Protocol 想要的代表性环境路径。需要三处新增。

### 6.1 通道补丁（从一个裸基座组合条件）

不手工维护 8 个模板目录（有漂移风险，违反"只有 SUT 变化"），而是从**一个裸基座模板** + 小的、可审查的**补丁函数**构建每个条件，每通道一个。这也让我们能在**真正构建通道之前**测试它们（例如把类型横幅注入 `dist/**/*.d.ts` 的_副本_；补丁 `README.md` 的副本），从而让实验决定是否发布该功能——而不是反过来。

补丁（每个由条件开关）：

- `patch:readme-callout`——把 AI 提示前置到项目副本的 core `README.md`。
- `patch:type-banner`——把 core 复制进项目（非符号链接），并把标准 `@see` 横幅注入每个组件的公开 `.d.ts`。
- `patch:agents-md`——运行真实的 `npx astryx init --features agents --agent-docs-path AGENTS.md`（复用 `interactive.ts` 的 `installAgentsDocs()`）。
- `patch:first-run-nudge`——把 `astryx` bin shim 换成同时打印提示的版本。
- `patch:postinstall-banner`——见 §9。

> **真实性守卫：** 每个补丁只能触碰真实 npm 消费者实际会收到的文件。Core 发布 `README.md` 和 `dist/**`（其 `package.json#files`），所以 c1/c2 是真实的。`AGENTS.md` _不_随包发布——它只在 `init` 后存在——这正是 c5 代表"用户跑了 init"（天花板）而非可发布默认值的原因。

### 6.2 Ground-truth CLI 插桩（别信自报）

现有符号链接 `node_modules/.bin/astryx → packages/cli/bin/astryx.mjs` 改为**日志 shim**：一个极小的包装器，把 `{ts, argv}` 追加到 `projects/<promptId>/.astryx-invocations.log`，然后 exec 真实 bin。这给出_每次_ CLI 调用（调用、init、哪些子命令）的**客观**记录，独立于 agent 声称。这是最重要的测量——发现由该日志定义，而非转录。

### 6.3 扩展结果 Schema（自报作为次要信号）

扩展子 agent 已写的每任务结果 JSON，加一个发现块（自报；与 ground-truth 日志交叉核对）：

```json
{
  "completedAt": "<ISO>",
  "discovery": {
    "sawCliReference": true,
    "howDiscovered": "readme | types | nudge | guessed | none",
    "ranInit": false,
    "cliCommandsRun": ["astryx component Button", "astryx build \"...\""]
  }
}
```

新聚合器 `discovery-aggregate.ts`（`universal-aggregate.ts` 的兄弟），按条件连接 ground-truth 日志 + 自报 + 现有 `universal.json`，输出漏斗率、质量增量和一致性统计。

---

## 7. 流程

1. **提示词**——复用 `test-sets/default.json` 的分层样本（各条件提示词相同，按 Checker Protocol §2/§3）。精选子集见 [`prompts.json`](prompts.json)。`expectedComponents` 仅用于评估，从不展示给 agent（不变量 §3）。
2. **重复次数**——每个 `条件 × 提示词` 用全新、无上下文的子 agent 运行 **K ≥ 5** 次（不变量 §5）。发现是随机的；率需要重复，不是单样本。
3. **Agent/模型**——发现_依赖测试框架_（Cursor 自动加载 `AGENTS.md`/`.cursorrules`；Claude Code 自动加载 `AGENTS.md`/`CLAUDE.md`；裸 API agent 什么都不自动加载）。至少运行：(a) Claude Code 子 agent（现有 runner），(b) 一个"不自动加载"的 agent，以把文件读取通道（types/README）与框架自动加载通道（AGENTS.md）隔离。每个结果记录 agent/模型。
4. **盲评**——评估用现有确定性 `universal-eval.ts`（已对条件盲）。发现日志是客观的。循环中无人工评分。

### 运行命令（提议，待框架工作落地后）

```bash
# Generate all conditions for a stratified sample, K reps each (prints iter IDs)
node internal/vibe-tests/cli-discovery-test/setup-discovery.mjs --sample 8 --reps 5

# Parent agent spawns fresh subagents per task file (as in the interactive runner)

# Evaluate
pnpm -F @astryxdesign/vibe-tests aggregate --iteration <id>        # reuse: quality dims
tsx internal/vibe-tests/cli-discovery-test/discovery-aggregate.ts --experiment <expId>   # new: funnel + consistency
```

---

## 8. 分析与决策规则

每个条件按 agent 报告：

- **发现率** = P(运行了任何 `npx astryx`)，所有运行上的 Wilson 95% CI。
- **Init 率**、**文档检索率**（同样处理）。
- **一致性** = 该 CI 的宽度 + 跨提示词/重复的方差。通道只有 CI 紧且跨提示词类别和两个 agent 都成立，才算"一致"。
- **质量增量** = 平均 `overall`（及幻觉率）相对 `c0-bare`。

**预注册决策规则（运行前调整）：**

- 通道**发布**，若其发现率 **≥ 80%** 且 CI 下界 **> c0-bare 上界**，且跨两个 agent 都成立。
- 优先选达到 **`c5-agents-md` 天花板 ≥ 90%** 的**最小被动组合**（`c6`/`c7`）。该组合成为推荐的默认防呆方案。
- 任何发现率均值高但方差大的通道标为"不可靠"，不计为主要机制（这正是"稳定"的要求）。

---

## 9. 风险与局限

- **Postinstall 真实性（c3）。** 符号链接环境中依赖已存在，agent 从不跑 install，也从不看横幅。真实模拟需从未安装项目开始并 shim 安装器 stdout——在 pnpm monorepo 中慢且脆弱。**建议：** 把 c3 当作低优先级、单独运行的探针（或分析推理），因为我们已知 postinstall 被 pnpm 10 / CI / `--ignore-scripts` 抑制。别让它阻塞核心矩阵。
- **框架依赖。** `AGENTS.md` 自动加载是_agent 框架_的属性，不是包的。类型/README 通道是我们能发布且不依赖框架的——在报告中点明，以免过度归功 c5。
- **自报偏差。** 由 ground-truth bin shim 缓解（§6.2）；自报是次要的。
- **c2 的复制 vs 符号链接。** 注入类型横幅需复制 core（非符号链接）。其余保持一致，使唯一差异是横幅文本。
- **提示词敏感性。** 更难提示词上发现更可能（猜测失败）。跨复杂度分层，以免高估或低估率。

---

## 10. 阶段

1. **阶段 0（本文档）**——设计 + 文件夹脚手架。← 我们在这里
2. **阶段 1**——框架：裸基座模板、补丁组合器、bin-shim 日志、`discovery-aggregate.ts`。用 `scripts/smoke-test.sh` 风格夹具验证（无 LLM）。
3. **阶段 2**——试点：小样本上 `c0-bare` vs `c5-agents-md`，验证漏斗 + 确认地板/天花板差距真实。
4. **阶段 3**——完整矩阵（所有条件 × K 重复 × 2 agent）。
5. **阶段 4**——报告 + 决策：实际发布哪些通道（反馈到 core 包的 P0/P1 建议）。

---

## 11. 文件夹布局

```
internal/vibe-tests/cli-discovery-test/
├── PLAN.md            # this file — experiment design
├── conditions.json    # the channel matrix (independent variable)
├── prompts.json       # curated stratified prompt subset (from default.json)
├── setup-discovery.mjs        # (Phase 1) builds conditions from base + patches
├── discovery-aggregate.ts     # (Phase 1) funnel + consistency aggregator
└── results/                   # (gitignored) per-experiment output
```

---

## 12. Checker Protocol 合规

| 不变量                        | 本实验如何遵守                                                                                                                                          |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| §1 公平评估器                 | 所有条件用同一 `universal-eval.ts` + 同一客观发现日志                                                                                                   |
| §2 只有 SUT 变化              | 一个裸基座 + 单通道补丁；提示词/人设/环境相同                                                                                                           |
| §3 从不泄露答案               | `expectedComponents` 仅评估用；不从预期组件预构建任何 CLI 命令；_对照_完全无 CLI 提示                                                                   |
| §4 代表性环境                 | 补丁只触碰真实 npm 消费者收到的文件（`README.md`、`dist/**`）；`AGENTS.md` 明确是 `init` 后天花板，非发布默认值                                          |
| §5 无上下文子 agent           | 每次运行全新子 agent；K 重复独立；无共享会话                                                                                                            |

---

## 13. 试点状态（已实现）

**无 LLM** 端到端构建并验证（验证机制，非结论）：

- `setup-discovery.mjs`——从裸"刚安装 `@astryxdesign/core`"基座（core + theme 符号链接，**CLI 未列出**）为每个 `条件 × 提示词 × 重复` 构建一个隔离项目，把 ground-truth `astryx` 日志 shim 接入 `node_modules/.bin`，写任务文件 + `discovery-config-<expId>.json`。试点接入 `c0-bare`（地板）和 `c5-agents-md`（天花板）；其他条件在加入 `PATCHES` 前受保护。
- `discovery-aggregate.ts`——读调用日志（+ 自报），打印带 Wilson 95% CI 的漏斗，写 `discovery-summary-<expId>.json`。

已验证：`c0` 无 `AGENTS.md`；`c5` 收到真实 `<!-- ASTRYX:START -->` 速查表；shim 在每次 `astryx` 调用记录 `{ts, argv}`；任务提示词**不含** `expectedComponents`（无答案泄露，§3）。

运行：

```bash
# 1. generate conditions (curated prompts, or --sample N / --reps K)
node cli-discovery-test/setup-discovery.mjs --prompts fwc-1,ps-1
# 2. spawn a FRESH, context-free sub-agent per printed task file
# 3. score the funnel
npx tsx cli-discovery-test/discovery-aggregate.ts --experiment <expId>
```

下一步：用真实 agent 在精选集上以 **K≥5** 跑步骤 2（n=2 的试点 CI 太宽，无法区分地板与天花板——这正是需要更多重复的原因），然后接入 `c1-readme` / `c2-types` / `c4-firstrun`。

已知说明：在此 monorepo 内，生成的 `AGENTS.md` 打印 `pnpm exec astryx`（run 前缀检测看到 pnpm）；独立消费者