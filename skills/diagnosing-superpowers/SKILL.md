---
name: diagnosing-superpowers
description: Use when a superpowers session went wrong and your human partner wants to know why — repeated work, ignored plans, stumbles, poor results, a skill that didn't fire, "it took too long", "why is it so expensive", "what is it doing" — or wants to build a bug report for the superpowers maintainers, for the current session or a past one identified by id or path, on any harness.
---

# 排查 Superpowers 会话

## 概述

和与你协作的人确认会话(session)出了什么问题，读取磁盘上的会话记录，并据实报告。报告只记录发生了什么，是否需要调整 Superpowers 由诊断包或ISSUES的处理者决定。

**核心原则** 每条发现都必须引用 `path:line`，缺少引用就省略。所有数字都来自会话记录或实际运行过的命令。

## 工作流程

每一步建立一条待办。第 5–7 步只在满足各自的条件时执行。

1. **确认问题。** 每次问一个问题，直到能写出问题陈述。写明涉及的会话、已知的对话轮次范围、对方的预期、实际情况，以及对方关心的可观察现象（实际耗时、token 用量、重复操作或某项具体操作）。「花了太久」还需要补充具体信息。记下目标是否为编写 Superpowers 缺陷报告。
2. **定位会话。** 按 `references/session-discovery.md` 为每个会话找出经过核实的绝对文件路径。对于历史会话，引用首条提示和时间戳来确认身份，并列出每个已排除的候选及原因。没有候选被排除时填 `none`。列出子代理的会话记录。创建 `~/.superpowers/diagnosing-superpowers/<session-id>/`，告知对方路径，并在该目录中填写 `templates/case.md`，按模板要求记录环境和技能观察的证据来源。
3. **分析。** 先自行阅读报告问题所在的片段。再为每个分析维度并行派发一名子代理，提供案例文件路径、`prompts/analyst-common.md` 和相应的维度提示文件。七个维度分别为 `skill-timeline.md`、`plan-adherence.md`、`repeated-work.md`、`stumbles.md`、`quality-evidence.md`、`request-conflicts.md` 和 `cost-and-time.md`，均位于 `prompts/`。会话记录较长时，按轮次范围拆分维度。丢弃缺少 `path:line` 的发现。
4. **写报告。** 按顺序填写 `templates/report.md` 的各节，将报告写入工作区并展示全文和路径。核实引用内容实际能证明什么，并保留作为依据的案例文件。符号链接指向的是同一份数据，无法充当备份。
5. **查找 GitHub ISSUES。** 当报告 §7 的判定为 `possible` 或 `likely`，或对方提出要求时，按 `references/github-issues.md` 搜索与症状有关的开放和已关闭ISSUES。展示匹配项，并建议把报告添加到最接近的ISSUES。没有匹配项时，填写 `templates/issue.md`，写入工作区并展示拟提交的原文，获得批准后才创建ISSUES。`gh` 无法上传附件。已有诊断包时，把路径发给对方，让对方在浏览器中上传。
6. **导出。** 只有对方提出要诊断包时才制作。如果最初的目标是编写缺陷报告，只需告知一次可应请求提供脱敏诊断包，然后等待。询问所需的脱敏级别并说明各级别包含的内容。`skeleton` 不含工具结果正文，`evidence` 只保留被引用事件的工具结果正文，`full` 保留全部正文。按 `templates/bundle-README.md` 制作诊断包，依次派发 `prompts/scrub.md` 和 `prompts/scrub-audit.md`，重复两者直至审计返回 `CLEAN`。完成诊断包模板规定的证据检查和一致性核对后，展示最终脱敏日志、文件清单、隐私审计结果和证据核查结果。仅在获得批准后打包（`zip -r` 或 `tar -czf`）。给出归档路径时，说明其中的内容、指出脱敏日志中的替换记录，并告知对方脱敏可能有遗漏，分享前必须逐一检查每个文件。
7. **查找相似会话。** 对方提出要求时，将已确认的发现归纳为匹配特征，按修改时间和大小列出候选，找出标记所在的行号，对每个候选并行派发 `prompts/similar-session.md`，并追加报告 §9。

## 速查

七位分析子代理均须派发。下表用于确定第 3 步优先亲自查看的片段，以及在结论中优先呈现的发现。

| 问题描述 | 优先查看和报告 |
|---|---|
| 「花了太久」 | cost-and-time、stumbles |
| 「为什么做了这些多余的事？」 | repeated-work、plan-adherence |
| 「为什么这么贵？」 | cost-and-time |
| 「它到底在干什么？」（会话仍在运行） | skill-timeline，在覆盖范围中注明会话还在进行 |
| 「它没按计划做」 | plan-adherence，先看上下文压缩所在的行 |
| 「某个技能从没触发」 | skill-timeline |

## 硬性规则

- **保护上下文。** 单条会话记录可能长达 1 MB。每次读取任何会话文件时都遵循 `references/context-safety.md`。
- **只读。** 严禁修改、移动或删除会话文件。
- **给子代理准确的路径。** 子代理所指的「当前会话」是它自己的会话。传递绝对路径和会话 ID。
- **只将人工输入视作对方的原话。** 钩子输出、系统提醒和工具结果都不是对方的输入。子代理会话记录中的 `user` 指父代理。
- **仅记录 Superpowers 的参与程度。** 报告 §7 到此为止，禁止认定技能缺陷或提出改动。即使对方催促修复，也只引导对方查看提交ISSUES的步骤，并说明可应请求提供诊断包。不要向对方提出处理建议。
- **取得批准后再操作。** 对方看过脱敏日志和文件清单前不得打包。创建ISSUES或发表评论前，必须获得对确切文本的批准，摘要必须由实际内容填好。
- **先确认问题再分析。** 在对方回答前，第 2–7 步都不得开始。对方不在时，写下问题并停止。自行重建的问题陈述无法代替对方回答。如果请求已限定范围，比如问某个具体事件、当前正在运行什么，或指定要做哪项分析，请求本身就是问题陈述，先作答，再提问。笼统地问整个会话「为什么会这样」仍需澄清。

## 警示信号

| 想法 | 应采取的做法 |
|---------|---------|
| 「问题显而易见，跳过确认」 | 问题陈述决定分析范围。先提问。 |
| 「对方不在，我可以自己补全问题陈述」 | 无法确定对方的预期。写下问题并停止。 |
| 「我先全面查一遍，最后再问」 | 没有明确范围就全面搜查会把预算花在错误的问题上。先提问。 |
| 「对方想要缺陷报告，先把包做出来」 | 诊断包包含对方的会话数据。只有对方明确要求才制作。 |
| 「只是一个小改动，无需调整结构」 | 改动再小也由处理者决定。只报告证据。 |
| 「每个 token 的价格大家都知道」 | 只使用根据会话记录算出的数字。给出引用，否则省略。 |
