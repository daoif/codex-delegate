# Collab 安装指南

codex-collab 是一个 Codex 协作桥接工具，通过 `codex app-server` JSON-RPC 协议与 Codex 通信，提供粒度审批、线程管理和进度流式输出。

## 前置条件

### 1. Bun ≥ 1.0

```powershell
bun --version
```

如未安装，前往 https://bun.sh/ 安装。

### 2. Codex CLI

```powershell
codex --version
```

如未安装：`npm install -g @openai/codex`

### 3. Git

需要将 codex-collab 仓库克隆到本地：

```shell
git clone https://github.com/Kevin7Qi/codex-collab.git <安装目录>
```

## 安装步骤

### Windows

```powershell
cd <codex-collab 目录>
powershell -ExecutionPolicy Bypass -File install.ps1
```

开发模式（实时同步源码修改）：

```powershell
powershell -ExecutionPolicy Bypass -File install.ps1 -Dev
```

**注意：** 开发模式需要 Windows 开发者模式或管理员权限（用于创建符号链接）。

### Linux / macOS

```bash
cd <codex-collab 目录>
./install.sh
```

## 安装验证

安装后重新打开终端，然后：

```powershell
codex-collab health
```

应该看到所有依赖检查通过。

## 安装位置

安装器会将文件部署到：

- Skill 目录：`~/.claude/skills/codex-collab/`
- 可执行文件：`~/.local/bin/codex-collab`（Linux/macOS）或 `%USERPROFILE%\.local\bin\codex-collab`（Windows）

安装完成后 `codex-collab` 命令可在终端中使用。

## 安装后配置

更新 config.json 中的 `backend` 为 `collab`：

```json
{
  "backend": "collab"
}
```

## 卸载

删除安装目录和可执行文件即可：

```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\codex-collab"
Remove-Item -Force "$env:USERPROFILE\.local\bin\codex-collab*"
```
