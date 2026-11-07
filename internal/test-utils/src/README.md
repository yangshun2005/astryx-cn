# /internal/test-utils/src

共享测试工具的源代码。

<!-- SYNC: 当此目录中的文件发生变化时，请更新本文档。 -->

| 文件                   | 角色  | 用途                                                                                                                                                               |
| ---------------------- | ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `index.ts`             | 入口 | 从 @testing-library/react 重新导出测试工具                                                                                                              |
| `jsdomRefusedRules.ts` | 配置 | 测试配置阻止进入 jsdom 样式表的规则标记，由 `setup.ts` 和 `stylexInject.ts` 共享                                                     |
| `setup.ts`             | 配置 | Vitest 配置文件；用 jest-dom 扩展匹配器，添加 jsdom polyfill，并阻止分组末成员规则（`:nth-last-child(1 of …)`）进入 jsdom 的样式表   |
| `stylexInject.ts`      | 配置 | vitest.config.ts 指定的 StyleX 运行时注入：包含 StyleX 自身的注入，以及其 0.19.0 开发运行时丢弃的每条规则，原因是该运行时仅以 at-rule 的前导部分作为规则键 |