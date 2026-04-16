---
title: 私有仓同步公开仓并发布 Pages 教程
type: cheatsheet
topic: git
tags: [github, pages, workflow, publish]
updated: 2026-04-16
source:
status: stable
---

## 目标
将私有仓 `my-notes` 作为主仓，仅对外同步白名单内容到公开仓 `notes`，并由 `notes` 独立部署 GitHub Pages。

## 架构分工
- 私有仓 `my-notes`：写作、整理、敏感内容隔离、同步公开内容。
- 公开仓 `notes`：仅接收公开内容并发布站点。
- 发布链路：
  1. `my-notes/.github/workflows/publish-public.yml` 触发脚本 `scripts/publish_public.sh`。
  2. 脚本把白名单内容推送到 `Allen-zhu/notes:main`。
  3. `notes/.github/workflows/docs.yml` 构建并部署 Pages。

## 一次性配置
1. 创建公开仓：`Allen-zhu/notes`。
2. 生成 GitHub Token（Classic PAT 更直观）：
   - 权限至少包含：`repo`（或最小化到可写目标仓库内容）。
   - 可选：设置过期时间与备注，便于轮换。
3. 在私有仓 `my-notes` 配置 Secret：
   - 位置：`Settings -> Secrets and variables -> Actions -> New repository secret`
   - 名称：`PUBLIC_REPO_TOKEN`
   - 值：上一步 Token
4. 在公开仓 `notes` 启用 Pages：
   - 位置：`Settings -> Pages`
   - `Build and deployment` 选择 `Source: GitHub Actions`

## 日常发布流程
1. 在 `my-notes` 修改公开内容（默认白名单）：
   - `20-cheatsheets/**`
   - `10-index/公开导航.md`
2. `git push` 到 `main/master`。
3. `my-notes` 自动触发 `publish-public`（命中路径时）。
4. 到 `notes` 查看 `docs` workflow 是否成功。
5. 访问公开地址：`https://allen-zhu.github.io/notes/`。

## 自动触发与手动触发
- 自动触发：仅在命中 `publish-public.yml` 的 `paths` 规则时触发。
- 手动触发：`my-notes -> Actions -> publish-public -> Run workflow`。
- 建议：若你频繁跨目录发布，可考虑放宽 `paths` 或取消路径限制。

## 常见故障与处理
### 1) `deploy-pages` 返回 404（Not Found）
- 现象：`actions/deploy-pages` 报错 `Failed to create deployment (status: 404)`。
- 根因：在错误仓库执行了 Pages 部署，或该仓库未启用 Pages。
- 处理：确保只在 `notes` 部署 Pages；`my-notes` 仅做同步，不做部署。

### 2) `publish-public` 没有自动触发
- 现象：push 后只看到 `verify-docs`，没有 `publish-public`。
- 根因：本次改动未命中 `paths` 白名单。
- 处理：
  1. 手动运行 `publish-public`；
  2. 或调整 `publish-public.yml` 的触发路径。

### 3) 公开仓 `notes` 没有新增内容
- 排查顺序：
  1. `my-notes` 的 `publish-public` 是否成功；
  2. `PUBLIC_REPO_TOKEN` 是否存在且权限足够；
  3. 脚本是否被敏感扫描拦截；
  4. `notes` 默认分支是否为 `main`；
  5. `notes` 的 `docs` workflow 是否存在且成功。

## 安全策略
- 只同步白名单目录，默认不公开项目私有笔记和反思记录。
- 发布前执行敏感模式扫描，命中则中止。
- 对已知敏感值做仓库级 denylist。
- 建议定期轮换 `PUBLIC_REPO_TOKEN`，并在变更后立即验证发布链路。

## 运维建议
1. 私有仓保留 `verify-docs`，做构建质量门禁，不做发布。
2. 公开仓独立 `docs` 发布，职责单一，故障域清晰。
3. 每次流程调整后做一次“最小验证”：
   - 改一行 `10-index/公开导航.md`
   - push
   - 验证 `publish-public` -> `notes/docs` -> 页面可访问。
