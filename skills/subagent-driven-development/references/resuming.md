# 从现有账本恢复执行

属于 superpowers:subagent-driven-development。本计划的工作目录里已有
`<workspace>/progress.md`。

如果账本首行指向本计划文件，有 `Task <N>: complete` 记录的任务都已
完成（DONE）。从第一项尚无完成记录的任务继续。某项任务的最后一条记录
若是修复轮次，说明修复循环还在进行，从下一轮继续。如果账本首行指向
其他计划，或者在旧的平铺路径 `.superpowers/sdd/progress.md` 发现遗留
账本，将它留在原处，为当前计划新建账本。
