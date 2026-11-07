# Astryx Vibe 测试

结构化评估，比较 LLM 在不同设计系统配置下生成 UI 代码的质量。相同提示词，不同系统，可量化的结果。

## 提示词集

提示词位于 [`test-sets/default.json`](test-sets/default.json)。每个提示词包含：

```json
{
  "id": "fwc-6",
  "category": "feature-with-constraint",
  "prompt": "Build a shipping method selector with three options: Standard (free, 5-7 days), Express ($9.99, 2-3 days), Overnight ($19.99, next day)",
  "expectedComponents": ["RadioList", "Card"],
  "complexity": "simple"
}
```

- **`prompt`**：交给 agent 的任务（所有配置完全相同）
- **`expectedComponents`**：仅用于评估，绝不展示给 agent
- **`category`**：按类型分组提示词（feature-with-constraint、workflow-description、clone-with-modification、state-display、data-display、responsive-challenge、io-integration、page-setup、typography、theme-customization）
- **`complexity`**：simple、moderate、complex

## 子 Agent 提示词

每个目标获得等价提示词。唯一变化的是设计系统名称和项目环境。

**所有库目标看到相同结构：**

```
You are generating React/TSX code using <system name>.
Your project is at <project dir>. Explore it to find available components.

## Task

<prompt text>

## Output
...same format...
```

**HTML 目标看到：**

```
You are generating React/TSX code using ONLY plain HTML elements and inline CSS.
Your project is at <project dir>. Do NOT use any component library.
Use standard HTML elements (div, button, input, etc.) with inline styles or a styles object.

## Task

<prompt text>

## Output
...same format...
```

注意：没有系统特定规则，没有预期组件，没有预构建命令。Agent 通过项目自身文件（README、package.json、符号链接源码）自行发现所需内容。

## 检查器协议

**运行或修改任何 vibe 测试前，验证以下 5 条不变量。**

### 1. 公平评估器

- 所有配置使用相同评估逻辑（tsc、评分维度）
- 相同通过/失败标准；不得对某个配置采用不同评分
- 目标感知评分可以接受（不同系统需要不同计数方法），但判断必须等价
- 评估必须对输出由哪个配置产生保持盲态

### 2. 仅被测系统变化

- 所有配置使用相同提示词（相同 ID、相同文本）
- 所有配置使用相同角色设定
- 相同输出格式要求
- 等价子 Agent 指令：若某配置获得"阅读 README"，所有配置都获得类似指令
- 无系统特定指导规则（如"使用 StyleX"或"使用 Tailwind"）；让文档来教

### 3. 绝不泄露答案

- `expectedComponents` 仅用于评估；绝不包含在子 Agent 提示词中
- 不要从预期组件预构建检索命令
- 不要暗示使用哪些组件
- 不要包含仅对某一系统有意义的规则
- Agent 应通过系统自身文档发现正确方法

### 4. 代表性环境

- 子 Agent 应拥有真实项目设置（node_modules、package.json）
- 可用文档/工具应与真实消费者一致
- 若真实用户只能获得 npm 包，不要给子 Agent 访问源码仓库的权限
- 测试实际交付机制（node_modules 中的 README，而非手写技能文档）

### 5. 无上下文子 Agent

- 子 Agent 必须全新启动，无先前对话上下文
- 不继承父会话中关于设计系统的知识
- 子 Agent 的唯一知识来自任务中发现的内容
- 不要跨提示词复用子 Agent 会话；每个提示词获得全新 agent

## 如何验证

任何测试运行前，审查生成的任务提示词：

```bash
# Generate an iteration without running it
node internal/vibe-tests/src/setup-nightly.mjs --sample 3

# Read the task files and compare:
# - Are the prompts identical (same task text)?
# - Does either prompt contain expectedComponents or pre-built commands?
# - Does either prompt contain system-specific coaching rules?
# - Do the doc retrieval instructions point to representative paths?
```

评估公平性方面，检查 `universal-eval.ts`：

- 搜索 `target ===` 找出所有目标感知分支
- 验证每个分支测量的是其系统的等价概念
- 注意是否有目标获得其他目标没有的跳过/加分

## 已知可接受的不对称

这些是有意且已记录的；它们略微偏向基线，使 Astryx 的胜出更可信：

- **效率：** Tailwind 的单行 `className` 比 Astryx 的多行 `stylex.create` 块获得更低的样式比率，尽管前者编码了更多决策
- **可维护性：** Tailwind 的刻度值（`p-4`、`text-sm`）计为语义化，相比 HTML 的原始 `16px` 计数方式更为宽松
- **Astryx+Tailwind 评分：** 混合目标同时统计 Astryx props 和 Tailwind 类的样式决策。这可能相对纯 Astryx 抬高其决策计数，但准确反映了代码的实际样式表面积

## 执行溯源

运行器可在每个结果旁写入可选的 `<promptId>.provenance.json` 附属文件。它记录版本化、执行器中立的任务、fixture、条件、执行器、时序和 token 元数据，不嵌入提示词或文件系统路径。收集过程复制该附属文件，通用聚合保留每次运行的 harness/模型标签，支持溯源过滤，并明确标记实测、推导、估算、完整和不完整成本数据。

Schema、示例、回退规则和运行器迁移契约见 [`docs/execution-provenance.md`](docs/execution-provenance.md)。

## 目录结构

```
internal/vibe-tests/
├── test-sets/           # Prompt batteries (JSON)
├── src/                 # Runner scripts, evaluation, and fixture guards
│   ├── setup-nightly.mjs     # 4-target nightly setup
│   ├── universal-eval.ts     # Static analysis scoring (5 dimensions)
│   ├── universal-aggregate.ts # Score aggregation
│   ├── universal-compare.ts  # Cross-target comparison
│   ├── build-previews.ts     # TSX → HTML compilation + tsc
│   ├── screenshot-previews.ts # Playwright screenshots
│   ├── build-report.ts       # Vite HTML report
│   └── deploy-report.ts      # gh-pages deployment
├── fixtures/            # Immutable standalone consumer-app fixtures
├── fixture-recipes/     # Pinned provenance and SHA-256 manifests
├── scripts/             # Fixture setup plus report helpers
├── .baseline/           # Real shadcn/ui components for baseline tsc
├── setup-test/          # Setup evaluation over the canonical fixture matrix
│   ├── PLAN.md               # controls, pilot stages, measures, decision rule
│   ├── matrix.json           # fixtures × controls × prompts × bundles × reps
│   ├── conditions.json       # controls plus the opt-in established-app strategies
│   ├── guidance/             # one document per patch, including both strategies
│   ├── run-setup.mjs         # sandbox + task + provenance preparation only
│   ├── setup-interactions.mjs # marker-driven dialog and nested-overlay opener
│   ├── setup-measure.mjs     # build + exact style, geometry, and overlay probes
│   ├── setup-workspace.mjs   # copy-on-write build root; the sandbox is never written to
│   ├── setup-integrity.mjs   # read-only diff attestation and escape-hatch checks
│   ├── setup-important.mjs   # syntactic `!important` detection for that checker
│   ├── setup-eval.ts         # strict deterministic deltas against each fixture
│   └── setup-aggregate.ts    # matrix coverage, A/B deltas, and final acceptance
├── results/             # Iteration results (gitignored)
└── README.md            # This file
```

## 规范 fixture 套件

`fixtures/` 下的三个应用分别代表：纯 Tailwind v4 对照组、成熟的 shadcn 风格 Tailwind v4 应用，以及带有标记的访客设计系统边界的原创企业风格应用。两个成熟应用 fixture 包含受控的 portal 对话框和嵌套 tooltip/menu 表面，以便 setup 测量裁剪、遮挡和层序。它们是独立包，具有精确依赖和锁文件；均未安装 Astryx。

将规范 fixture 文件视为不可变输入。Setup 验证其固定的 recipe 和 SHA-256 清单，然后将列出的文件复制到沙箱，不写入规范目录：

```bash
pnpm -F @astryxdesign/vibe-tests fixtures:verify
pnpm -F @astryxdesign/vibe-tests fixtures:prepare -- tailwind-v4-control --output /tmp/tailwind-control-run
pnpm -F @astryxdesign/vibe-tests fixtures:build
```

有意刷新 fixture 时，更新其固定 recipe、重新生成锁文件、审查编写的补丁，并刷新记录的哈希：

```bash
pnpm -F @astryxdesign/vibe-tests fixtures:refresh -- tailwind-v4-control
```

根仓库守卫验证溯源、精确清单、确定性源码、依赖隔离、工作区排除，以及与每个可发布包根目录的隔离。这使未来的 setup 或迁移测试独立于任何未落地的 harness 分支。

生成的公开产物绝不保留机器本地路径。Setup 配置使用相对于输出目录的路径；测量标识 fixture 并从构建诊断中隐去私有路径或主机；导出的溯源使用通用 usage-source 标签。Setup 和通用聚合在输出仍包含绝对路径或私有主机的报告前会失败关闭。