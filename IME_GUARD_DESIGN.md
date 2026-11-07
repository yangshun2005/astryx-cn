# IME 组合输入守卫 — 共享基础设计

> 供评审者阅读的草稿设计文档。未发布。分支：
> `nynexman4464/refactor/ime-guard-shared-base`。

## TL;DR 建议

1. **保留 `isImeKeyEvent` 作为唯一的底层谓词。** 它是正确的原语。**不要**添加 `useImeSafeKeyDown` 包装 Hook —— 原因见 §3，那会过度设计，且在 7 处中的 2 处完全错误。
2. **将其从 `hooks/useFocusTrap.ts` 迁移到 `utils/ime.ts`**（与 `utils/` 中纯谓词约定一致的可发现位置），权威的"为什么"文档注释只放在那里，别处不放。
3. **从 `useFocusTrap.ts` 重新导出（已弃用）以保持向后兼容**，并将 3 个现有导入方 + 3 个内联重复实现迁移到共享谓词。
4. 给 lint 规则（独立工作流）一个可指向的受认可符号：
   `@astryxdesign/core/src/utils/ime#isImeKeyEvent`。

## 1. 清单（core 中 7 处，lab 中 1 处不同模式）

| # | 位置 | 守卫 | 处理器 | 检测方式 | 事件来源 | 备注 |
|---|------|--------|---------|-----------|--------------|-------|
| 1 | `hooks/useFocusTrap.ts` | Escape（关闭焦点陷阱） | 原生 `keydown` | `isImeKeyEvent`（权威定义曾在此） | 原生 DOM `KeyboardEvent` | 也是导出辅助函数所在处 —— 对通用工具而言位置奇怪。 |
| 2 | `Typeahead/BaseTypeahead.tsx` | Enter/Esc/方向键/Home/End（候选导航 + 提交） | React `onKeyDown` | `isImeKeyEvent(e.nativeEvent)` | `e.nativeEvent` | 在所有按键逻辑之前提前返回。通过 PR #4860 添加。 |
| 3 | `PowerSearch/PowerSearchEditPopover.tsx` | Enter（保存）、Escape（取消） | React `onKeyDown` | `isImeKeyEvent(e.nativeEvent)` | `e.nativeEvent` | 在 Enter/Escape 分支之前提前返回。 |
| 4 | `Dialog/Dialog.tsx` | Escape（关闭） | 原生 `keydown` | `isImeKeyEvent(event)` **+** `hasActiveFocusTrapEscape()` | 原生 DOM `KeyboardEvent` | 复合守卫；还会让位于分层弹层。 |
| 5 | `ContextMenu/ContextMenu.tsx` | Escape（关闭） | 原生 `keydown`（document 监听器） | **内联** `e.isComposing \|\| e.keyCode === 229` | 原生 DOM `KeyboardEvent` | 重复的谓词。 |
| 6 | `Tooltip/useTooltip.tsx` | Escape（关闭，WCAG 1.4.13） | 原生 `keydown`（document 监听器） | **内联** `e.isComposing \|\| e.keyCode === 229` | 原生 DOM `KeyboardEvent` | 重复的谓词。 |
| 7 | `Chat/ChatComposerInput.tsx` | Enter（提交） | React `onKeyDown` | **内联** `e.nativeEvent.isComposing \|\| e.nativeEvent.keyCode === 229` | `e.nativeEvent` | 重复的谓词；仅守卫 Enter。 |
| — | `lab/CodeEditor/CodeEditor.tsx` | **onInput**（非 keydown） | React `onInput` + `onCompositionStart/End` | **ref 跟踪**（`isComposingRef`） | 不适用 | 不同模式且不同关注点 —— 见 §4。保持不变。 |

### 观察到的检测风格
- **共享谓词**（第 1–4 处）：调用 `isImeKeyEvent(...)`。
- **内联重复**（第 5–7 处）：直接写 `isComposing || keyCode === 229`。
- **ref 跟踪**（CodeEditor）：跟踪 `compositionstart`/`compositionend` 来门控 *input/change* 路径，而非 keydown 命令。

### SyntheticEvent 与 nativeEvent
- React 的 `KeyboardEvent` SyntheticEvent 确实暴露 `isComposing`。但 React 处理器位置（BaseTypeahead、PowerSearch、Chat）刻意读取 `e.nativeEvent`，以避免任何 React 规范化差异并访问 `keyCode`。
- 原生监听器位置（focus-trap、Dialog、ContextMenu、Tooltip）直接传入 DOM `KeyboardEvent`。
- 该谓词是**结构化类型**（`{isComposing?; keyCode?}`），因此两种形态都可接受，无需类型转换。

### `keyCode === 229` 是否仍然关键？—— 是。
`229` 是浏览器在按键事件正被 IME 处理时报告的哨兵 keyCode。某些 IME 和旧版 Safari 在触发组合中的 `keydown` 时 `isComposing` **尚未**为 `true`，但确实报告 `229`。去掉该回退会使这些情况回归。在浏览器矩阵审计给出相反结论之前，保留两个信号。（仅在 `utils/ime.ts` 中记录一次。）

## 2. 应放置的位置
`utils/` 已容纳纯事件/数据辅助函数（`getKey`、`mergeProps`、`composeEventHandlers`、`inputAria`），每个都是 `<name>.ts` + `<name>.test.ts`，并有一个桶文件 `index.ts`。`isImeKeyEvent` 是纯谓词 —— 它属于那里，而不是埋在 focus-trap Hook 中。新文件：`packages/core/src/utils/ime.ts`，从 `utils/index.ts` 重新导出。`useFocusTrap.ts` 保留一个已弃用的重新导出，以免现有相对导入失效。

## 3. 包装 Hook（`useImeSafeKeyDown`）—— 已拒绝，附原因
一个在组合输入期间空操作的 `useImeSafeKeyDown(handler)` 听起来整洁，但：
- **调用点并不统一。** 4 处在 `useEffect` 内使用原生 DOM 监听器（不是该 Hook 能包装的 React `onKeyDown` prop）；3 处使用 React 处理器。
- **在 2 处包装是错误的。** `useFocusTrap` 和 `Dialog` 将 IME 检查与*其他*条件（`isTopEscapeHandler`、`defaultPrevented`、`hasActiveFocusTrapEscape()`）组合，且必须仍运行这些周边逻辑 —— 一刀切地"在组合输入期间空操作整个处理器"会改变行为。
- **该谓词已是最小正确原语。** 提前的 `if (isImeKeyEvent(e)) return;` 明确、可 grep、可被 lint 定位。
- 包装器会引入一个 Hook、一个渲染标识关注点，以及 lint 规则必须特殊处理的第二样东西 —— 净负面。**建议：谓词 + lint 规则。**

## 4. CodeEditor 有意不在范围内
`lab/CodeEditor` 通过 `compositionstart/end` ref 跟踪来守卫 `onInput`（否则会将部分 CJK 音节写入 `value`）—— 这与"不要把组合中的 keydown 当作命令"是不同的关注点，因此共享的 keydown 谓词无法替代它。**标记：** 其 `handleKeyDown` 处理 Escape（启用 Tab 移动焦点）和 Tab 时*没有* IME 守卫，因此组合中的 Escape/Tab 可能误触发。潜在问题，位于 `lab`，且属行为层面 —— 留待单独修复。

## 5. 本分支实现的内容（仅低风险）
- 新增 `utils/ime.ts`（权威谓词 + 唯一权威文档注释）和 `utils/ime.test.ts`（双信号覆盖）。
- `utils/index.ts` 重新导出 `isImeKeyEvent`。
- `useFocusTrap.ts`：移除本地定义；从 `utils/ime` 导入；保留已弃用的重新导出以保持向后兼容。
- 将导入方迁移到工具路径：`Dialog`、`BaseTypeahead`、`PowerSearch`。
- 去重 3 个内联位置（`ContextMenu`、`Tooltip`、`ChatComposerInput`），改为调用 `isImeKeyEvent`，理由说明交由 `utils/ime.ts`。
- **无行为变更** —— 各处布尔逻辑相同；纯整合。