---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring skill invocation before ANY response including clarifying questions
---

<SUBAGENT-STOP>
如果你是受派执行具体任务的子代理，请忽略本技能。
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
只要你认为某项技能哪怕有 1% 的可能适用于当前工作，就必须调用它。

凡是适用于当前任务的技能，都必须使用。

这项要求没有商量余地。别找理由绕过它。
</EXTREMELY-IMPORTANT>

## 技能调用规则

**在任何回复或行动之前，先调用相关技能或用户指定的技能**，包括提出澄清问题、探索代码库或检查文件之前。如果读完发现技能不适用于当前情况，可以停止按该技能执行。

**进入计划模式前** 如果还没有做过构思，先调用 brainstorming 技能。

随后告知用户「我会用 [技能] 来 [完成的事项]」，并严格遵循该技能。如果技能包含检查清单，为每一项创建一条待办。

## 技能优先级

有多项技能适用时，先用流程技能确定做法，再用 frontend-design 等实施技能执行。brainstorming 和 systematic-debugging 是 Superpowers 最常用的流程技能，这个先后顺序适用于所有流程技能。

- 「来做 X 吧」→ 先用 superpowers:brainstorming，再用实施技能。
- 「修复这个缺陷」→ 先用 superpowers:systematic-debugging，再用对应领域的技能。

## 遇到警示信号就停下

出现以下想法时，先停下来。你正在给自己找借口。

| 想法 | 正确做法 |
|---------|---------|
| 「这只是个简单问题」 | 回答问题也是任务。先检查适用的技能。 |
| 「我得先问清楚背景」 | 先检查技能，再提出澄清问题。 |
| 「我先看看代码库」 | 技能会告诉你怎么探索。先检查技能。 |
| 「我很快就能查一下 Git 或文件」 | 文件不包含对话上下文。先检查技能。 |
| 「我先收集一点信息」 | 技能会告诉你怎么收集信息。 |
| 「这用不着专门的技能」 | 有适用的技能就使用它。 |
| 「我记得这个技能」 | 技能会更新。读取当前版本。 |
| 「这不算任务」 | 采取行动就是在处理任务。先检查技能。 |
| 「用技能太小题大做了」 | 简单的事也会变复杂。使用技能。 |
| 「我就先做这一件事」 | 做任何事之前先检查技能。 |
| 「这样感觉挺有效率」 | 缺少方法的行动会浪费时间，技能可以避免这一点。 |
| 「我知道那是什么意思」 | 理解概念之后，仍要实际调用技能。 |

## 宿主适配

如果你使用的是下列宿主之一，请阅读对应的参考文件，了解该宿主的专用指引。

- Claude Code: `references/claude-code-tools.md`
- Codex: `references/codex-tools.md`
- Pi: `references/pi-tools.md`
- Antigravity: `references/antigravity-tools.md`
- Hermes Agent: `references/hermes-tools.md`
- Muse: `references/muse-tools.md`

## 用户指令

用户指令（CLAUDE.md、AGENTS.md、GEMINI.md 等文件中的指令或用户的直接请求）优先于技能，技能又优先于默认行为。只有用户明确要求时，才跳过技能的流程或指令。
