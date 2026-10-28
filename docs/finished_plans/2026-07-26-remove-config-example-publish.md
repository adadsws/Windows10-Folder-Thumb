# Remove Duplicate Config Example Publish Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Archive the semantically duplicate local `config.ini.example`, remove it from the published repository, and keep `config.ini` as the sole usable configuration file.

**Architecture:** Move the source file intact into the local-only archive and record the change in the changelog. In the sanitized export, remove only the root `config.ini.example`, publish the changelog update through a fast-forward `main` push, verify the remote state, then archive this plan.

**Tech Stack:** INI, Markdown, Windows PowerShell 5.1, Git

## Global Constraints

- Work directly on `main` as previously authorized.
- Never delete the source file; move it intact to `~archived/config.ini.example`.
- Keep `config.ini` unchanged and published.
- Keep historical CHANGELOG entries; add a new current entry.
- Do not alter `secrets/**/*.example` rules or tests.
- Commit as `adadsws <1410990013@qq.com>`.
- Push only through `output/github-export/` to remote `main`, fast-forward and without force or PR.
- Never publish `~archived/`, `~ref/`, active plans, or private secrets.
- Archive this completed plan under `~archived/superpowers-plans/`.

---

### Task 1: Archive the semantically duplicate source file

**Files:**
- Move: `config.ini.example`
- To: `~archived/config.ini.example`
- Modify: `CHANGELOG.md`

**Interfaces:**
- Consumes: Two root configuration files with identical normalized text and different line endings.
- Produces: One published root configuration and one intact local archive.

- [x] **Step 1: Verify normalized configuration text**

Run:

```powershell
$exampleHash = (Get-FileHash -LiteralPath config.ini.example -Algorithm SHA256).Hash
$exampleHash
$configText = (Get-Content -LiteralPath config.ini -Raw -Encoding UTF8) -replace "`r`n", "`n"
$exampleText = (Get-Content -LiteralPath config.ini.example -Raw -Encoding UTF8) -replace "`r`n", "`n"
$configText -ceq $exampleText
```

Expected: normalized text comparison is `True`. Preserve `$exampleHash` for the post-move byte-integrity check; the original byte hashes differ because one file uses CRLF and the other LF.

- [x] **Step 2: Move the example into the local archive**

Resolve and verify the exact source and destination paths, confirm the destination does not exist, then run:

```powershell
Move-Item -LiteralPath config.ini.example -Destination ~archived/config.ini.example
```

Recompute the archive hash and confirm it equals `$exampleHash`.

- [x] **Step 3: Update CHANGELOG**

Add this bullet at the top of `## 2026-07-26` → `### 仓库治理`:

```markdown
- 将与 `config.ini` 内容重复的 `config.ini.example` 移入本地归档，并从发布仓库移除。
```

- [x] **Step 4: Stage, verify, and commit source changes**

Run:

```powershell
git add -A -- config.ini.example ~archived/config.ini.example CHANGELOG.md docs/superpowers/plans/2026-07-26-remove-config-example-publish.md
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-RepositoryComplianceHarness.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-RepositoryCompliance.ps1
'' | powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-PrePushGuard.ps1
git diff --cached --check
git commit -m "chore: archive duplicate config example"
```

Expected: the move, changelog, and plan progress are committed; checks pass.

### Task 2: Remove the example from the sanitized export

**Files:**
- Delete from export: `output/github-export/config.ini.example`
- Modify: `output/github-export/CHANGELOG.md`

**Interfaces:**
- Consumes: Verified source archive and changelog.
- Produces: A sanitized export with only `config.ini` at the root.

- [x] **Step 1: Synchronize the export**

Run:

```powershell
git -C output/github-export fetch origin main
git -C output/github-export pull --ff-only origin main
```

Expected: export starts clean at remote `main`.

- [x] **Step 2: Copy changelog and remove the exact export file**

Verify `output/github-export/config.ini.example` resolves inside the export root, then run:

```powershell
Copy-Item -LiteralPath CHANGELOG.md -Destination output/github-export/CHANGELOG.md
git -C output/github-export rm -- config.ini.example
```

- [x] **Step 3: Verify export scope and privacy**

Run:

```powershell
git -C output/github-export status --short
git -C output/github-export diff --check
git -C output/github-export diff --name-status
git -C output/github-export ls-files |
    rg '^(~archived/|~ref/|docs/superpowers/plans/|secrets/(?!.*\.example$))' --pcre2
```

Expected: only `CHANGELOG.md` is modified and `config.ini.example` is deleted; blocked-path scan has no matches.

- [x] **Step 4: Commit and verify the export**

Run:

```powershell
git -C output/github-export add -- CHANGELOG.md
git -C output/github-export commit -m "Remove duplicate config example"
powershell -NoProfile -ExecutionPolicy Bypass -File output/github-export/tests/Test-RepositoryComplianceHarness.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File output/github-export/tests/Test-RepositoryCompliance.ps1 -RepositoryRoot output/github-export
Push-Location output/github-export
'' | powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-PrePushGuard.ps1
git diff --check
git status --short
Pop-Location
```

Expected: export commit uses the requested identity; all checks pass; export is clean.

### Task 3: Push and verify remote main

**Files:**
- No source file changes.
- Remote target: `origin/main`

**Interfaces:**
- Consumes: One verified export commit.
- Produces: Remote `main` without `config.ini.example`.

- [x] **Step 1: Verify fast-forward scope and dry-run**

Run:

```powershell
git -C output/github-export fetch origin main
git -C output/github-export rev-list --count main..origin/main
git -C output/github-export rev-list --count origin/main..main
git -C output/github-export diff --name-status origin/main..main
git -C output/github-export push --dry-run origin main:main
```

Expected: export is zero behind and one ahead; outgoing changes are changelog modification and example deletion; dry-run passes the actual hook.

- [x] **Step 2: Push main**

Run:

```powershell
git -C output/github-export push origin main:main
```

Expected: remote `main` advances by fast-forward.

- [x] **Step 3: Verify source, export, and remote**

Run:

```powershell
git -C output/github-export fetch origin main
git -C output/github-export rev-parse main
git -C output/github-export rev-parse origin/main
git -C output/github-export cat-file -e origin/main:config.ini
git -C output/github-export cat-file -e origin/main:config.ini.example
git -C output/github-export ls-remote --heads origin
git branch --format='%(refname:short)'
git -C output/github-export branch --format='%(refname:short)'
```

Expected: export and remote hashes match; remote `config.ini` exists; `config.ini.example` lookup fails; source archive exists and is tracked; source, export, and remote expose only `main`.

### Task 4: Archive the completed plan

**Files:**
- Move: `docs/superpowers/plans/2026-07-26-remove-config-example-publish.md`
- To: `~archived/superpowers-plans/2026-07-26-remove-config-example-publish.md`

**Interfaces:**
- Consumes: Verified remote publication.
- Produces: A clean source with no completed active plan.

- [x] **Step 1: Complete and move the plan**

Mark every task-step checkbox `[x]`, then move the plan to:

```text
~archived/superpowers-plans/2026-07-26-remove-config-example-publish.md
```

- [x] **Step 2: Stage, verify, and commit the archive**

Run:

```powershell
git add -A -- docs/superpowers/plans/2026-07-26-remove-config-example-publish.md ~archived/superpowers-plans/2026-07-26-remove-config-example-publish.md
powershell -NoProfile -ExecutionPolicy Bypass -File tests/Test-RepositoryCompliance.ps1
git diff --cached --check
git commit -m "chore: archive config example removal plan"
git status --short
```

Expected: checks pass, archive commit succeeds, and source is clean.

---

## 原设计内容（归并于 2026-10-22）

# 移除重复配置示例设计

## 目标

移除与 `config.ini` 配置文本相同、仅换行编码不同的 `config.ini.example`，让用户只维护和使用一个不含隐私值、可直接运行的配置文件。

## 源仓库处理

- 将 `config.ini.example` 原样移动到 `~archived/config.ini.example`。
- 不删除文件内容，遵守“归档而不删除”的本地规则。
- `~archived/config.ini.example` 继续由本地 Git 跟踪，但不得 push。
- 根目录只保留 `config.ini`；其现有内容使用通用注释示例，不含真实用户路径。

## 发布仓库处理

- 从脱敏导出仓库的根目录删除 `config.ini.example`。
- 保留并发布 `config.ini`，使其成为唯一配置模板和实际运行配置。
- 使用独立导出仓库从远程 `main` fast-forward push，不 force push，不创建分支或 PR。

## 文档处理

- README 当前只引用 `config.ini`，无需修改。
- 在 `CHANGELOG.md` 当前日期下新增说明，记录重复的 `config.ini.example` 已归档并从发布仓库移除。
- 保留 CHANGELOG 中“曾新增 `config.ini.example`”的旧记录，因为它描述的是历史事件。

## 不受影响的内容

- `secrets/**/*.example` 规则保持不变；该规则只处理真实隐私文件的脱敏示例。
- 不修改 `set_folder_thumb.ps1`、`run.bat`、测试或 `config.ini` 内容。
- 不修改其他 `.example` 相关测试与仓库治理规则。

## 验证

- 规范化 CRLF/LF 后确认 `config.ini` 与 `config.ini.example` 文本一致。
- 确认归档文件与移动前的 `config.ini.example` 字节哈希一致。
- 确认源仓库根目录和远程 `main` 均不存在 `config.ini.example`。
- 确认源仓库归档路径存在并由本地 Git 跟踪，导出历史不包含该归档路径。
- 运行源仓库与导出仓库的合规检查和 pre-push 守卫。
- push 后确认源仓库、导出仓库和远程仍仅有 `main`。
