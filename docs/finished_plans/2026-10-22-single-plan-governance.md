# 单一 Plan 与任务资料归并计划

## 目标与已核实状态

- 用户要求当前项目满足全局 `AGENTS.md`：同一目标每次仅有一个当前 Plan，任务规范、补充规范、需求及其补充、澄清、变更、样例、验收标准和验证记录统一写入该 Plan。
- 当前 `docs/plans/` 无文件；`docs/superpowers/specs/` 有四份已完成任务的独立设计文档，其实施计划已在 `docs/finished_plans/`。当前分支为 `codex/single-plan-governance`，任务开始时工作树干净。
- 原设计文档是历史资料；并入对应完成计划后按文件管理规则归档原件，不物理删除。

## 范围与方案

- 在项目 `AGENTS.md` 固化单一当前 Plan 的具体规则；`AGENT_CONTEXT.md` 准确说明当前与已完成计划目录。
- 将四份旧设计文档的完整内容并入对应完成计划，保持历史细节及验收条件；原文件移动到 `~archive/<时间>-single-plan-governance/docs/superpowers/specs/`。
- 合规检查识别 `docs/plans/` 多于一个 Markdown Plan，以及旧 `docs/superpowers/specs/` 中仍有独立规格文件；测试覆盖正常与失败状态。
- 更新 `CHANGELOG.md`，完成后将本 Plan 移到 `docs/finished_plans/`，做本地 commit，不执行远程发布。

## 需求样例与验收标准

| 输入或状态 | 预期行为 | 预期输出 |
|---|---|---|
| `docs/plans/` 恰有本任务 Plan | 合规检查接受 | 退出码 0 |
| `docs/plans/` 出现第二个 Markdown Plan | 合规检查拒绝 | 明确指出计划数量超限 |
| 旧 `docs/superpowers/specs/` 仍有独立设计文档 | 合规检查拒绝 | 指出待归并文件 |
| 四份历史设计内容已归并、原件已归档 | 对应完成计划包含原设计全文，原路径无文件，归档可恢复 | 路径与内容核对通过 |
| 当前任务完成 | 活动 Plan 移入 `docs/finished_plans/` | `docs/plans/` 无当前 Plan；全部规定验证命令通过 |

## 实施清单

- [x] 归并四份历史设计文档，归档原件并核对内容。
- [x] 更新项目规则、上下文、变更日志和合规检查。
- [x] 执行 `AGENT_CONTEXT.md` 的全部验证命令及本任务负例检查，修复失败。
- [x] 完成并归档本 Plan，重新验证，提交本任务修改并核对工作树。

## 验证记录

- 四份归档原件与 Git `HEAD` 原文一致，四份对应完成计划均包含原设计全文；旧 `docs/superpowers/specs/` 无 Markdown 文件，活动 Plan 数为 1。
- `Test-RepositoryComplianceHarness.ps1` 已验证单份 Plan 接受、两份 Plan 拒绝、独立规格文件拒绝。
- 首轮夹具因 PowerShell 错误文字换行导致断言误判，已规范化空白；`Test-PrePushGuard.ps1` 初次等待无限标准输入，已为路径检查传入有限空输入。
- `AGENT_CONTEXT.md` 列出的六条命令于 2026-10-22 全部返回成功；`git diff --cached --check` 无输出。
- 最终归档、归档后的验证及本地提交见本任务最终 Git 状态。

## 人工介入

- 无已知必需人工介入。本任务不推送、不操作用户文件夹或真实 Explorer。
