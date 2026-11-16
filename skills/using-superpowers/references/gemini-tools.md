# Gemini CLI 工具映射

技能以「派发子代理」「创建待办」「读取文件」之类的操作描述流程。在 Gemini CLI 中使用下表所列的工具。

| 技能要求的操作 | Gemini CLI 对应工具 |
| --- | --- |
| 读取一个文件 | `read_file` |
| 一次读取多个文件 | `read_many_files` |
| 创建新文件 | `write_file` |
| 编辑文件 | `replace` |
| 运行 Shell 命令 | `run_shell_command` |
| 搜索文件内容 | `grep_search` |
| 按名称查找文件 | `glob` |
| 列出文件和子目录 | `list_directory` |
| 获取网页 | `web_fetch` |
| 搜索网页 | `google_web_search` |
| 调用技能 | `activate_skill` |
| 派发子代理（`Subagent (general-purpose):` 模板） | 使用 `invoke_agent` 并设置 `agent_name: "generalist"`，也可以用 `@generalist` 聊天语法，参见[子代理支持](#subagent-support)。 |
| 并行派发多项任务 | 在同一条回复中多次调用 `invoke_agent` |
| 跟踪任务（创建待办、标记完成） | `write_todos`，状态为 `pending`、`in_progress`、`completed`、`cancelled`、`blocked` |

## 指令文件

技能提到的「指令文件」在 Gemini CLI 中指 **`GEMINI.md`**。它按层级加载这些文件。全局路径是 `~/.gemini/GEMINI.md`，项目级文件位于工作区及其上级目录，工具访问子目录中的文件时还会加载该子目录的指令文件。

## 个人技能目录

用户级技能在 **`~/.gemini/skills/`**，也可使用 **`~/.agents/skills/`** 这一跨宿主路径（与 Codex 和 Copilot CLI 共享）。同一作用域中两个目录都存在时，`.agents/skills/` 优先。每个技能各占一个子目录，内含 `SKILL.md`，其 YAML 头部包含 `name` 和 `description`。

<a id="subagent-support"></a>

## 子代理支持

Gemini CLI 通过 `invoke_agent` 派发子代理，参数为 `agent_name` 和 `prompt`。聊天也提供等效快捷方式，输入 `@generalist <prompt>` 即可。内置代理包括 `generalist`、`cli_help`、`codebase_investigator`，启用浏览器工具后还有 `browser_agent`。

技能通过 `Subagent (general-purpose):` 派发，引用提示模板文件（如 `superpowers:subagent-driven-development` 的 `./implementer-prompt.md`），或直接提供提示词。在 Gemini CLI 中按下表处理。

| 技能采用的派发形式 | Gemini CLI 对应方式 |
| --- | --- |
| 引用 `*-prompt.md` 模板（如实施者、任务审查者、代码审查者） | 填完模板后调用 `invoke_agent`，传入 `agent_name: "generalist"` 和完整的提示词。 |
| 引用 `superpowers:requesting-code-review` 的 `./code-reviewer.md` | 调用 `invoke_agent`，传入 `agent_name: "generalist"` 和填完的审查模板。 |
| 直接给出提示词（未引用模板） | 调用 `invoke_agent`，传入 `agent_name: "generalist"` 和直接给出的提示词。 |

### 填写提示模板

技能模板含有 `{WHAT_WAS_IMPLEMENTED}` 或 `[FULL TEXT of task]` 等占位符。填完所有占位符后，再把完整提示词传给 `invoke_agent`。模板已经规定代理角色、审查标准和输出格式，子代理会按模板执行。

### 并行派发

Gemini CLI 支持并行派发子代理。在同一条回复中多次调用 `invoke_agent`，或在一条提示消息中多次使用 `@generalist`，即可并行执行独立的工作。有依赖的任务按顺序派发，其余任务可并行派发。

## Gemini CLI 的其他工具

| 工具 | 用途 |
| --- | --- |
| `save_memory`（旧版） | 在 `experimental.memoryV2 = false` 时跨会话保存信息 |
| `get_internal_docs` | 查阅 Gemini CLI 随附的文档 |
| `ask_user` | 向用户提出结构化问题（文本、单选或多选） |
| `enter_plan_mode` / `exit_plan_mode` | 进入或退出只读计划模式 |
| `update_topic` | 更新当前对话的主题或意图元数据 |
| `complete_task` | 标记 Gemini 子代理任务完成，并向上级代理返回结果 |
| `tracker_create_task`、`tracker_update_task`、`tracker_get_task`、`tracker_list_tasks`、`tracker_add_dependency`、`tracker_visualize` | 支持任务依赖及可视化的任务跟踪工具 |
| `read_mcp_resource`、`list_mcp_resources` | 访问 MCP 资源 |
