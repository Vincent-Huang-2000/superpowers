# Hermes Agent 工具映射

技能以「派发子代理」「创建待办」「读取文件」之类的操作描述流程。在 Hermes Agent 中使用下表所列的工具。

## 工具

| 技能要求的操作 | Hermes 工具 |
| --- | --- |
| 读取文件 | `read_file` |
| 创建新文件 | `write_file` |
| 编辑文件（定点修改） | `patch` |
| 运行 Shell 命令 | `terminal` |
| 搜索文件内容 | `search_files` |
| 按名称查找文件 | 在 `terminal` 中运行 `find` |
| 获取网页 | `web_extract(urls=[...])` |
| 搜索网页 | `web_search(query=...)` |
| 派发子代理 | `delegate_task(goal=..., context=..., toolsets=[...], role="leaf")` |
| 跟踪任务 | `todo` |
| 调用技能 | `skill_view("skill-name")` |

## 指令文件

技能提到的「指令文件」在 Hermes Agent 中指项目目录下的 **`AGENTS.md`**，全局指令文件位于 **`~/.hermes/SOUL.md`**。

## 调用技能

Hermes Agent 的 `skills` 工具集提供 `skill_view` 和 `skills_list`。按以下方式调用 Superpowers 技能。

```
skill_view("brainstorming")
skill_view("test-driven-development")
```

如果 `skill_view` 找不到 Superpowers 技能，例如插件尚未注册完毕、技能还未出现在目录里，可直接读取 `SKILL.md`。

```
read_file(path="~/.hermes/plugins/superpowers/skills/<skill-name>/SKILL.md")
```

其他缺少原生技能加载机制的宿主也可以通过直接读取文件来加载。

## 派发子代理

用 `delegate_task` 派发隔离的子代理，按需要并行或依次安排任务。

```
delegate_task(goal="...", context="...", toolsets=[...], role="leaf")
```

如果 `delegate_task` 不可用，由当前会话执行工作。

## 跟踪任务

使用 `todo` 跟踪会话内的任务。多代理任务看板可用 `hermes kanban` 命令（如已提供）。旧文档中的 `TodoWrite` 对应这里的跟踪任务操作。
