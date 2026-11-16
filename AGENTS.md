# 个人开发约定

## 改动范围

- 围绕当前任务做改动，避免夹带无关修改。
- 按实际开发需要定制技能、配置和工作流。

## 项目架构

Superpowers 将一套开发工作流打包给多个编程助手使用。各宿主共享 `skills/`，通过各自的插件清单、原生技能发现或会话钩子加载技能。

### 技能与工作流

- `skills/<技能名>/SKILL.md` 定义技能。开头的 `name` 和 `description` 供宿主发现技能，正文定义具体步骤。同目录可包含按需读取的 `references/`、提示模板，以及被技能调用的 `scripts/`。 [示例技能](skills/brainstorming/SKILL.md)
- `skills/using-superpowers/SKILL.md` 说明何时检查和调用技能。`skills/using-superpowers/references/` 集中记录宿主工具和能力的差异。调整工作流时先看对应技能，调整宿主工具用法时先看该宿主的映射。 [启动技能](skills/using-superpowers/SKILL.md) · [工具映射](skills/using-superpowers/references/)

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

### 发布、测试与设计文档

- 根目录 `scripts/` 用于版本维护、Shell 检查及 Codex 发布打包和同步，`assets/` 存放插件图标。 [维护脚本](scripts/) · [版本字段清单](.version-bump.json)
- `tests/` 按宿主和功能组织插件测试，覆盖清单、启动、打包和辅助脚本。技能行为评估由单独的仓库提供，可检出到 `evals/`。 [测试指南](docs/testing.md)
- `docs/plans/` 和 `docs/superpowers/specs/` 保存设计与实施记录。追踪当前加载行为时，以仓库中的清单和入口源码为准。 [实施计划](docs/plans/) · [设计说明](docs/superpowers/specs/)

## 验证

- 运行与改动相关的检查，已有测试见 [tests/](tests/)。
- 修改技能时，跑一个能覆盖所改行为的代表性场景。修改启动或安装流程时，通过实际入口验证。
- 技能行为评估可使用 [superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals/) 的 Quorum。仅在用户明确同意后运行该评估框架。

## 交付

- 简要说明改了什么、验证结果和未验证的事项。涉及依赖、插件入口或兼容性时，交代实际影响。
- 推送、发布或向上游提交改动时，按仓库所有者的指示执行。
