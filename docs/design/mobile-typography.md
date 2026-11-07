---
schema_version: 1
template_version: 1
kind: design
id: design:mobile-typography
authority: current
archive_reason: null
superseded_by: null
approved_by: rubyycheung
approved_at: 2026-09-30
owners: [imdreamrunner, rubyycheung]
review_triggers: [visual, responsive, theming]
verified_by: [scripts/check-knowledge.mjs]
architecture: [architecture:theme-tokens, architecture:theme-authoring-contract]
components: []
families: []
deciding_specs: [spec:AST-102/DEC-2]
---

# 移动端排版设计规范

## 用户意图

在粗指针 iOS 设备上，输入框会自动采用 16px 基准最小值；本设计让其余文本样式与之平衡。

## 设计原则

- **DR1 — 阅读基准。** 移动端基准 MUST 为 `M = max(B, 16)`；16px 基准即 `1rem`。用户设置文本大小后，该值取代此基准，DR2 的钉定不适用（`spec:AST-102` FR3）。
- **DR2 — 钉定目标。** 所选角色 MUST 保留其桌面端大小；其余角色按推导出的比例缩放。

## 结构与层级

`B` 为桌面端基准，`r` 为其比例，`k` 为原始步进，`a` 为所选钉定步进。

| 桌面端比例 r      | 钉定目标；步进 a | 设计意图                                                                 |
| ----------------- | ---------------- | ------------------------------------------------------------------------ |
| `r <= 1.25`       | Display 1；`+6`  | 阅读基准增大时，保留紧凑比例的 Display 字号。                            |
| `1.25 < r < 1.414` | Heading 2；`+2` | 保留 Heading 2，同时相对正文缩小更大的 Display 角色。                    |
| `r >= 1.414`      | Heading 3；`+1`  | 保留 Heading 3，使 Display 层级在窄屏上不压过正文。                      |

所需字号关系为：

```text
M = max(B, 16)
r_pin = r * (B / M)^(1/a)
desktop(k) = round(B * r^k)
mobile(k) = round(M * r_pin^k)
```

四舍五入到最近的整数参考像素，恰好一半时向上取整；除以 16 得到 rem。钉定之上角色缩小，之下角色增大（`B < 16`，取整前）。

## 状态表示

移动端条件之外，保留桌面端字号。
当 `B >= 16` 且未设置文本大小时，移动端配置保留桌面端基准与比例。

## 响应式与输入行为

- **DR3 — 激活条件。** 默认情况下，移动端 MUST 匹配 `(width < 1024px) and (pointer: coarse)`。

宽度指布局视口；coarse 指主指针。
使用由 `spec:AST-012` 规定的有效 `lg` 断点；默认值为
1024 CSS px，上边界为开区间。
浏览器支持与回退遵循 `spec:AST-013`；浏览器特定行为
需有该浏览器的证据。

## 无障碍意图

- **DR4 — 根字号。** 移动端配置 MUST NOT 更改文档根字号。

## 代表性示例

```text
B=14; r=1.333; a=2; M=16; r_pin=1.246907324142416
heading-2: 25→25px; body: 14→16px; display-1: 79→60px
```

### 验证

| 契约     | 验证方式                 | 代表性状态                          | 失败信号               |
| -------- | ------------------------ | ----------------------------------- | ---------------------- |
| DR2      | 审查比例到目标的映射     | 低于、等于、高于 1.25/1.414         | 目标或边界错误         |
| DR1, DR2 | 比较所需字号             | 计算示例；B>=16                     | 锚点或基准改变         |
| DR3      | 检查激活条件             | 窄/宽 × 粗/细；恰好 1024px          | 逻辑连接或边界错误     |
| DR4      | 检查根字号               | 移动端配置激活/未激活               | 文档根字号改变         |

## 视觉参考

不包含规范性资源。

## 组件契约链接

不引入组件特定的契约链接。

## 决策日志

### DEC-1 — 移动端字号关系

**引用：** `design:mobile-typography/DEC-1`
**决策者：** `rubyycheung`，`2026-09-30`
**决策：** 应用 DR1–DR4，使移动端文本样式与输入框最小值平衡。

## 待解决问题

- **OQ1 — 本策略必须支持哪些桌面端基准字号与比例？**

## 内容边界

本记录定义移动端字号关系，不涉及 Props 语法、
实现结构、当前审计评分或使用方示例。