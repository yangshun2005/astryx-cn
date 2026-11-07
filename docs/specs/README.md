# 系统规格

本目录存放重大共享系统变更的记录。每个规格位于 `docs/specs/<id>-<slug>/spec.md`；可选的同级 `plan.md` 仅用于多步骤实现。

组件和模块契约位于 `docs/specs/components/`，每个组件一个文件夹，核心记录与各平台的记录并列存放（`components/Avatar/Avatar.spec.md` 和 `components/Avatar/Avatar.swift.spec.md`）。目录布局见 [`docs/README.md`](../README.md)。

模板单独存放在 `docs/templates/knowledge/` 下。现有记录依据 `docs/schemas/knowledge/` 校验；修改模板不会悄悄改变已接受的历史。

当工作涉及多个组件、共享基础组件、架构、公共 API 策略、主题、无障碍策略、分发或生命周期时，适合编写系统规格。恢复现有契约的针对性修复无需新规格。

只有 `authority: current` 的记录具有权威性。草稿记录可能带有未解决的证据或负责人决策；它们不主导评审。已归档记录需说明其不再主导的原因，并在存在替代记录时链接到该记录。首次提升为 `current` 需负责人明确批准。