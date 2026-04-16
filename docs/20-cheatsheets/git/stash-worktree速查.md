---
title: Git stash 与 worktree 速查
type: cheatsheet
topic: git
tags: [git, stash, worktree]
updated: 2026-04-14
source: /Users/hongjunzhu/works/xianyuan/hsh-codeX/synerai_hsh/docs/dev-workflow.md
status: stable
---

## stash 常用
```bash
# 暂存当前改动到 stash 栈，并附带说明
git stash push -m "wip"
# 恢复指定 stash，但保留该 stash 记录
git stash apply stash@{0}
# 或 git stash pop（恢复后会删除该 stash，操作前确认）

# 先暂存部分文件，再只 stash 未暂存改动（保留 staged 结果）
git add file1
git stash push -k -m "other"

# 仅 stash 指定文件，避免影响其他改动
git stash push -m "temp" -- path/to/file1 path/to/file2

# 查看 stash 列表
git stash list
# 查看指定 stash 的详细差异
git stash show -p stash@{0}
# 删除指定 stash（删除后不可恢复）
git stash drop stash@{0}
```

## worktree 双环境（test/prod）
```bash
# 进入主仓库并更新远端引用
cd synerai_hsh
git fetch origin
# 创建 test 工作树，跟踪 origin/test
git worktree add -b test ../synerai_hsh-test origin/test
# 创建 main 工作树，跟踪 origin/main
git worktree add -b main ../synerai_hsh-prod origin/main
```

```bash
# 更新 test 工作树
cd ../synerai_hsh-test
git pull --no-rebase

# 更新 prod/main 工作树
cd ../synerai_hsh-prod
git pull --no-rebase
```

## 常见问题
- `src refspec ... does not match any`：本地无分支，先确认当前分支再推送。
- `detached HEAD`：`git switch -c <local-branch> origin/<remote-branch>`。
- 分支被其他 worktree 占用：改用新本地分支跟踪同一远端分支。
