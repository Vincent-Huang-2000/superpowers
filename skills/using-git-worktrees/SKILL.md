---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - ensures an isolated workspace exists via native tools or git worktree fallback
---

# 使用 Git worktree

## 概述

确保在隔离的工作区中开展工作。优先使用平台的原生 worktree 工具。仅在没有原生工具时，通过 Git 手动创建 worktree。

**核心原则** 先检测已有隔离，再尝试原生工具，最后使用 Git。遵循宿主的工作区管理方式。

**开始时告知用户**「我会用 using-git-worktrees 设置隔离的工作区。」

## 步骤 0：检测已有隔离

**在创建任何工作区前，先检查当前工作区是否已经隔离。**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**检查子模块** Git 子模块中也会出现 `GIT_DIR != GIT_COMMON`。在认定已处于 worktree 之前，先排除子模块。

```bash
# 如果返回路径，当前目录属于子模块，按普通仓库处理。
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**如果 `GIT_DIR != GIT_COMMON` 且当前目录不属于子模块**，说明已经处于关联 worktree。直接转到步骤 2 设置项目，复用当前 worktree。

根据分支状态报告：

- 在分支上：「已处于隔离工作区 `<path>`，当前分支为 `<name>`。」
- 在分离的 HEAD 上：「已处于隔离工作区 `<path>`。当前为外部管理的分离 HEAD，收尾时需要创建分支。」

**如果 `GIT_DIR == GIT_COMMON`，或当前目录属于子模块**，则当前是普通仓库检出。

检查用户是否已在指令中说明其 worktree 偏好。只有在偏好未知时，才在创建之前征求同意。

> 「要创建一个隔离的 worktree 吗？这样可以避免本次工作影响当前分支。」

如已有偏好，直接遵循。如果用户拒绝，就在当前位置工作，转到步骤 2。

## 步骤 1：创建隔离工作区

**按顺序尝试以下两种方式。**

### 1a. 原生 worktree 工具（首选）

用户已经在步骤 0 同意使用隔离工作区。检查当前宿主是否提供创建方式，例如 `EnterWorktree`、`WorktreeCreate`、`/worktree` 命令或 `--worktree` 参数。有的话，使用该方式并转到步骤 2。

原生工具会自动选择存放目录、创建分支并清理工作区。此时直接用 `git worktree add` 会创建宿主无法发现或管理的工作区。

仅在没有原生 worktree 工具时执行步骤 1b。

### 1b. 通过 Git 创建 worktree

**仅在步骤 1a 不适用时**，用 Git 手动创建 worktree。

#### 选择目录

按以下顺序选择。用户明确指定的目录优先于文件系统中已有的目录。

1. **检查项目指令中的目录偏好。** 如用户已经指定，直接使用。
2. **检查项目内是否已有 worktree 目录。**

   ```bash
   ls -d .worktrees 2>/dev/null     # 首选（隐藏目录）
   ls -d worktrees 2>/dev/null      # 备选
   ```

   使用找到的目录。如果两个目录都存在，使用 `.worktrees`。
3. **没有其他指引时**，使用项目根目录下的 `.worktrees/`。

#### 安全检查（仅限项目内的目录）

**创建 worktree 前，必须验证 Git 会忽略该目录。**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

如果 Git 未忽略所选目录，将其加入 `.gitignore`，提交该改动后再继续。这样可以避免意外提交整个 worktree 的内容。

#### 创建 worktree

```bash
# 根据选定的目录和分支名确定路径
path="$LOCATION/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**沙箱回退** 如果沙箱拒绝权限，导致 `git worktree add` 失败，告知用户创建受阻，将在当前目录工作。然后在原目录中完成项目设置和基线测试。

## 步骤 2：设置项目

自动识别项目类型并执行相应的设置命令。

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## 步骤 3：验证基线

运行测试，确认工作区的初始状态。

```bash
# 使用当前项目对应的命令
npm test / cargo test / pytest / go test ./...
```

**如果测试失败**，报告失败并询问用户是继续还是先调查。

**如果测试通过**，告知用户可以开始实施。

### 报告

```text
Worktree 已就绪，位于 <full-path>。
测试通过（<N> 项测试，0 项失败）。
可以开始实施 <feature-name>。
```

## 速查

| 情况 | 操作 |
| --- | --- |
| 已处于关联 worktree | 跳过创建（步骤 0） |
| 处于子模块中 | 按普通仓库处理（步骤 0 的子模块检查） |
| 有原生 worktree 工具 | 使用原生工具（步骤 1a） |
| 无原生工具 | 通过 Git 创建（步骤 1b） |
| `.worktrees/` 存在 | 使用该目录并确认 Git 忽略该目录 |
| `worktrees/` 存在 | 使用该目录并确认 Git 忽略该目录 |
| 两个目录都存在 | 使用 `.worktrees/` |
| 两个目录都不存在 | 检查项目指令，默认使用 `.worktrees/` |
| Git 未忽略目录 | 加入 `.gitignore` 并提交 |
| 创建时遇到权限错误 | 在当前目录继续 |
| 基线测试失败 | 报告失败并询问用户 |
| 没有 `package.json` 或 `Cargo.toml` | 跳过对应的依赖安装 |

## 常见借口及应对

| 借口 | 实际要求 |
| --- | --- |
| 「一看就知道这里没有 worktree」 | 执行步骤 0。宿主创建的隔离工作区和子模块都容易误判，以检测命令为准。 |
| 「查找原生工具还不如直接用 `git worktree add`」 | `EnterWorktree` 等原生工具负责目录、分支和清理。跳过这些工具会留下宿主无法发现或管理的工作区。 |
| 「这个目录肯定已经被忽略了」 | 运行 `git check-ignore`。Git 若未忽略这个目录，就可能提交整个 worktree 的内容。 |
| 「目录叫什么都可以」 | 优先遵循明确指令，再检查项目内的现有目录，最后使用默认的 `.worktrees/`。 |
| 「工作区刚建好，基线测试可以晚点跑」 | 基线有问题时，后续很难区分故障来源。现在运行测试，失败后是否继续由用户决定。 |
