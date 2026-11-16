# 最终审查与收尾

属于 superpowers:subagent-driven-development。此时计划中的所有任务均已完成。

整个分支的最终审查也需要审查材料。在执行计划的工作树中运行
本技能的 `bash scripts/review-package PLAN_FILE MERGE_BASE HEAD`（使用脚本的绝对路径）。MERGE_BASE
是创建分支时的基准提交，例如用 `git merge-base main HEAD` 获取。派发时附上
脚本打印的路径，这样最终审查者读一个文件即可获取分支差异。使用目前能力最强的
模型（见 SKILL.md 的「模型选择」），并使用 superpowers:requesting-code-review
的 [code-reviewer.md](../../requesting-code-review/code-reviewer.md)。指出账本中
暂缓的轻微问题和已搁置的发现，请审查者判断哪些必须在合并前修正。

如果最终审查提出问题，把完整清单交给**一个**修复子代理。逐个问题派发会让每个
代理都得重新理解背景、重跑测试套件。有一次这样修复的成本超过了执行所有任务的
总成本。修复之后只做一次针对这批改动的复审。在执行计划的工作树中运行本技能的
`bash scripts/review-package PLAN_FILE FIX_BASE HEAD`（使用脚本的绝对路径），打包本轮
修复的范围，并使用 [re-review-prompt.md](../re-review-prompt.md) 派发。仍有
未解决的问题时，按修复循环达到上限后的规则（references/fix-loop.md）处理。
可以写明裁定后搁置，也可以就影响后续工作的发现作出裁定并记入账本。SKILL.md
开头列出的四项停止条件仍然适用。修复只做这一轮。如果仍有影响后续工作的问题，
在 finishing-a-development-branch 提出收尾选项时向用户说明。

### 收尾

删除任何内容前，把账本里每条含 `Ruling:` 的记录全部汇入最终消息的
「Rulings I made」小节。这包括执行前检查的裁定、搁置问题的裁定，以及修复
达到上限后的裁定。按裁定发生的顺序列出，每条都说明判断错误的代价。
用户会据此了解你代为作出的所有决定，并修正不合适的处理。

整个分支最终审查通过且修复已合入时，删除本计划的工作目录
（`rm -rf <workspace>`），后续以 Git 历史为记录。其他计划的同级目录原样保留。

使用 superpowers:finishing-a-development-branch。
