# Astryx 知识地图

本目录存放维护者知识，这些知识必须能与其所管辖的代码一同接受审查。面向使用者的文档仍保留在组件的 `.doc.mjs` 文件和 `packages/cli/assets/docs/` 中。

## 放置规则

- `architecture/`：已交付的系统、其边界与不变量。
- `contributing/`：实用的贡献者工作流，用于呈现当前所有者记录，但不替代这些记录。
- `design/`：由人维护的视觉与交互规范。
- `themes/`：包内主题规范的指引与索引。
- `families/`：同级组件共享的契约。
- `specs/`：重要的已提议或已交付的系统变更及其决策。
- `specs/components/`：组件与模块契约，每个组件一个文件夹。
- `templates/knowledge/`：编写模板。模板绝不与记录混放。
- `schemas/knowledge/`：模板与记录的结构要求，带版本号。

每个组件（无论属于哪个包、哪个平台）的记录都集中在一个文件夹中：`specs/components/<ComponentRoot>/`，以存放该组件代码的文件夹命名。该目录树镜像 `scripts/component-packages.cjs` 注册的包的源码树：

- 目录布局的包（`core`、`lab`）保留 `packages/<package>/src/<ComponentRoot>/`，其记录为 `specs/components/<ComponentRoot>/<PublicName>.spec.md`；
- 扁平布局的包（`charts`、`richtext`、`vega`）保留 `packages/<package>/src/<PublicName>.tsx`，其记录为 `specs/components/<PublicName>/<PublicName>.spec.md`；
- 平台实现的记录与核心记录并列，命名为 `specs/components/<ComponentRoot>/<PublicName>.<language>.spec.md`，例如 `specs/components/Avatar/Avatar.swift.spec.md`。

`<PublicName>` 通常与根名一致；像 `NavMenu/NavHeadingMenu.spec.md` 这样的公开成员，仅当该根中存在一个精确的顶层或完整内联使用者文档条目声明了相同公开名时才有效。在扁平包中，公开具名导出及对应的 TSX 模块定义组件边界。当两个包保留同名文件夹时，记录归属于存放其代码的包。

可独立契约化的公开 Hook、插件、工具和子系统使用 `kind: module` 记录，命名为 `<PublicName>.spec.md`，位于同一组件文件夹下至少一层目录，通常在 `modules/`（`specs/components/Dialog/modules/DialogHeader.spec.md`）或插件自己的文件夹（`specs/components/Table/plugins/rowStatus/useTableRowStatus.spec.md`）。子目录无需与模块代码所在位置一致；查找通过代码文件名或其 `verified_by` 证据到达记录。其规范 id 为 `module:<ParentComponent>/<PublicName>`；模块的 `parent_component` 与父组件的 `modules` 列表必须一致。私有转换辅助函数无需记录。

组件记录发现与 PR 路由会忽略隐藏路径段、`*.generated.spec.md`，以及 fixture、test、generated、build-output、coverage 和 `node_modules` 目录。这些文件不是知识记录，也不是仅规范变更。主题契约与包并列存放于 `packages/themes/<theme>/<theme>.spec.md`。

## 权威性

每条知识记录都声明 `authority: draft | current | archived`。

- `draft` 文档不是政策；未解决的证据和所有者决策作为阻塞项记录在文档内部。
- 初次提升为 `current` 需要文档元数据中记录明确的所有者批准。任何当前 `.github/ENGOWNERS` 成员可批准所有记录类型；当前设计与主题记录及规范性设计资产还接受当前 `.github/DESIGNOWNERS` 成员批准。旧版 v1 `owners` 元数据仅为描述性，不授予批准权。
- 创建、更改或归档 `current` 记录的 Pull Request 会等待审查门禁 `review-required`，直到所有者通过 `/approve-spec <full-sha>` 或审查批准其内容（`architecture:knowledge-contracts` DEC-14）。仅含草稿的记录无需所有者批准。
- 只有 `current` 文档指导实现与审查。
- 规范描述持久的理想行为，独立于任何单个 Pull Request。Pull Request 和 Issue 只能作为明确非权威的示例、历史证据或引用出现。规范绝不用于批准、拒绝、分类、指定或授权某个特定 Pull Request。
- 创建或实质性扩展记录前，按规范所有者/id、受影响路径与导出符号、语义行为术语搜索当前记录和打开的 Pull Request。默认扩展现有规范所有者或向其投影；仅为不同的事实边界创建新记录，并说明原因。
- 当前记录仅依赖其他当前记录。`modules` 和 `parent_component` 是结构性所有权链接，因此可连接活跃的草稿/当前记录，而不使草稿行为具有权威性。其他草稿关系列在候选记录上；提升时在所有者审查下更新受影响的反向链接。
- `archived` 文档声明 `archive_reason`（`superseded`、`withdrawn` 或 `historical`），并在存在替代时链接 `superseded_by`。

## 模板与 Schema

模板创建文档；Schema 保持现有文档结构对齐。二者有意分离：

- `template_version` 记录创建文档所用的编写形式。
- `schema_version` 记录文档必须满足的契约。
- 编辑性模板变更只能递增 `template_version`。
- 添加或更改必需元数据或章节会创建新的 `schema_version`；已发布的 Schema 文件不可变。
- 每条活跃记录（`draft` 或 `current`）必须使用定义其类型的最新 Schema。添加新类型不会迁移无关记录；结构性更改现有类型仅迁移该类型的活跃记录。已归档记录可保留仍签入的旧 Schema。

已发布的 Schema 版本可扩展更早的 Schema。检查器组合链并跟踪定义每种记录类型的最新版本。因此新类型是增量的；更改现有类型仅迁移该类型。

可选的 `review-applicability:v1` JSON 块可将记录中选定的主张路由为跨领域审查基线。这是独立版本化的元数据叠加层，类似 `anatomy-theming:v1`，因此添加它不会改变记录类型的必需形状，也不会迁移未选择加入的记录。该块必须声明 `scope: "global"`，并将每个语义触发器映射到同一记录中一个或多个精确主张 id。验证会拒绝格式错误、空、重复或过时的主张路由。路由需要被审查的 head，验证提供的基线权威提交是其 `origin/main` 合并基，仅包含 `authority: current` 记录，并仅返回匹配的主张；草稿记录和不匹配的路由会被省略。

可选的 `path-applicability:v1` JSON 块列出当放置、路径字段、名称或继承链接均无法到达时记录所管辖的仓库路径：`{"paths": ["apps/docsite/**"]}`。模式为仓库相对 glob：`**` 是跨任意深度的整段，`*` 在单段内匹配，`?` 匹配一个字符。与 `review-applicability:v1` 一样，它是独立版本化的叠加层，因此选择加入不会迁移记录类型。模式必须以字面路径段开头。验证会拒绝格式错误、不安全、重复、全仓库和无匹配的模式。匹配仅为路径定位记录；主张范围和优先级不变。当当前记录无法从任何非记录本身的受跟踪路径到达时，`pnpm check:knowledge` 也会失败（`architecture:knowledge-contracts` INV25 和 INV26）。

新记录类型应优先使用一个类型化关系列表而非并行字段。若 `system-spec` 获得未来 Schema 修订，用包含 `architecture:*`、`family:*`、`theme:*` 和 `contributing:*` 等 id 的单个类型化 `affects` 列表替换其旧版 `affects_*` 字段；本 PR 不改变 v1 system-spec 形状。

运行 `pnpm specs:for <path>` 列出链接到任意仓库路径的当前记录：记录自身放置、`applies_to` 和 `verified_by` 条目、family 与 design 成员关系、`affects_consumer_docs` id、文件文本中引用的 system-spec 编号（不在记录路径内），以及这些记录继承的引用。仅跟随可见记录，因此除非传入 `--include-draft`，草稿绝不会添加链接。限定于单个主张或决策的引用（`spec:AST-NNN/FR3`）列出该主张，而非整个记录。决定链接记录的规范单独列出，匹配的组件和模块触发器路由的全局主张也单独列出（与 `review-global-baselines.mjs` 相同的选择）。使用 `--diff origin/main`（或 `--diff <base>..<head>`）时，它读取变更，在未指定文件时列出其文件，并将新增行匹配其所属记录 `concerns` 的主张排在前面。默认视图列出主张和链接的记录 id；`--records` 显示每条记录如何链接。无链接记录的路径会如实报告。记录从工作树读取，因此在 Pull Request 分支上，结果包含该分支提议的记录，不是审查权威。`pnpm specs:for --unrouted` 列出无管辖记录的受跟踪文件；`--unreached` 列出无其他路径到达的当前记录。

运行 `pnpm check:knowledge` 验证模板和记录。纯规范记录 Pull Request 不添加 Changesets，因为它们不发布包变更。

## 可见性

本仓库及其 wiki 是公开的。内部操作、凭据、服务、路由、时间表和私有恢复流程保留在访问受控的私有位置，不得在此命名或链接。