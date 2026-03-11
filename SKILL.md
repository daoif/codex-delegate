---
name: codex-delegate
description: 将任务委派给 OpenAI Codex CLI 执行。用于深度代码分析、代码审查、并行研究等场景。当用户明确要求使用 Codex，或全局/项目级配置中指定通过 Codex 完成某项工作时触发。支持 exec（直接执行）和 collab（协作桥接）两种后端模式。
license: MIT
compatibility: 需要 OpenAI Codex CLI (npm install -g @openai/codex)。collab 模式额外需要 Bun 运行时。
metadata:
  author: daoif
  version: "0.1.0"
---

# Codex 委派技能

通过 OpenAI Codex CLI 委派任务，让 Codex 作为独立 agent 执行深度分析、代码审查、代码搜索等工作。主 AI 负责构造 prompt、验证结果、做实际修改。

## 核心原则

1. **Codex 做分析，你做修改** — Codex 擅长在大型代码库中搜索和分析，你负责验证其结论并做实际的代码变更
2. **给足上下文** — 每次调用 Codex 时，prompt 中必须包含充分的背景信息（问题描述、相关文件路径、已知线索）
3. **不阻塞主流程** — 长时间运行的 Codex 任务应在后台执行，不要阻塞主 AI 的工作
4. **验证优先** — Codex 的输出需要你验证后再采信，不要盲信

## 环境适配

本 skill 与 AI 宿主环境无关。不同宿主（VS Code Copilot、Claude Code、CLI 等）执行终端命令和读取文件的方式不同。

如果你的 skill 目录下存在 `references/adapters/` 文件夹，请根据你当前运行的环境读取对应的适配文档，了解如何映射通用操作到具体工具调用。

如果没有适配文档，按照你所在环境的标准方式执行"在终端运行命令"和"读取文件"即可。

## 配置

读取 [config.json](./config.json) 确定当前工作模式。如果配置文件不存在，参考 [config.example.json](./config.example.json) 创建，或按 [初始化指南](./references/init-guide.md) 交互式生成。

配置字段说明：

| 字段 | 含义 | 可选值 |
|------|------|--------|
| `backend` | 后端模式 | `exec`（直接 CLI）/ `collab`（协作桥接） |
| `sandbox` | 沙箱权限 | `read-only` / `workspace-write` / `danger-full-access` |
| `concurrency` | 并发模式 | `single`（单任务）/ `multi`（多任务并行） |
| `scope` | 工作范围 | `current-repo`（当前仓库）/ `multi-repo`（多仓库） |
| `model` | 模型选择 | `auto` 或具体模型名 |
| `timeout` | 超时（秒） | 数字，默认 600 |
| `outputDir` | 输出目录 | 相对路径，默认 `.codex-output` |

## 工作流程

### 1. 确认配置
```
读取 config.json → 确定 backend 是 exec 还是 collab
```

### 2. 根据 backend 查阅操作手册
- `exec` → 阅读 [exec 操作手册](./references/exec-manual.md)
- `collab` → 阅读 [collab 操作手册](./references/collab-manual.md)
  - 如果 collab 未安装 → 阅读 [collab 安装指南](./references/collab-install.md)

### 3. 根据场景选择工作模式
阅读 [工作模式指南](./references/patterns.md) 了解不同场景（分析、审查、研究）的最佳实践。

### 4. 并行任务（如需）
如果需要同时运行多个 Codex 任务，阅读 [并行执行指南](./references/parallel.md)。

## 快速参考（exec 模式）

以下是最常用的命令模板，具体细节见 exec 操作手册。

**只读分析：**
```shell
codex exec -C "<工作目录>" -s read-only --full-auto -o "<输出文件>" "<prompt>"
```

**代码审查：**
```shell
codex exec review -o "<输出文件>" --uncommitted
```

**续聊：**
```shell
codex exec resume --last "<follow-up prompt>" -o "<输出文件>"
```

**读取结果：**
```
读取输出文件内容（使用你所在环境的文件读取能力）
```

## 触发条件

此技能在以下情况被加载：
1. 用户明确要求使用 Codex 做某事（如"让 Codex 分析一下..."）
2. 全局或项目级配置中指定通过 Codex 完成特定任务
3. 用户使用 `/codex-delegate` 斜杠命令
