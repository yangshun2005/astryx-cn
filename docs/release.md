# 使用可信发布发布到 npm

公开的[发布流程](https://github.com/meta/astryx/wiki/Release-Process)是操作权威。本页说明仓库和 npm 机制。

Astryx 通过 npm 可信发布（OIDC）从 GitHub Actions 发布公开的 `@astryxdesign/*` 包。CI 不携带长期有效的 npm 发布令牌。每个稳定包都从不可变的发布标签构建，并带来源证明发布。

## 发布顺序

1. 在当前 `main` 的精确提交上运行规范的全范围 `release-check`。该提交还需要 Main 工作流的一次完整单元测试运行通过：main 的调度器会在 main 顶端每次 release-check 时派发一次（包括个人触发的），约 15 分钟内完成，前提是该提交仍是顶端。发布负责人也可以在自己的检查旁派发 Main 工作流，将 `sha` 设为被检查的提交，从而精确测试该提交；不带 `sha` 的派发测试的是启动时 main 的顶端，不构成凭据。只能从同时具备两者的 SHA 切出，且该 SHA 仍在当前 `main` 历史中；稳定发布拒绝缺少两者的切出提交。次版本切出还要求发布负责人 Cindy 在创建分支前立即对确切的 `vX.Y.0` 明确签字确认；计划或常设授权永远不能替代该签字。
2. 创建唯一的标记分支 `release/vX.Y.Z`。其版本是切出提交声明的固定组版本。标记和计划冻结切出 SHA、Changeset 路径和哈希、分支身份以及计划摘要。
3. 在该分支上运行 `pnpm version-packages`。它会按声明的层级接纳冻结的 Changesets，在已声明的版本下写入变更日志，提升暂存的 codemod，刷新精确的内部固定版本，并只删除冻结的 Changesets。它从不选择或提升版本。在 Version Packages PR 合并前刷新并验证锁文件。
4. 在生成的 PR 合并前通过公开的编辑审核门槛，这样失败的标准在该 PR 上修复，而不是事后移动发布头。对于补丁版本，按公开的[博客评审标准](https://github.com/meta/astryx/wiki/Blog-Review-Rubric#three-required-scored-criteria)在 `update` 配置下自评完整的生成变更日志和 GitHub Release 说明草稿，在 PR 上记录结果，并修复和重新评分每个失败的标准。次版本要求 Cindy 对变更日志、发布说明和必需的公开博客进行明确的人工编辑批准，并以相同方式记录。然后将生成的 PR 合并到发布分支，运行精确头部分支 CI，并对该精确分支头打标签并发布。拒绝从 `main` 进行稳定发布。
5. 通过 `chore/merge-vX.Y.Z-to-main` 将已发布分支的输出压缩合并回当前 `main`。可信验证器要求不可变标签、活动分支头和计划摘要；精确的生成字节和冻结的删除；以及保留每个切出后的 Changeset、提交、无关字段和 main 的当前版本声明。分支生命周期文件永远不会落到 `main` 上。
6. 在单独的负责人 PR 中，跨固定组版本、精确内部固定版本和锁文件声明下一个计划。已发布版本的补丁后继是默认值。仅当负责人明确指定时才使用次版本后继。

切出后，main 的声明仍是其未来计划，可以独立于冻结的发布分支变化。必需的拉取请求 lint 不会将其与切出或最新稳定标签比较。Canary 发布拥有该排序边界：它只从高于稳定版的一个普通声明发布，跳过相等，低于稳定版或分裂声明则失败。可信合并回保留 main 的声明；单独的下一计划 PR 在发布后推进它。

## Changesets 和 codemods

功能 PR 使用 `pnpm changeset:new` 添加普通 Changesets。待处理的 Changesets 不选择发布版本。当包为 `0.x` 时，`[breaking]` 工作仅在 main 已声明下一个次版本时才可接纳；补丁声明会拒绝它。在有待处理的 `[breaking]` Changeset 时，将次版本计划降为补丁的 PR 本身会被拒绝。

未发布变更的核心 codemods 暂存在 `packages/cli/assets/codemods/transforms/next/`。在 Version Packages PR 期间，`pnpm version-packages` 在写入变更日志后运行 `scripts/promote-codemod-next.mjs`。它将除 `README.md` 外的每个暂存条目复制到 `transforms/v<declared-version>/`，从 `next` 清除已提升的文件，并更新注册表。随变更日志一起审查这些生成的文件。

## 稳定版和 Canary 发布

两条发布路径都在 `.github/workflows/release.yml` 中，因为 npm 每个包只允许一个可信 GitHub 工作流。

- **稳定版：** 从不可变 `vX.Y.Z` 标签显式派发，附带确切的发布分支、版本、头部、标签和计划摘要。它发布 `latest`。
- **Canary：** 最多每小时从 main 顶端发布，或按需从 `main` 发布（派发时 `channel: canary`）。它仅在声明严格高于最新稳定标签时，将 `<declared-version>-canary.<short-sha>` 发布到 `canary`。它从不授权或改变稳定发布状态。

发布步骤使用 pnpm 原生发布：

```sh
pnpm publish ./packages/<package> \
  --tag latest --provenance --access public --no-git-checks
```

## 可信发布者设置

信任按包配置。它必须指定 `release.yml` 并允许直接 `npm publish`。新公开包必须先存在于 npm 上才能注册信任，因此每个新包一次，npm 组织所有者需在自己的终端中运行（npm 会在浏览器中打开只有所有者能完成的双因素提示）：

```sh
npm i -g npm@latest
npm --version   # 11.15 or later
npm login --registry https://registry.npmjs.org
node scripts/npm/setup-trusted-publishing.mjs
node scripts/npm/setup-trusted-publishing.mjs \
  --bootstrap --setup-trust --workflow release.yml
```

- 默认调用仅审计。
- `--bootstrap` 用已弃用的 `0.0.0-bootstrap.0` 占位符在 `bootstrap` dist-tag 下占用缺失的包名。npm 将包的第一个版本设为 `latest`，因此普通安装该包会解析到空占位符，直到包的首次稳定发布。
- `--setup-trust` 为 `meta/astryx` 注册 `release.yml`，允许直接发布。脚本拒绝低于 11.15 的 npm：旧版 npm 忽略发布权限，创建一个只能暂存发布的发布者，并将每个现有发布者列为冲突。
- 仅在有意替换现有信任记录时使用 `--replace`，绝不要用来清除旧版 npm（低于 11.15）报告的冲突。

然后在 npmjs.com 上打开包的可信发布者设置（`https://www.npmjs.com/package/<name>/access`），确认 `meta/astryx` 的 `release.yml` 条目允许 `npm publish`，而不仅是 `npm stage publish`。同一页面也是脚本无法完成时的手动路径：添加 GitHub Actions 发布者，所有者 `meta`，仓库 `astryx`，工作流 `release.yml`，无环境，并允许 `npm publish`。

npm 会在 48 小时内使未经任何发布验证的新可信发布者过期。下一次 canary（每小时运行，或从 `main` 的 canary 派发）会发布包的 canary 并验证它。如果该窗口内没有 canary 发布，请重新创建发布者。

注册表读取可能在几分钟内持续报告刚创建的包缺失。在该窗口内重新运行 `--bootstrap` 会因占位符版本得到 `409`；名称已被占用，脚本继续信任设置。

仓库固定 pnpm 11。CI 的 Node 行必须捆绑 npm 11.5.1 或更高版本以进行 OIDC 发布；本地信任设置命令要求 npm 11.15 或更高版本。