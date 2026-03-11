# Collab 模式操作手册

使用 `codex-collab` CLI 通过 app-server 协议与 Codex 交互。相比 exec 模式，collab 提供粒度审批、线程管理和进度流式输出。

## 前提

确保 `codex-collab health` 通过。如未安装，参考 [collab 安装指南](./collab-install.md)。

## 命令格式

```powershell
codex-collab <命令> [选项]
```

## 常用命令

### 执行任务

```powershell
codex-collab run "<prompt>" -d "<工作目录>" -s read-only --content-only
```

`--content-only` 只输出结果文本，抑制进度行。

### 代码审查

```powershell
codex-collab review -d "<工作目录>" --content-only
```

审查模式：

- 默认 `pr`（PR 审查）
- `--mode uncommitted`（未提交的更改）
- `--mode commit --ref <SHA>`（特定提交）

### 续聊

```powershell
codex-collab run --resume <id> "<follow-up prompt>" -d "<工作目录>" --content-only
```

### 线程管理

```powershell
codex-collab jobs              # 列出所有线程
codex-collab progress <id>     # 查看进行中的任务进度
codex-collab output <id>       # 查看完整输出日志
codex-collab kill <id>         # 中断运行中的任务
codex-collab delete <id>       # 删除线程
codex-collab clean             # 清理旧日志
```

## 选项参考

| 选项                     | 说明                                         | 默认值           |
| ------------------------ | -------------------------------------------- | ---------------- |
| `-d, --dir <路径>`       | 工作目录                                     | 当前目录         |
| `-m, --model <模型>`     | 模型名                                       | auto（最新可用） |
| `-r, --reasoning <级别>` | 推理强度：low/medium/high/xhigh              | auto（模型最高） |
| `-s, --sandbox <模式>`   | read-only/workspace-write/danger-full-access | workspace-write  |
| `--approval <策略>`      | never/on-request/on-failure/untrusted        | never            |
| `--content-only`         | 只输出结果文本                               |                  |
| `--timeout <秒>`         | 超时                                         | 1200             |
| `--resume <id>`          | 续聊线程                                     |                  |
| `--base <分支>`          | PR 审查的基础分支                            | main             |

## 全局配置

```powershell
codex-collab config                      # 查看当前配置
codex-collab config model gpt-5.3-codex  # 设置默认模型
codex-collab config reasoning high       # 设置推理强度
codex-collab config model --unset        # 重置为自动检测
```

优先级：CLI 参数 > config 文件 > 自动检测

## 执行方式

### run 和 review — 后台执行

这些命令可能耗时数分钟，应在后台终端中执行。启动后主 AI 可以继续其他工作。完成后检查输出。期间可以用 `progress <id>` 查看进度。

### 其他命令 — 前台执行

`jobs`、`kill`、`progress`、`output` 等秒级完成，在前台同步执行即可。

## 审批机制

当 `--approval` 设为 `on-request` 或 `untrusted` 时，Codex 执行某些工具调用前会请求审批。

审批请求写入 `~/.codex-collab/approvals/<id>.json`，可以用：

```powershell
codex-collab approve <id>   # 批准
codex-collab decline <id>   # 拒绝
```

主 AI 可以根据请求内容自动决定是否批准。

## exec vs collab 选择

| 场景                   | 推荐   |
| ---------------------- | ------ |
| 快速只读分析           | exec   |
| 需要粒度审批           | collab |
| 需要管理多个线程       | collab |
| 简单直接不想装依赖     | exec   |
| 长时间运行需要监控进度 | collab |
