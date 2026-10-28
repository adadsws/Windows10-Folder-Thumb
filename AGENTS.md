# 项目代理规则

- 每个当前目标在 `docs/plans/` 最多保留一份 Plan；不同目标可分别保留活动 Plan。Plan 文件按 `<日期>-<目标标识>.md` 命名，同一目标持续使用同一标识。同一目标的任务规范、补充规范、需求及其补充、澄清、变更、样例、验收标准和验证记录都更新到该 Plan，不在 `docs/superpowers/specs/` 或其他路径另建任务规格、设计或验收文档。完成后将该 Plan 移到 `docs/finished_plans/`；历史资料按文件管理规则归档。
- 修改运行入口、脚本或仓库治理文件后，执行 [AGENT_CONTEXT.md](AGENT_CONTEXT.md) 列出的全部验证命令。
- 自动化测试不得真实终止或启动 Explorer，不得修改用户文件夹图标；入口测试只允许在 `~temp/` 使用替身进程。
- 回收站测试必须向 `Move-FileToRecycleBin` 注入隔离的 `RecycleAction`，不得操作真实 Windows 回收站。
- `run.bat` 是根入口，运行源码位于 `scripts/set_folder_thumb.ps1`，配置固定从根目录 `config.ini` 读取。
- `reference/` 仅保存 `.gitmodules` 锁定的上游 submodule；更新前必须确认并同步 `reference/README.md`。
- 当前源仓库历史含 `~archive/` 等本地专用路径且与远程分叉，不得直接 push。收到明确发布请求后，按全局规则从 `~temp/github-export/` 创建脱敏导出仓库。
