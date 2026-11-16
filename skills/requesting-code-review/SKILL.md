---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# 发起代码审查

派发代码审查子代理，趁问题还未影响后续工作时发现它们。只为审查者提供
评估所需的背景，禁止把当前会话历史交给它。

**核心原则** 尽早审查，及时复审。

## 何时发起审查

**必须审查**

- 子代理驱动开发中的每项任务完成后
- 主要功能实施完成后
- 合并到 main 前

**可按需审查**

- 工作受阻时，借助独立视角
- 重构前，检查现有行为
- 修复复杂缺陷后

## 如何发起审查

**1. 获取 Git 提交 SHA**

```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or: git merge-base origin/main HEAD
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发代码审查子代理**

使用 [code-reviewer.md](code-reviewer.md) 的模板，填写其中占位符，
派发 `general-purpose` 子代理。

**占位符**

- `{DESCRIPTION}` 简述实现内容。
- `{PLAN_OR_REQUIREMENTS}` 写明预期行为。
- `{BASE_SHA}` 起始提交。
- `{HEAD_SHA}` 结束提交。

**3. 处理反馈**

- 立即修复 Critical 级问题。
- 继续前先修复 Important 级问题。
- 记录 Minor 级问题，留待以后处理。
- 如果审查者判断有误，说明技术理由并提出异议。

## 示例

```
[刚完成第 2 项任务：添加验证函数]

你：继续之前，先请审查者检查代码。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | cut -d' ' -f1)
HEAD_SHA=$(git rev-parse HEAD)

[派发代码审查子代理]
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，涵盖 4 类问题
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的第 2 项任务
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[子代理返回]
  优点：架构清晰，测试验证了实际行为
  问题：
    Important：缺少进度提示
    Minor：报告间隔使用魔法数字（100）
  评估：可以继续

你：[修复缺少进度提示的问题]
[继续第 3 项任务]
```

## 常见借口及应对

| 借口 | 实际要求 |
|--------|---------|
| 「我自己读差异就行，不用派发审查者」 | 你负责协调工作。直接审查差异会占用继续推进所需的上下文窗口。派发审查子代理，让差异和评估过程留在它的上下文中，由它返回发现。 |
| 「审查者必须看完整会话历史才能看懂改动」 | 只提供审查所需的背景。这样审查者会专注于交付成果。 |

## 遇到警示信号就停下

**禁止以下做法**

- 因为改动「很简单」就跳过审查
- 忽略 Critical 级问题
- 在 Important 级问题尚未修复时继续
- 争辩以回避合理的技术反馈

**审查者判断有误时**

- 说明技术理由，提出异议。
- 指出证明代码正常工作的实现或测试。
- 请审查者澄清。

模板见 [code-reviewer.md](code-reviewer.md)。
