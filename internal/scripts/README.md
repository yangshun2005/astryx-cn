# /internal/scripts

内部构建时脚本。不随包发布，不属于任何包。

<!-- SYNC: 当此目录中的文件发生变化时，更新本文档。 -->

| 文件                             | 用途                                                                                                                                                                                                                                                                           |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `upload-crowdin-screenshots.mjs` | 构建 storybook，为 i18n 目录键捕获上下文内截图，上传至 Crowdin，并 POST 标签位置，使每张截图按像素链接到匹配的源字符串。                                                                                                    |
| `lib/crowdin-strategies.mjs`     | 声明式测量策略（visibleText、textRun、option、placeholder、ariaLabel、chipOperator、filterInput、footerButton、srOnlyLabel、srOnlyReveal、liveRegionReveal），供上传脚本解析标签坐标。                                             |
| `bottom-sheet-safari-check.mjs`  | 手动真实 Safari 检查（通过 `safaridriver` 使用 W3C WebDriver，无依赖），针对已部署的 Storybook 验证 BottomSheet 堆叠、替换、非模态 popover 承载、toast 顺序和焦点返回；输出 pass/fail/skipped/not-driven 表格并写入 JSON + 截图。 |

Crowdin 在目录中每个字符串旁显示的文件级上下文是文件记录上的一个字段，而非 `crowdin.yml` 可声明的内容；`.github/workflows/crowdin-upload.yml` 中的一个步骤在源上传后设置它。

## upload-crowdin-screenshots.mjs

为 [Crowdin](https://crowdin.com/)（astryx 使用的翻译平台）自动化"带上下文标签的截图"流程。

译者可以看到每个目录键及其在 UI 中出现位置的真实截图。截图上的方框精确标记哪些像素对应哪个键。这大幅减少了"这个字符串是什么意思？"的来回沟通。

### 用法

```bash
CROWDIN_PERSONAL_TOKEN=xxx CROWDIN_PROJECT_ID=nnn \
  node internal/scripts/upload-crowdin-screenshots.mjs [flags]
```

常用标志：

| 标志             | 用途                                                                                             |
| ---------------- | --------------------------------------------------------------------------------------------------- |
| `--dry-run`      | 捕获截图 + 解析标签坐标，但跳过 Crowdin 上传。                             |
| `--validate`     | 验证每个声明的标签键都存在于 `packages/core/locales/en.json` 中，然后退出。                 |
| `--skip-build`   | 复用现有的 `apps/storybook/dist/` 构建。                                                        |
| `--only=<name>`  | 要运行的目标名称，逗号分隔（例如 `--only=dialog-close-button,banner-dismissable`）。         |
| `--replace-tags` | 对现有 Crowdin 截图，在上传新坐标前删除之前 POST 的标签。 |
| `--delete-first` | 在重新上传前删除 Crowdin 上的整个截图记录（id + 标签）。                     |

令牌：个人 API 令牌来自 https://crowdin.com/settings#api-key。项目 ID 是 Crowdin URL 中可见的数字 ID。

### 目标如何声明

每张截图是 `targets` 数组中的一个条目，包含：

```js
{
  name: 'dialog-close-button',
  storyId: 'core-dialog--default',
  viewport: {width: 900, height: 700},
  interact: async page => { /* optional Playwright interaction */ },
  selector: null,  // or CSS selector to crop to
  manualTags: [
    t('astryx.dialog.close', 'ariaLabel', 'Close'),
  ],
}
```

`t(key, strategyName, ...args)` 声明一个标签。策略是一个小函数，根据渲染后的页面返回像素坐标——分类见 `lib/crowdin-strategies.mjs`。出现新的测量模式时，在那里添加新策略。

### 与其他内容共享元素的字符串

`visibleText` 匹配整个文本就是该字符串的元素，因此无法触及 CommandPalette 页脚提示——`<span><Kbd/>Navigate</span>` 读作 `↑↓Navigate`。`textRun` 匹配裸文本节点并用 `Range` 测量它，标记该词而不含键徽章。

### 仅屏幕阅读器可见的标签

某些目录字符串是控件上的 aria-label，这些控件不渲染可见文本（裸复选框、无标签的组合框）。有两种策略处理这种情况：

- `srOnlyLabel`——标签矩形落在可见的祖先控件上。适用于控件有其他识别特征（图标、相邻文本）的情况。
- `srOnlyReveal`——在截图捕获期间在控件旁注入一个小黄色标签气泡，然后标记该气泡。译者看到实际的 aria-label 文本固定在控件上。适用于控件完全没有可见线索的情况。

Reveal 气泡只存在于截图流水线中——它们从不在生产环境中渲染。视口边缘控件旁的气泡会沿该边缘滑动直到能放下。

### Live-region 播报

通过 `useAnnounce()` 播报的字符串落在 `<body>` 上 1×1 的裁剪区域中，并在几秒后自行清除，因此它们没有像素也没有有用的位置。`liveRegionReveal(text, anchorSelector, placement?)` 检查某个区域正在播报 `text`，然后在 `anchorSelector`（播报所涉及的控件）旁显示一个气泡。

时序很重要：截图前遍历（`MUTATING_STRATEGIES` 中的每个策略）必须在文本仍然存活时运行，然后测量遍历读回注入时记录的矩形。因此目标的 `interact` 必须在该窗口内让页面保持在该状态。

### 标签必须适合截图

Crowdin 会拒绝超出上传图像的标签矩形，因此每个策略都要求视口包含候选的矩形。首屏以下的匹配会被报告为未解析，而不是裁剪或滚动到视图中——如果要标记的字符串位于视口外，请扩大目标的 `viewport`。

### 确定性标记，而非 `--auto-tag`

Crowdin 内置的 `--auto-tag` 使用模糊文本匹配，会产生误报（例如标记错误的 "Close"）。此脚本完全确定性——每个声明的标签要么解析到特定的 DOM 元素，要么运行将其报告为未解析。每次调用都会验证目录键。

### 何时重新运行

- 添加了值得向译者展示视觉上下文的新目录键后。
- 组件重新设计改变了已标记字符串的像素位置后。
- 添加了更好地暴露现有键的新 story 后。

`--replace-tags` 干净地处理"只更新像素位置"的情况。

## bottom-sheet-safari-check.mjs

为 BottomSheet 堆叠和替换行为提供真实 Safari 证据，这是 Linux CI（Chromium、Playwright WebKit）无法产生的。在启用了 Safari ▸ 开发 ▸ 允许远程自动化的 Mac 上：

```bash
safaridriver --port 4799 &
node internal/scripts/bottom-sheet-safari-check.mjs <storybook base URL> <label> [out dir]
```

每次运行将 `<label>-results.json`（每个逐帧样本）、`<label>-summary.md` 和截图写入输出目录（标签先为文件名扁平化：`/`、`\`、空格和其他不安全字符变为 `-`，因此分支名可直接使用）（默认 `internal/scripts/output/safari/`，git 忽略），如果某项检查失败则退出码为 1——因此针对当前 `main` 的每次运行都退出 1：其 `show()` 宿主未通过非模态承载行，以及依赖顶层承载的无遮罩行（`focus return: lone no-scrim sheet, Escape`、`replace handoff (no-scrim)`），这正是堆叠变更所填补的缺口——当 Storybook 索引无法读取（基础 URL 错误）时不运行并退出 1。部署缺少的 story 记录为 `skipped`——堆叠和替换 story（`core-bottomsheetstacking--*`、`core-bottomsheetreplace--*`）随开放顺序堆叠变更一起发布，在没有该变更的部署上被跳过，此时切换器和无遮罩检查仍会运行；Safari 的 WebDriver 无法可靠驱动的检查（hover）为 `not-driven`，绝不伪造，前置条件未满足的检查同样如此（sheet 操作没有呈现 sheet；快速 Back 未找到 Back 按钮——在它确实测量到的 1 → 2 交接被评分后，因此失败的交接仍为 `fail`）；当某步骤抛出异常时，该步骤拥有的每项检查都记录为 `fail` 并附带错误，因此表格保持其形状。macOS Safari 证据可提供参考，但不满足 iOS Safari 的声明（规范 AST-013）。