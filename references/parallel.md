# 并行执行指南

当需要同时进行多个独立的 Codex 任务时，使用并行执行。

## 前置条件

- config.json 中 `concurrency` 设为 `multi`
- 每个任务需要独立的输出文件名，避免覆盖

## 执行方式

### Exec 模式并行

每个任务用独立的后台终端：

```shell
# 任务 1（后台终端）
codex exec -C "<目录>" -s read-only --full-auto -o ".codex-output/task1.txt" "<prompt1>"

# 任务 2（后台终端）
codex exec -C "<目录>" -s read-only --full-auto -o ".codex-output/task2.txt" "<prompt2>"
```

等待各任务完成后，读取各输出文件。

### Collab 模式并行

```powershell
# 任务 1（后台）
codex-collab run "<prompt1>" -d "<目录>" -s read-only --content-only

# 任务 2（后台）
codex-collab run "<prompt2>" -d "<目录>" -s read-only --content-only
```

用 `codex-collab jobs` 查看所有线程状态，用 `codex-collab progress <id>` 查看进度。

## 并行适用场景

| 场景         | 示例                                 |
| ------------ | ------------------------------------ |
| 多模块分析   | 分别分析 server、shared、web 三个包  |
| 正交问题调查 | 问题 A 和问题 B 互不相关             |
| 方案对比     | 让两个 Codex 分别分析方案 A 和方案 B |
| 多仓库工作   | 在不同项目目录下同时执行任务         |

## 注意事项

1. **每个任务输出到不同文件** — 防止输出覆盖
2. **不要并行写同一个目录** — 如果多个 Codex 用 `workspace-write` 操作同一目录，可能产生冲突。使用 [Worktree 隔离](./worktree.md) 让每个 Codex 在独立目录中工作
3. **API 速率限制** — 并行任务增加 API 调用量，注意 OpenAI 的速率限制
4. **资源消耗** — 每个 Codex 实例占用系统资源，不建议超过 3 个并行任务
5. **结果综合** — 所有任务完成后，主 AI 综合各任务结果做出决策

## 并行工作流示例

假设需要同时调查前后端两个问题：

```
步骤 1: 构造两个独立的 prompt

步骤 2: 分别在两个后台终端启动
  终端 A → codex exec ... -o .codex-output/frontend.txt "前端问题分析..."
  终端 B → codex exec ... -o .codex-output/backend.txt "后端问题分析..."

步骤 3: 检查完成状态
  各终端是否已完成？

步骤 4: 读取结果
  读取 .codex-output/frontend.txt
  读取 .codex-output/backend.txt

步骤 5: 综合分析，做出修改
```
