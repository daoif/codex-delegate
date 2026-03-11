# codex-delegate

一个 [Agent Skills](https://agentskills.io/) 技能，让 AI Agent 将任务委派给 [OpenAI Codex CLI](https://github.com/openai/codex) 执行。

## 什么是这个？

当你的 AI Agent（Copilot、Claude Code、Cursor 等）遇到需要深度分析的任务时，与其让它自己在代码库中慢慢搜索，不如委派给 Codex——一个专门做代码分析的 Agent。

**典型场景：**
- 在大型代码库中追踪复杂的调用链
- 代码审查（提交前审查、PR 审查）
- 并行研究多个不相关的问题
- 架构分析和方案对比

## 工作模式

```
[主 AI] ──构造 prompt──→ [Codex 在终端执行]
                              ↓ 分析结果写入输出文件
[主 AI] ←──读取并验证──── [输出文件]
  ↓
[主 AI 做实际代码修改]
```

核心原则：**Codex 做分析，主 AI 做修改**。

## 支持的后端

| 后端 | 说明 | 依赖 |
|------|------|------|
| `exec` | 直接使用 `codex exec` CLI | Codex CLI |
| `collab` | 通过 [codex-collab](https://github.com/Kevin7Qi/codex-collab) 桥接 | Codex CLI + Bun |

## 安装

### 前置条件

- [OpenAI Codex CLI](https://github.com/openai/codex)：`npm install -g @openai/codex`

### 项目级安装

将此目录复制到你项目的 skills 目录下：

```bash
# 任选一个位置
cp -r codex-delegate your-project/.github/skills/codex-delegate
cp -r codex-delegate your-project/.agents/skills/codex-delegate
cp -r codex-delegate your-project/.claude/skills/codex-delegate
```

### 用户级安装

```bash
# 任选一个位置
cp -r codex-delegate ~/.copilot/skills/codex-delegate
cp -r codex-delegate ~/.claude/skills/codex-delegate
cp -r codex-delegate ~/.agents/skills/codex-delegate
```

### 配置

复制配置模板并按需修改：

```bash
cp config.example.json config.json
```

配置字段：

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `backend` | 后端模式 | `exec` |
| `sandbox` | 沙箱权限 | `read-only` |
| `concurrency` | 并发模式 | `single` |
| `model` | 模型 | `auto` |
| `timeout` | 超时（秒） | `600` |

## 兼容性

基于 [Agent Skills 开放标准](https://agentskills.io/)，兼容所有支持该标准的 AI 工具：

- VS Code GitHub Copilot
- OpenAI Codex CLI
- Claude Code
- Cursor
- Goose
- Roo Code
- 以及更多...

环境特定的工具映射在 `references/adapters/` 目录下。

## 文件结构

```
codex-delegate/
├── SKILL.md                    # 主入口
├── config.example.json         # 配置模板
├── config.json                 # 本地配置（git ignored）
└── references/
    ├── init-guide.md           # 初始化指南
    ├── exec-manual.md          # exec 操作手册
    ├── collab-install.md       # collab 安装指南
    ├── collab-manual.md        # collab 操作手册
    ├── patterns.md             # 工作模式指南
    ├── parallel.md             # 并行执行指南
    └── adapters/
        ├── copilot.md          # VS Code Copilot 适配
        └── claude-code.md      # Claude Code 适配
```

## License

MIT
