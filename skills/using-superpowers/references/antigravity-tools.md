# Antigravity CLI（`agy`）工具映射

技能以「派发子代理」「创建待办」「读取文件」之类的操作描述流程。在 Antigravity CLI（`agy`）中使用下表所列的工具。

| 技能要求的操作 | Antigravity CLI 对应工具 |
| --- | --- |
| 派发子代理（`Subagent (general-purpose):` 模板） | 使用 `invoke_subagent`，指定内置的 `TypeName`。完整权限用 `self`，只读任务用 `research`。 |
| 跟踪任务（创建待办、标记完成） | 用 `write_to_file` 创建[任务工件](#task-tracking)，设置 `IsArtifact: true` 和 `ArtifactType: "task"`。`manage_task` 用于管理后台进程。 |

<a id="task-tracking"></a>

## 跟踪任务

Antigravity 没有待办清单工具。`manage_task` 的 `list`、`kill`、`status` 和 `send_input` 操作用于管理后台进程。技能要求创建待办清单或跟踪任务时，用 `write_to_file` 保存 Markdown 清单作为**任务工件**，设置 `IsArtifact: true` 和 `ArtifactMetadata.ArtifactType: "task"`。进行中使用 `replace_file_content` 或 `multi_replace_file_content` 更新。

开始多步骤任务时，创建任务工件，列出计划的每一步。完成一步就标记为完成（`- [x]`）。计划变更时更新清单，以清单为准确认剩余工作。对话较长时，每开始一步前重读清单。
