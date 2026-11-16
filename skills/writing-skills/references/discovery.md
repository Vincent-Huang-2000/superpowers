# 编写易于发现的 SKILL.md

这是 superpowers:writing-skills 的参考资料。它提供 SKILL.md 模板，并说明如何为技能命名、编写描述和控制篇幅，让代理能找到并阅读技能。

## SKILL.md 结构

**YAML 前置元数据**

- 必须包含 `name` 和 `description` 两个字段。完整字段列表见 [agentskills.io/specification](https://agentskills.io/specification)。
- 总长度最多 1024 字符。
- `name` 只能使用字母、数字和连字符，不能包含括号或其他特殊字符。
- `description` 用第三人称，只描述何时使用。
  - 中文以「适用于……」开头，把重点放在触发条件上。
  - 写明具体症状、场景和上下文。
  - **绝不能概述技能的操作过程或工作流程。** 原因见下文「优化技能发现」。
  - 尽可能少于 500 字符。

```markdown
---
name: Skill-Name-With-Hyphens
description: 适用于 [specific triggering conditions and symptoms]
---

# 技能名称

## 概览
用 1 至 2 句话说明技能是什么、核心原则是什么。

## 适用条件
[Small inline flowchart IF decision non-obvious]

用项目符号列出症状和使用场景。
写明哪些情况不适用。

## 核心模式（适用于技术或模式类技能）
展示使用前后的代码对比。

## 快速参考
用表格或列表呈现常见操作，便于扫读。

## 实施
简单模式的代码直接写在正文中。
长篇参考资料或可复用工具用文件链接。

## 常见错误
写明可能出错的地方及修复方式。

## 实际效果（可选）
提供具体结果。
```

## 优化技能发现（SDO）

未来的代理首先要能找到这项技能。

### 1. 写出具体的描述

代理读取描述来决定当前任务要加载哪些技能。写描述时，要让代理能回答「现在需要读这个技能吗？」

中文以「适用于……」开头，聚焦触发条件。下例沿用英文 `Use when...` 来展示英文描述中的写法。

**描述只能写何时使用。** 不得在描述中概述技能的过程或工作流程。

测试发现，代理可能直接按描述里的工作流程行动，跳过技能正文。某项技能的描述只说「在任务之间进行代码审查」，结果代理只做了一次审查，尽管流程图明明要求做两次，先检查规格符合性，再检查代码质量。

将描述改为只写「适用于执行包含独立任务的实施计划时」后，代理正确阅读了流程图，遵循了两阶段审查流程。

把流程写进描述，相当于给了代理一条捷径，它可能因此略过技能正文。

```yaml
# ❌ 不佳：概述了流程，代理可能直接照做，跳过技能正文
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ❌ 不佳：包含过多流程细节
description: Use for TDD - write test first, watch it fail, write minimal code, refactor

# ✅ 正确：只写触发条件，省略工作流程
description: Use when executing implementation plans with independent tasks in the current session

# ✅ 正确：只写触发条件
description: Use when implementing any feature or bugfix, before writing implementation code
```

编写内容时：

- 用具体的触发条件、症状和场景说明何时适用。
- 描述问题本身（如竞态条件、行为不一致）。`setTimeout`、`sleep` 之类的语言相关表现通常不宜作为触发条件。
- 除非技能本身针对特定技术，否则让触发条件独立于具体技术。
- 如果技能确实针对特定技术，就在触发条件中明确写出。
- 使用第三人称。描述会注入系统提示词。
- **无论如何，都不能概述技能的操作过程或工作流程。**

```yaml
# ❌ 不佳：太抽象、含糊，没有说明何时使用
description: For async testing

# ❌ 不佳：第一人称
description: I can help you with async tests when they're flaky

# ❌ 不佳：提到某项技术，但技能并不限于它
description: Use when tests use setTimeout/sleep and are flaky

# ✅ 正确：以「Use when」开头，描述问题，省略流程
description: Use when tests have race conditions, timing dependencies, or pass/fail inconsistently

# ✅ 正确：针对特定技术的技能，触发条件明确
description: Use when using React Router and handling authentication redirects
```

### 2. 覆盖常用搜索词

采用代理实际会搜索的词：

- 错误消息或术语，如 `Hook timed out`、`ENOTEMPTY`、`race condition`。
- 症状，如 `flaky`、`hanging`、`zombie`、`pollution`。
- 同义词，如 `timeout/hang/freeze`、`cleanup/teardown/afterEach`。
- 工具名称，如实际命令、库名和文件类型。

### 3. 用名称说明用途

使用主动语态，把动词放在最前面：

- ✅ `creating-skills`　❌ `skill-creation`
- ✅ `condition-based-waiting`　❌ `async-test-helpers`

### 4. 节省 token（重要）

入门技能和频繁引用的技能会载入每一场对话。每个 token 都占用上下文。

按英文单词计的目标长度：

- 入门流程每项少于 150 词。
- 频繁加载的技能合计少于 200 词。
- 其他技能少于 500 词，仍要尽量简洁。

把细节放到工具的帮助信息中：

```bash
# ❌ 不佳：在 SKILL.md 中逐一记录参数
search-conversations 支持 --text、--both、--after DATE、--before DATE、--limit N

# ✅ 正确：引用 --help
search-conversations 支持多种模式和过滤条件。运行 --help 查看细节。
```

使用交叉引用：

```markdown
# ❌ 不佳：重复工作流程
搜索时，使用模板派发子代理……
[20 lines of repeated instructions]

# ✅ 正确：引用其他技能
始终使用子代理（节省 50 至 100 倍上下文）。必须使用 [other-skill-name] 了解工作流程。
```

压缩示例。以下括号里的词数对应英文原例：

```markdown
# ❌ 不佳：冗长的示例（42 词）
用户：「我们以前是怎么在 React Router 中处理身份验证错误的？」
你：我会搜索之前的对话，查找 React Router 身份验证相关做法。
[派发子代理，搜索查询为「React Router authentication error handling 401」]

# ✅ 正确：简洁的示例（20 词）
用户：「React Router 的身份验证错误是怎么处理的？」
你：正在搜索……
[派发子代理 → 汇总]
```

删除冗余：

- 不复述已交叉引用的技能内容。
- 不解释从命令就能看出的用法。
- 同一种模式只保留一个示例。

验证词数：

```bash
wc -w skills/path/SKILL.md
# 入门流程：每项尽量少于 150 词
# 其他频繁加载的技能：合计尽量少于 200 词
```

用行动或核心见解来命名：

- ✅ `condition-based-waiting`　❌ `async-test-helpers`
- ✅ `using-skills`　❌ `skill-usage`
- ✅ `flatten-with-flags`　❌ `data-structure-refactoring`
- ✅ `root-cause-tracing`　❌ `debugging-techniques`

动名词（以 `-ing` 结尾）很适合给流程命名。`creating-skills`、`testing-skills` 和 `debugging-with-logs` 都直接说明了要做什么。

### 5. 交叉引用其他技能

在文档中引用其他技能时，仅使用技能名，并明确标注是否必需：

- ✅ 正确：`**必用子技能**：使用 superpowers:test-driven-development`
- ✅ 正确：`**必备知识**：必须理解 superpowers:systematic-debugging`
- ❌ 不佳：`参见 skills/testing/test-driven-development`，没说明是否必需。
- ❌ 不佳：`@skills/testing/test-driven-development/SKILL.md`，会强制加载，消耗上下文。

避免用 `@` 引用。它会立即强制加载文件，实际用到之前就可能消耗超过 20 万 token 的上下文。
