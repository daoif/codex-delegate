# Exec 模式操作手册

使用 `codex exec` CLI 直接执行任务。这是最简单直接的方式。

## 命令格式

```shell
codex exec [选项] "<prompt>"
```

## 核心选项

| 选项          | 说明                                   | 示例                          |
| ------------- | -------------------------------------- | ----------------------------- |
| `-C <目录>`   | 工作目录                               | `-C "/path/to/project"`       |
| `-s <模式>`   | 沙箱权限                               | `-s read-only`                |
| `-o <文件>`   | 输出最终消息到文件                     | `-o .codex-output/result.txt` |
| `--full-auto` | 自动执行（workspace-write + 自动审批） |                               |
| `--json`      | 以 JSONL 事件流输出                    |                               |
| `-m <模型>`   | 指定模型                               | `-m o3`                       |
| `--ephemeral` | 不持久化会话                           |                               |
| `--search`    | 启用网络搜索                           |                               |

## 常用命令模板

### 只读分析

最常用的模式。Codex 在项目中搜索和分析，输出结果到文件。

```shell
codex exec -C "<工作目录>" -s read-only --full-auto -o "<输出文件>" "<prompt>"
```

**示例：**

```shell
codex exec -C "/path/to/project" -s read-only --full-auto -o ".codex-output/analysis.txt" "分析 src/executor/ 目录下的执行引擎架构，列出所有关键函数及其调用关系"
```

### 代码审查

审查未提交的更改：

```shell
codex exec review --uncommitted -o ".codex-output/review.txt"
```

审查特定提交：

```shell
codex exec review --commit <SHA> -o ".codex-output/review.txt"
```

带自定义审查重点：

```shell
codex exec review --uncommitted "重点关注类型安全和错误处理" -o ".codex-output/review.txt"
```

### 代码搜索

```shell
codex exec -C "<工作目录>" -s read-only --full-auto -o ".codex-output/search.txt" "在整个代码库中搜索所有使用了 <某个 API> 的地方，列出文件路径、行号和上下文"
```

### 带写权限的任务

当需要 Codex 创建或修改文件时：

```shell
codex exec -C "<工作目录>" -s workspace-write --full-auto -o ".codex-output/result.txt" "<prompt>"
```

**注意：** 这会让 Codex 直接修改工作区文件。确保在执行前理解 prompt 的影响范围。

## 续聊

每次 `codex exec` 执行后会自动保存会话。可以续聊：

**续上一次会话：**

```shell
codex exec resume --last "<follow-up prompt>" -o ".codex-output/followup.txt"
```

**续指定会话（需要会话 ID）：**

```shell
codex exec resume <session-id> "<follow-up prompt>" -o ".codex-output/followup.txt"
```

**什么时候续聊 vs 新会话：**

- 同一个问题需要追问 → 续聊（Codex 保留了上下文）
- 全新的不相关问题 → 新会话

## 执行方式

### 前台执行（阻塞）

适用于预计 < 2 分钟的快速任务。在终端中同步执行，等待完成后直接获取输出。

### 后台执行（非阻塞）

适用于复杂分析（可能耗时 5-20 分钟）。在后台终端中执行，主 AI 可以继续其他工作。之后检查是否完成，完成后读取输出文件。

具体如何启动前台/后台终端，取决于你所在的 AI 宿主环境。

## 输出处理

1. `-o` 参数指定的文件包含 Codex 的最终输出文本
2. 读取该输出文件获取结果
3. **验证** Codex 的结论 — 不要盲信，至少抽查关键文件确认
4. 基于验证后的信息做实际修改

## Prompt 构造技巧

### 好的 prompt

- 给出具体的文件路径或目录范围
- 说明上下文和背景
- 明确期望的输出格式
- 如果有已知线索，一并提供

**示例：**

```
在 src/routes/run.ts 中，/execute/test-action 端点的实现
有一个问题：当 target 为空时不应该报错，而是应该跳过定位直接执行动作。
请分析这个端点的完整逻辑，找出需要修改的位置，并给出修改建议。
相关类型定义在 src/types/ 目录下。
```

### 差的 prompt

```
帮我看看代码有什么问题
```

（太模糊，Codex 不知道看哪里、看什么）

## 错误处理

| 情况             | 处理                                     |
| ---------------- | ---------------------------------------- |
| Codex 超时       | 增加 timeout 或简化 prompt               |
| 输出文件为空     | 检查 Codex 是否报错（看终端输出）        |
| Codex 结论不准确 | 续聊要求它给出依据，或者换个角度重新分析 |
| API Key 问题     | 检查 OPENAI_API_KEY 环境变量是否正确配置 |
