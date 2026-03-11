# Worktree 隔离指南

当主 AI 和 Codex 同时工作时，使用 Git worktree 隔离工作目录，防止并发修改冲突。

## 为什么需要 worktree

1. **防写保护不可靠** — Codex 的 `read-only` 沙箱在某些平台（如 Windows）上可被绕过，Codex 可能意外修改主工作区文件
2. **并发修改冲突** — 主 AI 和 Codex 同时修改同一文件会产生竞态条件
3. **并行开发** — 多个 Codex 各自在独立分支上开发不同模块，最后 review + merge

## Git worktree 简介

Git 原生功能（2.5+），允许一个仓库拥有多个工作目录，共享同一个 `.git` 数据库。

```
<主工作区>/                    ← 你正常工作的目录
  └── .git/                    ← 共享的 git 数据库

<worktree>/                    ← Codex 的独立工作目录
  └── .git                     ← 一个文件，指向主 .git/
```

- 不是 clone，不需要网络，不复制 git objects
- 磁盘开销 ≈ 源代码大小（不含 `.git/`、`node_modules`）
- 创建和删除都很快

## 操作流程

### 场景 A：只读分析（推荐）

Codex 只需要读代码、不做修改。

```shell
# 1. 创建 worktree（detach 模式，不占分支）
git worktree add <worktree路径> --detach HEAD

# 2. 让 Codex 在 worktree 中工作
codex exec -C "<worktree路径>" -s read-only --full-auto -o "<输出文件>" "<prompt>"

# 3. 清理
git worktree remove <worktree路径>
```

### 场景 B：独立分支开发

Codex 在独立分支上修改代码，完成后 review + merge。

```shell
# 1. 创建 worktree 并新建分支
git worktree add <worktree路径> -b codex/<任务名> HEAD

# 2. 让 Codex 在 worktree 中工作（允许写入）
codex exec -C "<worktree路径>" --full-auto -o "<输出文件>" "<prompt>"

# 3. 查看 Codex 的修改
cd <worktree路径>
git diff
git log --oneline

# 4. 如果满意，合并到主分支
cd <主工作区>
git merge codex/<任务名>

# 5. 清理
git worktree remove <worktree路径>
git branch -d codex/<任务名>
```

### 场景 C：多 Codex 并行开发

多个 Codex 分别在独立分支上开发不同模块。

```shell
# 为每个任务创建独立 worktree + 分支
git worktree add .codex-wt-frontend -b codex/frontend HEAD
git worktree add .codex-wt-backend -b codex/backend HEAD

# 各自启动（后台）
codex exec -C ".codex-wt-frontend" --full-auto -o ".codex-output/frontend.txt" "<前端任务>"
codex exec -C ".codex-wt-backend" --full-auto -o ".codex-output/backend.txt" "<后端任务>"

# 各自完成后 review
cd .codex-wt-frontend && git diff
cd .codex-wt-backend && git diff

# 逐个合并
cd <主工作区>
git merge codex/frontend
git merge codex/backend

# 清理
git worktree remove .codex-wt-frontend
git worktree remove .codex-wt-backend
git branch -d codex/frontend codex/backend
```

## 路径约定

| 方式           | 路径示例               | 特点                                  |
| -------------- | ---------------------- | ------------------------------------- |
| 仓库内子目录   | `.codex-wt-<名称>/`   | 方便管理，需 .gitignore 排除          |
| 仓库外同级目录 | `../<仓库名>-codex/`  | 不污染主仓库                          |
| 系统临时目录   | `$TEMP/codex-<名称>/` | 用完即删，但路径不直观                |

建议使用仓库内子目录（`.codex-wt-*`），并在 `.gitignore` 中排除：

```
.codex-wt-*/
```

## 注意事项

1. **node_modules 不共享** — worktree 有独立的文件树，如果 Codex 需要运行测试/构建，需要在 worktree 中单独 `npm install`
2. **同一分支限制** — 同一分支不能同时在多个 worktree 中 checkout。用 `--detach` 或创建独立分支解决
3. **及时清理** — 用完后 `git worktree remove` 释放磁盘空间和分支锁
4. **输出文件路径** — `-o` 的输出文件建议用主工作区的绝对路径（如 `.codex-output/`），这样不管 Codex 在哪个 worktree 工作，结果都汇集到同一位置
