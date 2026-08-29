# 上游同步手册

目标是在不覆盖个人定制和未提交工作的前提下，将课程作者更新作为一项可审阅、可测试、可回退的变更引入 fork。

## 自动流程

`.github/workflows/upstream-sync.yml` 每周一 02:17 UTC 检查一次，也支持在 GitHub Actions 页面手动触发。工作流会：

1. 只抓取 `origin/main` 与 `upstream/main`；
2. 如果没有新提交，直接结束；
3. 从当前 `origin/main` 重建专用分支 `automation/upstream-sync`；
4. 尝试合并 `upstream/main`；
5. 无冲突时更新专用分支并创建或更新 PR；
6. 有冲突时停止并将工作流标记为失败。

工作流不会执行合并后的项目代码，不会自动合并 PR，也不会 force-push `main`。`--force-with-lease` 只用于完全由自动化管理的 `automation/upstream-sync` 分支。

`upstream` 仅用于读取作者更新，其 push URL 被显式设为 `DISABLED`。所有自动化写入只允许发往个人 fork `origin`，不会修改课程作者仓库。

仓库需要在 **Settings → Actions → General → Workflow permissions** 中允许 GitHub Actions 创建 Pull Request。工作流本身只申请 `contents: write` 和 `pull-requests: write`。

## 本地手动流程

先检查工作区和分叉关系：

```bash
git status --short --branch
git fetch --all --prune
git rev-list --left-right --count main...origin/main
git rev-list --left-right --count main...upstream/main
```

工作区干净时创建同步分支：

```bash
git switch main
git pull --ff-only origin main
git switch -c chore/sync-upstream-YYYYMMDD
git merge --no-edit upstream/main
make verify
make check
git push -u origin chore/sync-upstream-YYYYMMDD
gh pr create --base main
```

若存在未提交工作，先确认它是否为需要保留的源码、可公开快照或纯运行时产物。不要用 `reset --hard`、`clean -fd` 或 `gh repo sync --force` 处理不明改动。

## 冲突处理

1. 先从冲突 PR 建立本地分支，不直接在 `main` 上处理。
2. 对照 `git diff --ours`、`git diff --theirs` 和产品验收目标逐个文件决策。
3. 上游接口变化优先由个人适配层兼容；不能兼容时记录 ADR。
4. 解决后运行 `make verify`；仓库级变化运行 `make check`。
5. 在 PR 中写清保留了哪一侧、原因、测试证据和仍未验证的边界。

可在本机启用 Git 的冲突解决记忆：

```bash
git config rerere.enabled true
git config fetch.prune true
```

## 同步记录

| 日期 | 合入前基线 | 上游目标 | 结果 | 验收 |
|---|---|---|---|---|
| 2026-08-29 | `e6731d8` | `a6cc399` | 7 个提交可 fast-forward；通过同步分支交付 | 见对应 PR/CI |
