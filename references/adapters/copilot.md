# VS Code Copilot 环境适配

当主 AI 运行在 VS Code GitHub Copilot（Agent 模式）中时，使用以下工具映射。

## 操作映射

| 通用操作 | Copilot 工具 |
|---------|-------------|
| 在前台终端执行命令 | `run_in_terminal`（`isBackground: false`） |
| 在后台终端执行命令 | `run_in_terminal`（`isBackground: true`） |
| 检查后台终端输出 | `get_terminal_output`（传入终端 ID） |
| 读取文件 | `read_file` |
| 搜索文件 | `file_search` / `grep_search` / `semantic_search` |

## 前台执行示例

```
工具: run_in_terminal
参数:
  command: codex exec -C "<工作目录>" -s read-only --full-auto -o ".codex-output/result.txt" "<prompt>"
  isBackground: false
  timeout: 120000  # 2 分钟超时
```

## 后台执行示例

```
工具: run_in_terminal
参数:
  command: codex exec -C "<工作目录>" -s read-only --full-auto -o ".codex-output/result.txt" "<prompt>"
  isBackground: true
```

之后用 `get_terminal_output` 检查状态，完成后用 `read_file` 读取输出文件。

## 注意事项

- Copilot 不支持并行调用 `run_in_terminal`，需要依次启动后台终端
- 后台终端启动后会返回终端 ID，用于后续检查
- 前台执行是阻塞的，设置合理的 `timeout` 避免无限等待
