# 代理上下文

## 技术架构

项目是面向 Windows PowerShell 5.1 的交互式脚本工具。根目录 `run.bat` 负责设置 UTF-8 code page 并启动 `scripts/set_folder_thumb.ps1`；脚本读取根目录 `config.ini`，用 `System.Drawing` 生成 256×256 ICO，并通过 `desktop.ini` 配置文件夹图标。`scripts/file_cleanup.ps1` 使用 `Microsoft.VisualBasic.FileIO` 实现恢复模式的回收站操作，并保留独立的永久清理函数供缩略图重新生成使用。

用户安装和操作方法见 [README.md](../README.md)。

## 项目结构

- `scripts/`：运行源码和文件清理边界。
- `tests/`：仓库合规、启动入口和 pre-push 守卫测试。
- `docs/plans/`：各当前目标的唯一 Plan，包含所属任务的规范、需求、样例、验收标准与验证记录；不同目标可并行保留 Plan。
- `docs/finished_plans/`：已完成目标的 Plan。
- `reference/`：不参与运行的固定上游 submodule 与重建说明。
- `~archive/`：只在本地 Git 保留、不得远程发布的旧文件。
- `~outputs-intermediate/`：可重建、供后续检查或复用的中间输出，不纳入 Git。
- `~outputs-final/`：可重建、供审计或交付的最终输出，不纳入 Git。
- `~temp/`：可随时重建的临时文件，不纳入 Git。

## 开发与验证

从项目根目录运行：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-FileCleanup.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-Launcher.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-RepositoryComplianceHarness.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-RepositoryCompliance.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-PrePushGuard.ps1
powershell -NoProfile -Command "[void][scriptblock]::Create((Get-Content -LiteralPath 'scripts/set_folder_thumb.ps1' -Raw -Encoding UTF8))"
```

测试必须隔离 Explorer 和用户文件夹副作用。维护规则和发布边界见 [AGENTS.md](AGENTS.md)。

## 评测集与验收

以下样例同时检查输入、执行过程和结果。自动化测试的证据是命令退出码及其输出；涉及实际 Explorer 的人工体验不由自动化测试替代。

| 来源 | 输入与前置状态 | 预期行为（过程） | 预期输出（结果） | 验证证据 |
|---|---|---|---|---|
| 真实：用户运行入口 | `run.bat` 与根目录 `config.ini`，选择退出 | 入口以 UTF-8 启动脚本，不触碰用户文件夹 | 正常退出 | `Test-Launcher.ps1` 的替身进程测试 |
| 真实：恢复模式 | 含生成图标和 `desktop.ini` 的隔离夹具 | 注入 `RecycleAction`，不访问真实回收站 | 可回收项传给注入动作，失败项保留并警告 | `Test-FileCleanup.ps1` |
| 预测：文档位置回归 | 将 `AGENT_CONTEXT.md` 放在根目录的隔离仓库 | 合规检查拒绝旧位置，不修改实际仓库文件 | 非零退出码及明确的 `docs/` 位置提示 | `Test-RepositoryComplianceHarness.ps1` 的旧位置样例 |
| 真实：仓库治理 | 根目录 README、`docs/` 内规范文档及受控 Git 文件 | 检查文件跟踪、目录、Plan 与发布边界 | 合规检查通过，违规夹具被拒绝 | 合规检查及 Harness、pre-push 测试 |

交付时逐项核对上述证据；未覆盖的真实 Explorer 视觉刷新效果明确列为未验证，不宣称自动化通过。

## 已知问题

- 实际运行会终止并重启 Explorer，未保存的 Explorer 窗口状态可能丢失。
- 图标刷新依赖 Windows Explorer、`desktop.ini` 和文件系统属性，在非 Windows 环境不可用。
- 网络共享等位置可能不支持 Windows 回收站；恢复模式会警告并保留无法回收的文件。
- 当前 `main` 与 `origin/main` 已分叉；源仓库包含本地专用历史，不能直接 push。
