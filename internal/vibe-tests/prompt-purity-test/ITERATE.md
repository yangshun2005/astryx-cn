# ITERATE — 自主提示词纯净度搜索（由 `/loop` 驱动）

这是每个 tick 的运行手册，用于**自动演进 astryx-init 提示词**，直到 agent 可靠地不再偏向原生 CSS/Tailwind（或在偏向时能自我纠正）。它由 `/loop` 命令运行，自定节奏（无固定间隔），以每次 harness 运行完成为门控。

**调用方式**（在仓库根目录、活跃的 Cursor agent 会话中）：

```
/loop follow internal/vibe-tests/prompt-purity-test/ITERATE.md
```

无间隔 => 动态模式：立即运行一次迭代，然后在当前 harness 运行结束后唤醒并运行下一次。完全自主，无迭代上限——仅在确认胜出或真正进入平台期时停止（见停止条件）。接受 token 消耗：更好的已发布提示词能节省更多下游成本。

前置条件（一次）：`cursor-agent` CLI 必须已登录（`cursor-agent status`）。若未登录，停止并告知用户；不要循环。

---

## 状态（跨 tick 保留）

维护运行日志，使每个 tick 都能与所有先前尝试对比，且永不重复测试同一措辞：

- 路径：`"$TMPDIR/astryx-purity/_loop/iterations-log.json"`（在仓库外，与所有运行时产物一致）。首次 tick 若不存在则创建：`{ "tick": 0, "champion": "B-selfcheck", "history": [] }`。
- 每个 `history[]` 条目：`{ tick, expId, ranConditions, variantHashes, rates: { <cond>: { veeredUncaught, caughtGivenVeered, finalPurityMean, n } }, note }`。
- "champion" = 目前找到的最佳变体（`veeredUncaught` 最低，平局时取 `finalPurityMean` 更高者）。初始 champion = `B-selfcheck`。

`A-control` 是不可变基线——永不编辑 `variants/A-control.md`。

---

## 单个 tick

### 1. 读取当前状态

- 读取 `iterations-log.json`，确定当前 `champion`。
- 若 `history` 为空，这是种子 tick：一起运行 `A-control`、`B-selfcheck`、`C-strong`（使用下方探索设置）以建立基线并选出首个 champion。种子 tick 跳过步骤 2。

### 2. 编写一个新挑战者（种子 tick 跳过）

- 基于上次运行在 `purity-summary.json` 中的逐次详情（`veeredUncaught` 运行中哪些标记存活、`ranDocRetrieval` 是否偏低、捕获是否发生较晚），编写一个针对观察到的失败模式的新变体措辞。可变化的思路（一次只改一个轴，以便了解什么重要）：
  - 命令强度 / 顺序（自检在前 vs 在后）
  - 显式标记清单 vs 简洁规则
  - 强制 `astryx component`/`search` 验证 vs 不强制
  - "重读文件"框架 vs "输出前"框架
  - 长度（简洁的一行有时胜过一大段文字）
- 写入新文件 `variants/g<NN>-<slug>.md`（永不覆盖 A/B/C 或先前代次——历史必须可复现）。在 `conditions.json` 的 `conditions[]` 中添加匹配条目：`{ "id": "g<NN>-<slug>", "role": "candidate", "variant": "g<NN>-<slug>" }`。
- 不要测试文本哈希已出现在 `history` 中的措辞。

### 3. 运行（探索 = 便宜、快速）

- 只比较三个条件，保持运行聚焦：control、champion、challenger。
- 探索设置：`--prompts dd-1,tc-6 --reps 3`。
- 在后台启动并设置完成唤醒（动态 `/loop`）。先生成 id，使唤醒载荷携带它：

```bash
EXP=$(node -e "console.log(require('crypto').randomBytes(4).toString('hex'))")
( node internal/vibe-tests/prompt-purity-test/run-purity.mjs \
    --exp "$EXP" --conditions A-control,<champion>,<challenger> --prompts dd-1,tc-6 --reps 3 \
    --concurrency 6 ; \
  echo "AGENT_LOOP_WAKE_purity {\"expId\":\"$EXP\",\"phase\":\"explore\"}" ) 2>&1
```

通过 Shell 工具启动，设置 `block_until_ms: 0`，并在 `^AGENT_LOOP_WAKE_purity` 上设置 `notify_on_output`。同时按 loop 技能设置一个长回退心跳（例如 `sleep 1800`）。然后结束本轮——不要阻塞。

### 4. 唤醒后——评估

- 读取 `"$TMPDIR/astryx-purity/$EXP/purity-summary.json"`。
- 追加 `history` 条目（每个条件的 rates + 变体哈希）。
- 比较 challenger 与 champion：若 challenger 的 `veeredUncaught` 均值更低（平局时取 `finalPurityMean` 更高者），则成为新 champion。更新 `champion`。
- 向用户报告一行 tick 摘要：control vs champion vs challenger 在 `veeredUncaught`、`caughtGivenVeered`、purity 上的表现——以及 champion 是否变化。

### 5. 决定：确认、继续或停止

- **晋升到确认**：当 champion 在探索中胜过 `A-control`（`veeredUncaught` 更低且 purity 更高）。运行确认：
  `--conditions A-control,<champion> --prompts dd-1,dd-5,wd-1,wd-4,tc-6,ps-1 --reps 7`
  （K>=5，完整精选集），同样的后台+唤醒模式，`"phase":"confirm"`。
- 确认唤醒后：若 champion 是**决定性的**——其 `veeredUncaught` Wilson 上界低于 control 的下界，且平均 purity 更高——进入 Ship。否则记录它，继续搜索（回到步骤 2）。
- **继续**（无间隔）：重新设置并开始下一个 tick（步骤 2）。

---

## 停止条件

当以下任一成立时停止循环（杀掉任何 watcher/heartbeat PID，不再重新设置）：

- **胜出已确认**——champion 决定性地通过了确认运行。进入 Ship。
- **平台期**——最佳 `veeredUncaught` 在最近 3 个不同 challenger 上均未改善。报告目前最佳；仅当它也决定性地胜过 control 时才 ship，否则报告没有变体能可靠胜过 control，交还给用户。
- **地板**——`A-control` 在确认集上已显示约 0 `veeredUncaught`（无可修复）；报告并停止。
- 用户说停止。

---

## Ship（仅在确认胜出后）

这完成 Phase 4 / `ship` 待办：

1. 将获胜措辞写入 `packages/cli/src/commands/agent-docs.mjs` 中的 `generateCompressedIndex()`——将自检行添加到生成的块中（遵守 `stylex` / `tailwind` / `css` 样式系统分支）。
2. 更新测试 `packages/cli/src/commands/agent-docs.test.mjs`，断言新自检文本存在于生成的索引中。
3. 运行该文件的 CLI 测试。
4. 报告：获胜变体文本、其确认 rates vs control、以及 diff。
5. 停止循环。

---

## 报告与安全

- 每个 tick：输出简短状态（运行了哪些条件、漏斗增量、champion 变化、下一步动作）。停止时：说明原因并给出最终胜出者 + rates。
- 所有运行保持在 `"$TMPDIR/astryx-purity"` 下（绝不在仓库中），因此并行 Cursor 工作、文件监视器和 `git status` 不受影响。仓库中仅 `conditions.json` + `variants/g*.md`（以及 Ship 时的两个 CLI 文件）发生变化。
- 永不编辑 `variants/A-control.md`。永不在任何提示词中放置 `expectedComponents`。
- 只运行一个循环——设置前检查是否已有运行中的 `AGENT_LOOP_WAKE_purity` watcher 并复用它。