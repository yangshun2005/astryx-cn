# /internal

不发布到 npm 的内部工具包。

<!-- SYNC: When files in this directory change, update this document. -->

| 目录                   | 角色 | 用途                                                                 |
| ---------------------- | ---- | -------------------------------------------------------------------- |
| `a11y-spec/`           | 测试 | 可复用的无障碍规范测试契约及其证据测试装置                           |
| `interaction-harness/` | 测试 | 用于触摸交互、动画验证和故事板的 Playwright 工具                     |
| `scripts/`             | 工具 | 一次性自动化脚本（Crowdin 截图上传等）                               |
| `test-utils/`          | 测试 | Vitest 的共享测试工具和初始化配置                                    |
| `vibe-tests/`          | 测试 | 组件系统的 LLM 可用性测试装置                                        |