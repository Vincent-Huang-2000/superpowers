# 个人开发约定

## 改动范围

- 围绕当前任务做改动，避免夹带无关修改。
- 按实际开发需要定制技能、配置和工作流。

## 项目架构

Superpowers 将一套开发工作流打包给多个编程助手使用。各宿主共享 `skills/`，通过各自的插件清单、原生技能发现或会话钩子加载技能。

### 技能与工作流

- `skills/<技能名>/SKILL.md` 定义技能。开头的 `name` 和 `description` 供宿主发现技能，正文定义具体步骤。同目录可包含按需读取的 `references/`、提示模板，以及被技能调用的 `scripts/`。 [示例技能](skills/brainstorming/SKILL.md)
- `skills/using-superpowers/SKILL.md` 说明何时检查和调用技能。`skills/using-superpowers/references/` 集中记录宿主工具和能力的差异。调整工作流时先看对应技能，调整宿主工具用法时先看该宿主的映射。 [启动技能](skills/using-superpowers/SKILL.md) · [工具映射](skills/using-superpowers/references/)

#### 技能索引

按当前任务选择技能，通过宿主提供的方式加载对应 `SKILL.md`，再按正文引用读取后续技能、模板或参考文档。技能正文中的 `superpowers:<技能名>` 指向下表对应的技能。

| 技能 | 何时使用 |
| --- | --- |
| [using-superpowers](skills/using-superpowers/SKILL.md) | 会话开始时，确定如何发现、选择和调用适用技能。 |
| [brainstorming](skills/brainstorming/SKILL.md) | 设计新功能或调整行为前，澄清需求、比较方案并确认设计。 |
| [writing-plans](skills/writing-plans/SKILL.md) | 已有设计或需求时，把多步骤工作拆成可执行、可验证的计划。 |
| [using-git-worktrees](skills/using-git-worktrees/SKILL.md) | 开始需要隔离的开发工作或准备执行计划时，检查现有工作区或准备 worktree。 |
| [executing-plans](skills/executing-plans/SKILL.md) | 用户选择由当前会话执行计划，或宿主没有子代理工具时。 |
| [subagent-driven-development](skills/subagent-driven-development/SKILL.md) | 选择由子代理执行实施计划时，逐项派发实现和独立审查。 |
| [dispatching-parallel-agents](skills/dispatching-parallel-agents/SKILL.md) | 有两个或更多任务可以独立处理，且没有共享状态或顺序依赖时。 |
| [test-driven-development](skills/test-driven-development/SKILL.md) | 实现功能或修复缺陷时，先写失败测试，再实现并重构。 |
| [systematic-debugging](skills/systematic-debugging/SKILL.md) | 遇到缺陷、测试失败或意外行为时，先复现并定位根因，再提出修复。 |
| [requesting-code-review](skills/requesting-code-review/SKILL.md) | 完成任务、实现主要功能或准备合并时，整理变更范围并发起独立审查。 |
| [receiving-code-review](skills/receiving-code-review/SKILL.md) | 收到审查意见后，先核实其含义和适用性，再处理反馈。 |
| [verification-before-completion](skills/verification-before-completion/SKILL.md) | 声明工作完成、缺陷已修复或测试通过前，运行相关检查并读取结果。 |
| [finishing-a-development-branch](skills/finishing-a-development-branch/SKILL.md) | 实现完成且检查通过后，处理合并、PR、保留分支或清理等收尾选项。 |
| [writing-skills](skills/writing-skills/SKILL.md) | 编写、修改技能或验证技能行为时。 |
| [diagnosing-superpowers](skills/diagnosing-superpowers/SKILL.md) | 排查 Superpowers 会话中的技能未触发、计划被跳过、重复工作等问题，或整理缺陷报告时。 |

#### 常见交接路径

- 新增功能或调整行为时，用 `brainstorming` 确认设计后交给 `writing-plans`。计划确认后，按用户选择和宿主能力进入 `executing-plans` 或 `subagent-driven-development`。两种执行方式都会通过 `using-git-worktrees` 检查工作区，并在最终审查通过后进入 `finishing-a-development-branch`。
- 修复缺陷时，从 `systematic-debugging` 开始，定位根因后用 `test-driven-development` 补复现测试并修复，最后用 `verification-before-completion` 核实结果。
- 发起代码审查时用 `requesting-code-review`，收到反馈后用 `receiving-code-review` 核实和处理。

### 宿主的安装与加载入口

| 宿主 | 入口与加载方式 |
| --- | --- |
| Codex | `.agents/plugins/marketplace.json` 定义开发市场 `superpowers-dev`，其中的插件条目指向仓库根目录。`.codex-plugin/plugin.json` 声明 `./skills/`，由 Codex 原生发现技能。显式的 `"hooks": {}` 用于关闭对 Claude Code hook 的自动发现。 [市场清单](.agents/plugins/marketplace.json) · [插件清单](.codex-plugin/plugin.json) |
| Claude Code | `.claude-plugin/plugin.json` 是插件清单。`hooks/hooks.json` 注册 `SessionStart`，通过 `run-hook.cmd` 调用 `session-start`，将 `using-superpowers` 加入会话上下文。 [插件清单](.claude-plugin/plugin.json) · [Hook 配置](hooks/hooks.json) · [Hook 启动器](hooks/run-hook.cmd) · [启动脚本](hooks/session-start) |
| Cursor | `.cursor-plugin/plugin.json` 声明 `./skills/` 和 Cursor 专用的 hook 配置。`sessionStart` 通过 `run-hook.cmd` 调用 `session-start`，将 `using-superpowers` 加入会话上下文。 [插件清单](.cursor-plugin/plugin.json) · [Hook 配置](hooks/hooks-cursor.json) · [Hook 启动器](hooks/run-hook.cmd) · [启动脚本](hooks/session-start) |
| OpenCode | `package.json` 声明包入口，根目录 `index.js` 是 OpenCode V2 的本地目录入口。两者均指向负责注册技能和注入启动指引的 JS 插件。 [包清单](package.json) · [本地入口](index.js) · [插件源码](.opencode/plugins/superpowers.js) |
| Pi | `package.json` 的 `pi` 字段声明 `skills/` 和 TS 扩展。Pi 原生发现技能，扩展通过会话事件将启动指引加入上下文。 [包清单](package.json) · [扩展源码](.pi/extensions/superpowers.ts) |
| Hermes | `.hermes-plugin/plugin.yaml` 声明插件和 `pre_llm_call` hook。Python 入口注册技能，并通过该 hook 为主会话首轮提供启动指引。 [插件清单](.hermes-plugin/plugin.yaml) · [插件源码](.hermes-plugin/__init__.py) |
| Gemini CLI | `gemini-extension.json` 通过 `contextFileName` 指定 `GEMINI.md`，由宿主加载其中引用的启动技能和工具映射。 [扩展清单](gemini-extension.json) · [上下文文件](GEMINI.md) |
| Kimi Code | `.kimi-plugin/plugin.json` 声明 `./skills/`，通过 `sessionStart.skill` 指定 `using-superpowers`，并提供工具映射。 [插件清单](.kimi-plugin/plugin.json) |

其他宿主的安装命令和入口见 [README 的安装章节](README.md#installation)和[宿主移植指南](docs/porting-to-a-new-harness.md)。

在同一台机器保留多个开发版本时，见[本地安装多个 Superpowers 开发版本](docs/local-development-installation.md)。

### 发布、测试与设计文档

- 根目录 `scripts/` 用于版本维护、Shell 检查及 Codex 发布打包和同步，`assets/` 存放插件图标。 [维护脚本](scripts/) · [版本字段清单](.version-bump.json)
- `tests/` 按宿主和功能组织插件测试，覆盖清单、启动、打包和辅助脚本。技能行为评估由单独的仓库提供，可检出到 `evals/`。 [测试指南](docs/testing.md)
- `docs/plans/` 和 `docs/superpowers/specs/` 保存设计与实施记录。追踪当前加载行为时，以仓库中的清单和入口源码为准。 [实施计划](docs/plans/) · [设计说明](docs/superpowers/specs/)

## 验证

- 运行与改动相关的检查，已有测试见 [tests/](tests/)。
- 修改技能时，跑一个能覆盖所改行为的代表性场景。修改启动或安装流程时，通过实际入口验证。
- 技能行为评估可使用 [superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals/) 的 Quorum。仅在用户明确同意后运行该评估框架。

## Git 协作规范

- **开工前检查**：使用 `git status --short --branch` 确认当前分支、工作区和暂存区状态。默认沿用当前任务分支；如果用户授权创建新分支或 Worktree，可按任务需要执行。遇到用户或其他协作者已有的修改时，必须保留原内容，按文件或差异片段隔离本次修改。

- **暂存与提交规范**：仅在获得用户提交授权时执行暂存和提交操作。每次提交围绕一个明确目的，按文件或差异片段暂存本次任务的修改，避免混入用户已有的修改。提交前使用 `git diff --cached` 核对内容，并使用 `git diff --cached --check` 检查暂存差异。

- **Git 操作授权边界**：普通代码修改、问题修复或分析任务不代表用户授权暂存或提交，默认将修改保留在工作区。若用户明确要求 Agent 自主完成开发流程（包括创建分支、Worktree 或提交 Commit），则允许在授权范围内自主执行必要的 Git 操作，无需逐次确认。无论何种模式，均不得擅自修改、清除或提交用户已有的暂存内容，也不得覆盖或丢弃用户已有的工作区修改。若无法安全隔离本次修改，应停止相关 Git 写操作并请求用户确认。

- **远端协作与历史保护**：同步上游或推送前核对远端和目标分支。解决冲突时保留双方仍需要的改动，完成后重新运行相关检查。未经明确授权，不得执行可能丢失用户修改的破坏性操作；需要改写已共享的提交历史时，先与相关协作者协调。

## Commit 编写规范

- 沿用个人分支的提交格式 `<type>: <中文摘要>`。范围能帮助定位时可用 `<type>(<scope>): <中文摘要>`。
- 常用类型`<type>`包括 `feat`（新增功能）、`fix`（修复问题）、`docs`（文档）、`refactor`（重构）、`test`（测试）和 `chore`（维护）。按提交的主要目的选择类型。
- 摘要用简洁的动词短语写明具体改动。需要补充背景、影响或验证结果时，标题后空一行再写正文。验证结果以实际运行过的检查为准。

## 交付

- 简要说明改了什么、验证结果和未验证的事项。涉及依赖、插件入口或兼容性时，交代实际影响。
- 推送、发布或向上游提交改动时，按仓库所有者的指示执行。
