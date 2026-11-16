# Muse 工具映射

技能以「派发子代理」「创建待办」「读取文件」之类的操作描述流程。在 Muse 中使用下表所列的工具。

| 技能要求的操作 | Muse 对应工具 |
| --- | --- |
| 读取文件 | `read_file` |
| 读取多个文件 | 多次调用 `read_file` 或使用 `search` |
| 创建新文件 | `write_file` |
| 编辑文件 | `edit_file` |
| 运行 Shell 命令 | `bash` |
| 搜索文件内容 | `search` |
| 按名称查找文件 | 使用 `search` 的 `glob` 功能 |
| 获取网页 | `web_fetch` |
| 搜索网页 | `web_search` |
| 调用技能 | 通过 `read_file` 读取 `skills/<name>/SKILL.md`，也可用原生技能工具 |
| 派发子代理（`Subagent (general-purpose):` 模板） | 填写提示模板后调用 `subagent_spawn` |
| 跟踪任务（创建待办、标记完成） | `write_todos`，或用 `bash` 管理任务文件 |
| 向用户提问 | `request_user_input` |

## 指令文件

技能提到的「指令文件」在 Muse 中指项目根目录的 **`CLAUDE.md`** 或 **`AGENTS.md`**。配置了分层加载时，Muse 会按层级读入这些文件。

## 调用技能

Muse 通过 `muse skills` 提供原生技能支持。调用 Superpowers 技能时，读取并遵循对应的 `SKILL.md`。启动技能 `using-superpowers` 已在 `SessionStart` 由插件钩子自动注入，可以直接遵循。

## 派发子代理

用 `subagent_spawn` 将工作派给隔离的子代理。派发前填好提示模板，例如 `implementer-prompt.md` 或 `task-reviewer-prompt.md`。如果没有子代理工具，在当前会话执行。

## 跟踪任务

用 `write_todos` 管理清单。为技能清单的每一项创建待办，随进度标记为 `in_progress` 或 `completed`。如果 `write_todos` 不可用，用 `write_file` 或 `edit_file` 维护 Markdown 任务清单。
