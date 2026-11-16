Title: <skill or symptom>: <one-line observable> (<harness>)

- [x] I searched existing issues and this is not a duplicate（搜索词：<query terms>，最接近的议题：<#n title, or "none">）

## Who's filing this report? (required)

<本会话的模型和宿主> 代表 <对方希望使用的称呼，或「与我协作的人」> 提交。对方的请求为「<本会话中促成诊断或议题提交的请求，已脱敏>」。

## Human-authored summary (required, under 200 words)

<粘贴对方对问题和预期行为的原话。向对方索取原文，这里仍有占位内容时不得提交议题。如果字数过长，请对方自行缩短。>

## Environment (required)

| Field | Value | 证据来源类型及证据 |
|-------|-------|-------------------------------|
| Superpowers version | <version> (<sha or "not a checkout">) | <historical evidence / unverified snapshot / current observation / unknown>，<location> |
| Harness (Claude Code, Cursor, etc.) | <harness> | <label>，<location> |
| Harness version | <version> | <label>，<location> |
| Your model + version | <model ids seen> | <label>，<location> |
| All plugins installed | <list> | <label>，<location> |
| OS + shell | <os version>, <shell> | <label>，<location> |

## Is this a Superpowers issue or a platform issue?

- [ ] I confirmed this issue does not occur without Superpowers installed

How I verified that this issue does not reproduce without Superpowers:
<填 "Not tried"，或说明对方在未安装 Superpowers 时运行了什么、发生了什么。有会话记录时附上路径。只有会话记录显示问题确实未复现，才能勾选上面的选项。>

下文列出 Superpowers 参与的证据。判定原因需要进一步证据。

## What happened?

<先写问题陈述，再写分析结论。将 `path:line` 引用改写为 `transcript line <n>`。>

## Steps to reproduce

1. <首条人工提示，已脱敏>
2. <导致问题的各轮操作，每轮一行>
3. <可观察现象>

## Expected behavior

<取自问题陈述。>

## Actual behavior

<取自分析结论。>

## Debug log or conversation transcript

会话 ID：<ids>。已交付的本地归档：<path, redaction level <level> | none built>。诊断包附件：<不预先声明已上传，仅在获批后上传>。
报告判定的 Superpowers 参与程度：<possible | likely>，证据见 <transcript lines>。本报告仅陈述发现。

---
通过 `diagnosing-superpowers` 技能提交。提交方的模型和宿主见「Who's filing」，被排查会话的模型、宿主、版本和插件见「Environment」表。
