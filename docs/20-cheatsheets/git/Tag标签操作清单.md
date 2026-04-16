---
title: Git Tag 标签操作清单
type: cheatsheet
topic: git
tags: [git, tag, release]
updated: 2026-04-14
source: 用户整理（2026-04-14）
status: stable
---

## 结论
Tag 适合做版本锚点，推荐优先使用带注释标签（`-a -m`）并在远端及时同步。

## 一、创建标签（打书签）
- `git tag v1.0.0`：创建轻量标签，仅指向当前提交，不含备注信息。
- `git tag -a v1.0.0 -m "发布说明内容"`：创建带注释标签，包含作者、时间和说明，推荐用于发布版本。
- `git tag "test-v$(date +'%y.%m%d.%H%M%S')"`：创建带时间戳标签，便于测试批次追踪（Linux/Mac/Git Bash）。
- `git tag -a v1.0.0 <commit_id>`：给历史提交补打标签，适合补齐发布记录。

## 二、查看标签（翻书签）
- `git tag`：列出所有本地标签。
- `git tag -l "test-*"`：按模式过滤标签，例如仅查看 `test-` 前缀。
- `git show v1.0.0`：查看标签详情（注释、提交信息、差异）。
- `git log --oneline --graph --decorate`：在提交图中显示标签位置。

## 三、推送与同步（云端同步）
- `git push origin v1.0.0`：推送单个标签到远端。
- `git push origin --tags`：一次性推送所有本地标签。
- `git fetch origin --tags`：拉取远端标签到本地（补齐缺失标签）。

## 四、切换与回溯（版本跳转）
- `git checkout v1.0.0`：切换到指定标签快照，仅建议查看代码（会进入 detached HEAD）。
- `git checkout -b branch_name v1.0.0`：从标签创建新分支，适合在旧版本上修复问题。

## 五、删除与清理（维护空间）
- `git tag -d v1.0.0`：删除本地标签。
- `git push origin --delete v1.0.0`：删除远程标签。
- `git tag -l "test-*" | xargs git tag -d`：批量删除本地 `test-` 前缀标签。

## 注意事项
- 删除标签操作不可逆，执行前先 `git tag -l "<pattern>"` 复核范围。
- `git checkout <tag>` 后如果要继续开发，务必新建分支避免提交丢失。
- 批量删除命令建议先 dry-run：`git tag -l "test-*"`。
