# git-sync-to

安全地将当前 Git 分支同步到目标分支，不切换当前工作目录的分支。

```bash
git sync-to test
```

## 特性

- 允许当前工作区保留未提交修改，仅提示这些修改不参与同步。
- 要求当前分支已经推送，并与远端分支完全一致。
- 如果目标分支已有 worktree，安全复用且不会删除。
- 如果目标分支没有 worktree，自动创建并清理临时 worktree。
- 使用 `pull --ff-only` 更新目标分支，拒绝隐式合并。
- 合并冲突时立即执行 `merge --abort`，不自动解决冲突。
- 不使用强制推送。
- 支持自定义远端和预检查模式。

## 要求

- Git 2.20+
- Bash 3.2+
- macOS 或 Linux

## 安装

下载仓库后，将脚本安装到 PATH：

```bash
install -m 0755 bin/git-sync-to /usr/local/bin/git-sync-to
```

Apple Silicon Mac 通常可以安装到：

```bash
install -m 0755 bin/git-sync-to /opt/homebrew/bin/git-sync-to
```

安装后，Git 会自动将 `git-sync-to` 识别为 `git sync-to` 子命令。

## 使用

```bash
# 将当前分支合并并推送到 origin/test
git sync-to test

# 仅执行远端刷新和安全检查
git sync-to test --dry-run

# 使用其他远端
git sync-to test --remote upstream

# 查看帮助
git sync-to --help
```

当前工作区的已暂存、未暂存修改和未跟踪文件均可保留，脚本不会改动这些内容或暂存区。本次仅同步当前分支已提交且已推送的内容；如果有未推送的提交，仍会停止并提示先推送。

## 执行流程

1. 检查当前分支和未完成的 Git 操作；当前工作区有未提交修改时仅提示。
2. 执行 `git fetch` 刷新远端引用。
3. 确认当前分支已推送且没有领先或落后远端。
4. 查找目标分支是否已经在某个 worktree 中检出。
5. 复用已有目标 worktree，或创建临时 worktree，并确认目标 worktree 干净。
6. 使用 `pull --ff-only` 更新目标分支。
7. 将当前分支已提交且已推送的内容合并到目标分支。
8. 合并成功后推送目标分支。
9. 仅清理脚本自己创建的临时 worktree。

## 安全约束

以下情况会立即停止，不执行合并或推送：

- 当前处于 detached HEAD。
- 目标 worktree 存在未提交修改。
- 当前工作区或目标 worktree 存在未完成的 merge、rebase、cherry-pick 等操作。
- 当前分支尚未推送或与远端不一致。
- 目标远端分支不存在。
- 目标分支无法快进到远端状态。
- 合并发生冲突。

## License

[MIT](LICENSE)
