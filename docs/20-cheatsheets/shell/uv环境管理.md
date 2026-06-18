---
title: uv 常用命令速查
type: cheatsheet
topic: shell
tags: [python, uv, venv]
updated: 2026-04-15
source: /Users/hongjunzhu/works/xianyuan/hsh-codeX/synerai_hsh/docs/dev-workflow.md, 实战补充
status: stable
---

## 结论
日常优先使用 `uv run` 与 `uv sync`，减少手动激活虚拟环境带来的环境不一致问题。

## uv 工作流程（推荐）
1. 首次进入项目执行 `uv sync`，创建并同步 `.venv`。  
2. 日常运行统一使用 `uv run <command>`，避免“激活错环境”。  
3. 新增依赖用 `uv add` / `uv add --dev`，然后提交 `uv.lock`。  
4. 团队协作拉取代码后，先 `uv sync` 再启动服务或脚本。  

## 安装与版本
```bash
# 查看 uv 版本，确认命令可用
uv --version
```

## 依赖同步
```bash
# 按 pyproject.toml + uv.lock 同步依赖到 .venv
uv sync

# 包含开发依赖一起同步
uv sync --dev
```

## 运行命令（推荐）
```bash
# 在项目环境中直接执行脚本（无需手动 activate）
uv run python3 "scripts/run_auto_assign.py"
```

```bash
# 启动服务（直接走 uv 管理的环境）
uv run uvicorn app.main:app --forwarded-allow-ips "*" --proxy-headers --host 0.0.0.0 --port 8008 --log-level warning
```

## 管理依赖
```bash
# 添加运行时依赖（会更新 pyproject.toml 和 uv.lock）
uv add requests

# 添加开发依赖
uv add --dev mkdocs mkdocs-material

# 移除依赖
uv remove requests
```

## 锁文件与环境
```bash
# 仅更新锁文件（不安装）
uv lock

# 创建虚拟环境（通常 uv sync 会自动创建）
uv venv
```

## 传统激活方式（可选）
```bash
# 激活环境，使 python/pip 指向 .venv
source ".venv/bin/activate"

# 退出环境
deactivate
```

## 实际启动项目（synerai_hsh）
```bash
# 1) 进入项目目录
cd "/Users/hongjunzhu/works/xianyuan/hsh-codeX/synerai_hsh"

# 2) 同步依赖
uv sync

# 3) 启动服务
uv run uvicorn app.main:app --forwarded-allow-ips "*" --proxy-headers --host 0.0.0.0 --port 8008 --log-level warning
```

```bash
# 执行项目脚本（示例）
cd "/Users/hongjunzhu/works/xianyuan/hsh-codeX/synerai_hsh"
uv sync
uv run python3 "scripts/run_auto_assign.py"
```

## 常见问题排查
- `ModuleNotFoundError`：先执行 `uv sync`，再使用 `uv run ...` 重试。
- 依赖与锁文件不一致：执行 `uv lock` 后再 `uv sync`。
- 构建时报 cache 权限错误：检查 `~/.cache/uv` 权限，必要时清理后重试。
- CI 与本地行为不一致：确保提交 `uv.lock`，并在 CI 使用 `uv sync --dev`。
