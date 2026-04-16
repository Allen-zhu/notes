---
title: 多账号 SSH 配置与 GitHub/GitLab 接入
type: cheatsheet
topic: git
tags: [git, ssh, github, gitlab]
updated: 2026-04-14
source:
status: stable
---

## 结论
多账号场景的核心是为每个账号使用独立密钥，并在 `~/.ssh/config` 中用 `Host` 别名做路由。

## 一、场景与命名规范
- 个人 GitHub：`github-personal`
- 公司 GitHub：`github-work`
- 公司 GitLab：`gitlab-work`

建议密钥命名：
- `~/.ssh/id_ed25519_github_personal`
- `~/.ssh/id_ed25519_github_work`
- `~/.ssh/id_ed25519_gitlab_work`

## 二、创建 SSH 密钥
```bash
# 个人 GitHub
ssh-keygen -t ed25519 -C "your_email@example.com" -f "$HOME/.ssh/id_ed25519_github_personal"

# 公司 GitHub
ssh-keygen -t ed25519 -C "your_work_email@example.com" -f "$HOME/.ssh/id_ed25519_github_work"

# 公司 GitLab
ssh-keygen -t ed25519 -C "your_work_email@example.com" -f "$HOME/.ssh/id_ed25519_gitlab_work"
```

说明：
- `-t ed25519`：推荐算法，安全性与性能较好。
- `-C`：注释字段，通常写邮箱便于识别。
- `-f`：指定密钥文件名，避免覆盖默认 `id_ed25519`。

## 三、权限与 ssh-agent
`ssh-agent` 是 SSH 私钥托管进程：它在内存中管理已加载私钥，并在 `ssh/git` 连接时代为完成认证签名，减少重复输入私钥口令。

常用配套命令：
- `ssh-add -l`：查看当前 agent 已加载的密钥。
- `ssh-add -D`：清空当前 agent 中已加载的全部密钥。
- `ssh-agent -k`：终止当前 agent 进程。

```bash
# 目录与文件权限
chmod 700 "$HOME/.ssh"
chmod 600 "$HOME/.ssh/id_ed25519_github_personal" "$HOME/.ssh/id_ed25519_github_work" "$HOME/.ssh/id_ed25519_gitlab_work"
chmod 644 "$HOME/.ssh/"*.pub

# 启动 agent
eval "$(ssh-agent -s)"

# 加载私钥（Linux/macOS 通用）
ssh-add "$HOME/.ssh/id_ed25519_github_personal"
ssh-add "$HOME/.ssh/id_ed25519_github_work"
ssh-add "$HOME/.ssh/id_ed25519_gitlab_work"
```

macOS 可选：
```bash
# 写入钥匙串（macOS）
ssh-add --apple-use-keychain "$HOME/.ssh/id_ed25519_github_personal"
ssh-add --apple-use-keychain "$HOME/.ssh/id_ed25519_github_work"
ssh-add --apple-use-keychain "$HOME/.ssh/id_ed25519_gitlab_work"
```

## 四、配置 ~/.ssh/config
`~/.ssh/config` 示例：
```sshconfig
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github_personal
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github_work
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes

Host gitlab-work
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519_gitlab_work
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes
```

说明：
- `Host` 是本地别名，后续 remote URL 使用该别名。
- `UseKeychain yes` 主要用于 macOS；Linux 可保留或移除。

## 五、公钥拷贝
```bash
cat "$HOME/.ssh/id_ed25519_github_personal.pub"
cat "$HOME/.ssh/id_ed25519_github_work.pub"
cat "$HOME/.ssh/id_ed25519_gitlab_work.pub"
```

macOS 可直接复制到剪贴板：
```bash
pbcopy < "$HOME/.ssh/id_ed25519_github_personal.pub"
```

## 六、配置到 GitHub/GitLab
- GitHub：`Settings -> SSH and GPG keys -> New SSH key`，粘贴 `.pub` 内容保存。
- GitLab：`Preferences(或 User Settings) -> SSH Keys -> Add new key`，粘贴 `.pub` 内容保存。

注意：
- 每个平台/账号都要添加对应公钥。
- 私钥（无 `.pub` 后缀）禁止上传到任何平台。

## 七、仓库 remote 绑定
新增仓库时直接使用别名：
```bash
# GitHub 个人
git clone "git@github-personal:<username>/<repo>.git"

# GitHub 公司
git clone "git@github-work:<org>/<repo>.git"

# GitLab 公司
git clone "git@gitlab-work:<group>/<repo>.git"
```

已有仓库切换 remote：
```bash
git remote -v
git remote set-url origin "git@github-work:<org>/<repo>.git"
```

## 八、连接验证
```bash
ssh -T "git@github-personal"
ssh -T "git@github-work"
ssh -T "git@gitlab-work"
```

预期：
- GitHub 会显示 `Hi <user>! You've successfully authenticated...`
- GitLab 会显示 `Welcome to GitLab, @<user>!`

## 九、常见问题排查
- `Permission denied (publickey)`：确认公钥已添加到正确账号，并执行 `ssh -vT git@<host_alias>` 看使用的是哪把密钥。
- `Could not resolve hostname <host_alias>`：检查 `~/.ssh/config` 的 `Host` 别名拼写。
- `Repository not found`：检查 remote 路径 `<org>/<repo>` 是否正确，且账号有权限。
- 标签/提交作者混乱：检查仓库级 `user.name` 与 `user.email`。

## 十、推荐最小流程（可复用）
1. `ssh-keygen` 为每个账号生成独立密钥。
2. 写 `~/.ssh/config` 绑定 `Host` 别名与 `IdentityFile`。
3. 把对应 `.pub` 添加到 GitHub/GitLab 账号。
4. `ssh -T git@<host_alias>` 验证。
5. 仓库 `origin` 改成 `git@<host_alias>:<org>/<repo>.git`。
