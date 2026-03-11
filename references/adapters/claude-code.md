# Claude Code 环境适配

当主 AI 运行在 Claude Code CLI 中时，使用以下工具映射。

## 操作映射

| 通用操作           | Claude Code 工具                  |
| ------------------ | --------------------------------- |
| 在前台终端执行命令 | `Bash`（直接执行）                |
| 在后台终端执行命令 | `Bash` + `run_in_background=true` |
| 检查后台任务输出   | `TaskOutput`（`block=false`）     |
| 读取文件           | `Read`                            |
| 搜索文件           | `Grep` / `Glob`                   |

## 前台执行示例

```
工具: Bash
命令: codex exec -C "<工作目录>" -s read-only --full-auto -o ".codex-output/result.txt" "<prompt>"
```

## 后台执行示例

```
工具: Bash
参数: run_in_background=true, dangerouslyDisableSandbox=true
命令: codex exec -C "<工作目录>" -s read-only --full-auto -o ".codex-output/result.txt" "<prompt>"
```

**重要：** codex-collab 需要写入 `~/.codex-collab/`，这在 Claude Code 的沙箱外，所以必须加 `dangerouslyDisableSandbox=true`。

## 注意事项

- `run_in_background=true` 的命令完成后会自动通知，不需要轮询
- 不要用 `TaskOutput(block=true)` 等待——这会阻塞
- 如果使用 codex-collab，run 和 review 始终后台执行
