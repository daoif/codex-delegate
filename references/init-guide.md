# 初始化指南

首次使用 codex-delegate 技能时，按以下步骤初始化配置。

## 环境检测

依次检查以下条件：

### 1. Codex CLI
```powershell
codex --version
```
- 如果找不到命令 → 需要安装：`npm install -g @openai/codex`
- 确认版本 ≥ 0.110.0

### 2. OpenAI API Key
Codex CLI 需要 `OPENAI_API_KEY` 环境变量。检查方式：
```powershell
$env:OPENAI_API_KEY
```
如果为空，需要用户配置。

### 3. Bun（仅 collab 模式需要）
```powershell
bun --version
```
- 如果不需要 collab 模式，可以跳过
- 如果需要安装：https://bun.sh/

### 4. codex-collab（仅 collab 模式需要）
```powershell
codex-collab health
```
- 如果找不到命令 → 参考 [collab 安装指南](./collab-install.md)

## 配置问答

根据检测结果，引导确定以下配置：

### Q1: 后端模式 (`backend`)
- **exec**（推荐）— 直接使用 `codex exec` CLI，无额外依赖
- **collab** — 通过 codex-collab 桥接，支持粒度审批和线程管理，需要 Bun

选择依据：
- 如果 Bun 未安装且不想安装 → `exec`
- 如果需要粒度化的工具调用审批 → `collab`
- 大多数场景 `exec` 就够了

### Q2: 沙箱权限 (`sandbox`)
- **read-only**（推荐）— Codex 只能读取文件，不能执行写操作
- **workspace-write** — Codex 可以在工作区内写文件
- **danger-full-access** — 完全访问权限（危险，不推荐）

选择依据：
- 纯分析/搜索任务 → `read-only`
- 需要 Codex 修改代码 → `workspace-write`
- 几乎不应该用 `danger-full-access`

### Q3: 并发模式 (`concurrency`)
- **single** — 一次只运行一个 Codex 任务
- **multi** — 可以同时运行多个 Codex 任务

选择依据：
- 刚开始使用 → `single`
- 需要并行研究多个问题 → `multi`

### Q4: 工作范围 (`scope`)
- **current-repo** — Codex 只在当前仓库目录下工作
- **multi-repo** — Codex 可以在多个仓库间工作

选择依据：
- 大多数情况 → `current-repo`
- 需要跨项目分析 → `multi-repo`

### Q5: 模型 (`model`)
- **auto** — 让 Codex 自动选择最新最佳模型
- 或指定具体模型名（如 `o3`、`gpt-4.1` 等）

### Q6: 超时 (`timeout`)
- 默认 600 秒（10分钟）
- 复杂分析可以设到 1200（20分钟）

## 生成配置

根据问答结果，生成 `config.json` 写入 skill 目录：
```
.agents/skills/codex-delegate/config.json
```

## 输出目录

创建 `.codex-output/` 目录用于存放 Codex 的输出文件。将此目录加入 `.gitignore`（如果不想提交产出物的话）。
