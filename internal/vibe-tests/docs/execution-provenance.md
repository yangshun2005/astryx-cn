# 执行溯源

Vibe-test 运行器可为每个结果输出一个可选的、执行器中立的溯源附属文件。规范文件名为 `<promptId>.provenance.json`，与 `<promptId>.tsx` 和 `<promptId>.json` 并列。

版本 1 契约由以下内容定义：

- `schema/execution-provenance-v1.schema.json`，供 JSON Schema 使用方使用
- `src/provenance.ts`，提供 TypeScript 类型和解析器

未知字段会被保留，生产者可添加数据而不破坏版本 1 读取器。读取器会拒绝未知的 `schemaVersion`。缺少附属文件或附属文件部分有效时，使用旧版聚合回退。附属文件存在但格式错误则视为错误。

## 示例

```json
{
  "schemaVersion": 1,
  "task": {
    "id": "profile-card",
    "sha256": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
  },
  "fixture": {
    "id": "starter-project",
    "commit": "0123456789abcdef"
  },
  "condition": "setup",
  "rep": 1,
  "executor": {
    "harness": "local-cli",
    "model": "example-model",
    "effort": "standard",
    "harnessVersion": "2.0.0",
    "runnerVersion": "1.4.0"
  },
  "execution": {
    "status": "succeeded",
    "startedAt": "2026-01-02T03:04:05.000Z",
    "finishedAt": "2026-01-02T03:04:09.000Z",
    "durationMs": 4000,
    "attempt": 1,
    "retry": 0
  },
  "usage": {
    "inputTokens": 1200,
    "outputTokens": 340,
    "source": "runner",
    "complete": true
  }
}
```

`executor` 下的标签是不透明的。框架不解释也不限制运行器、harness、模型或 effort 名称。

## 聚合

`universal-aggregate.ts` 保留每次运行的摘要，并按以下方式对结果分组：

- harness 和模型一起
- fixture（存在时）
- condition（存在时）

它接受 `--harness`、`--model`、`--fixture` 和 `--condition` 过滤器。旧结果归入 `unknown` harness 和模型标签下。

附属文件的时长和 token 用量优先于旧版元数据和估算值。每个选中的值都带有来源和质量。仅当组内每次运行都有 `usage.complete: true` 时，token 总量才可比较；否则分组总量为 `null`，由完整/不完整运行计数说明原因。

## 运行器迁移

现有运行器无需更改其结果 JSON。在每个结果旁添加附属文件，让 `collect-results.mjs` 复制它。当 setup 或 adoption 运行器需要 harness、模型、effort、condition 和重复次数的笛卡尔积时，使用 `src/provenance-matrix.ts` 中的 `expandExecutionMatrix`。

这是增量适配器契约：专用运行器分支可以变基、输出版本 1 附属文件，并保持当前任务/结果格式。不要在溯源中放入原始提示、文件系统路径、凭据或工具记录；改用稳定的 ID 和哈希。