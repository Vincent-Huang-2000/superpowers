---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work
---

# 开发分支收尾

## 概述

**核心原则** 验证测试 → 检测环境 → 给出选项 → 执行选择 → 清理。

**开始时告知用户**「我会用 finishing-a-development-branch 完成本次工作。」

## 步骤 1：验证测试

运行项目的完整测试套件（`npm test`、`cargo test`、`pytest` 或 `go test ./...`）。

**如果测试失败**，报告失败并停下来。等测试全通过后再给出收尾选项。

```text
测试失败（<N> 项失败）。收尾前需要修复。

[展示失败结果]
```

**如果测试通过**，转到步骤 2。

## 步骤 2：检测环境

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
# 趁还在工作区内记录路径。步骤 5 会切换目录，步骤 6 清理时需要这个值。
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

根据检测结果选择菜单和清理方式。

| 状态 | 菜单 | 清理 |
| --- | --- | --- |
| `GIT_DIR == GIT_COMMON`（普通仓库） | 标准的 3 个选项 | 无需清理 worktree |
| `GIT_DIR != GIT_COMMON`，位于命名分支 | 标准的 3 个选项 | 根据工作区来源处理（见步骤 6） |
| `GIT_DIR != GIT_COMMON`，位于分离的 HEAD | 精简的 2 个选项（无合并） | 工作区由外部管理，保留原位 |

## 步骤 3：确定基准分支

基准分支是本次工作从中分出的分支，通常可以从计划、对话或当前分支的上游配置中找到。如果仍不确定，询问「这个分支是从 <最可能的基准分支> 分出来的吗？」

先确认再合并。合入错误的分支，撤销成本很高。

## 步骤 4：给出选项

**普通仓库和位于命名分支的 worktree，给出以下 3 个选项。**

```text
实现已完成。你想怎样处理？

1. 在本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保留分支现状（我稍后处理）

请选择一项。
```

**位于分离的 HEAD，给出以下 2 个选项。**

```text
实现已完成。当前处于分离的 HEAD，工作区由外部管理。

1. 推送为新分支并创建 Pull Request
2. 保留现状（我稍后处理）

请选择一项。
```

按上面的原文简洁地呈现相应选项。仅当用户明确要求时，才进入下文「用户要求放弃改动时」的流程。等待用户回答，由用户决定集成方式。

## 步骤 5：执行选择

### 选项 1：在本地合并

```bash
# 获取主仓库根目录，确保从安全的位置运行命令
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 先合并，验证成功后再移除任何内容
git checkout <base-branch>
git pull
git merge <feature-branch>

# 在合并结果上运行测试
<test command>
```

如果合并结果的测试失败，停下来调查，保留 worktree 和分支。此时合并结果还在本地，可以恢复。

合并结果验证通过后，按步骤 6 清理 worktree，然后删除分支。

```bash
git branch -d <feature-branch>
```

### 选项 2：推送并创建 PR

```bash
git push -u origin <feature-branch>
# 从分离的 HEAD 推送时，在远端为新分支命名：
# git push origin HEAD:refs/heads/<new-branch>
```

用代码托管平台提供的工具向 `<base-branch>` 发起拉取请求或合并请求。可以用平台 CLI，也可以打开多数平台在推送后返回的创建链接。遵循仓库已有的 PR 模板和约定，并把 PR 链接发给用户。

保留 worktree，方便用户后续处理 PR 反馈。

### 选项 3：保留现状

报告「已保留分支 <name>，worktree 位于 <path>。」

### 用户要求放弃改动时

仅在用户明确要求丢弃这次工作时执行这条流程。先向用户确认。

```text
此操作会永久删除以下内容。

- 分支 <name>
- 所有提交 <commit-list>
- 路径 <path> 下的 worktree

请输入 discard 确认。
```

等待用户原样输入 `discard`。收到确认后，进入主仓库目录。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

然后按步骤 6 清理 worktree，并强制删除分支。

```bash
git branch -D <feature-branch>
```

## 步骤 6：清理工作区

**仅在选项 1 或用户已确认放弃时清理。** 选项 2 和选项 3 始终保留 worktree。需要清理的两种流程均已进入主仓库根目录。移除 worktree 必须从其外部执行，使用步骤 2 在切换目录前记录的 `GIT_DIR`、`GIT_COMMON` 和 `WORKTREE_PATH`。

**如果 `GIT_DIR == GIT_COMMON`**，当前是普通仓库，无需清理 worktree。

**如果 `WORKTREE_PATH` 位于 `.worktrees/` 或 `worktrees/` 下**，则该 worktree 由 Superpowers 创建，负责清理。

```bash
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 清理过期登记记录
```

**如果移除被拒绝**（`contains modified or untracked files`），说明 worktree 中有可能只存于此处的文件，例如未提交的计划、笔记或临时文件。自行使用 `--force` 会永久删除这些文件。先展示涉及的内容并询问用户。

```bash
git -C "$WORKTREE_PATH" status --porcelain -uall
```

```text
无法移除 worktree，以下文件尚未提交。

<file list>

1. 清理前把这些文件提交到 <branch>
2. 把这些文件移到 <main repo root>
3. 删除这些文件（无法恢复）

请选择一项。
```

执行用户选择后，再移除 worktree。

**其他情况**说明工作区由宿主管理，保留原位。如果平台提供退出工作区的工具，就使用该工具。

## 速查

| 选项 | 合并 | 推送 | 保留 worktree | 清理分支 |
| --- | --- | --- | --- | --- |
| 1. 在本地合并 | 是 | — | — | 是 |
| 2. 创建 PR | — | 是 | 是 | — |
| 3. 保留现状 | — | — | 是 | — |
| 放弃（仅限明确要求） | — | — | — | 是（强制） |

## 常见借口及应对

| 借口 | 实际要求 |
| --- | --- |
| 「这轮之前已经跑过测试了」 | 在即将集成的内容上运行完整套件。测试通过只能证明当时测试的文件状态。 |
| 「用户显然希望直接合并」 | 集成方式由用户决定。给出菜单并等待。 |
| 「这个功能似乎不需要了，我提议放弃吧」 | 按原文给出完整菜单。用户明确要求放弃后，才进入对应流程。 |
| 「『嗯，删掉吧』也算确认」 | 只有用户输入 `discard` 才授权删除。 |
| 「PR 已经创建，worktree 可以清掉了」 | 用户会在原 worktree 中处理 PR 反馈，保留到改动合入为止。 |
| 「另一个 worktree 好像过期了，顺手清掉吧」 | 仅清理 `.worktrees/` 或 `worktrees/` 下本次涉及的 worktree。其他工作区由宿主管理。 |
| 「移除被拒绝，加上 `--force` 就能清理完」 | 拒绝意味着某些文件可能只存在于该 worktree 中。`--force` 会永久删除它们。先展示文件并询问用户。 |
| 「合并后的测试失败可能只是偶发」 | 合并结果一旦失败就停止，调查期间保留分支和 worktree。 |
| 「基准分支肯定是 main」 | 确认分叉点，必要时询问。合入错误的分支代价很高。 |
| 「推送被拒绝，强推一下就好」 | 推送被拒绝意味着远端已有变化，先调查。只有用户明确要求时才强推。 |
