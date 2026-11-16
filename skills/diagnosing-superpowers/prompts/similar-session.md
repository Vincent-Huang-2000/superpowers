你负责匹配候选会话。判断一份候选会话记录中是否出现了被排查会话中的相同行为。不得修改任何文件。

输入如下。
- CASE：被排查会话的案例文件绝对路径。先读该文件，了解上下文保护规则、已确认的记录含义，以及要使用的提取命令。
- CANDIDATE：要检查的一份候选会话记录的绝对路径。
- SIGNATURE：用于匹配的标记清单。每项为以下一种。
  - `skill-sequence: <skill A> then <skill B> within <n> turns`（在指定轮次内依次调用两项技能）
  - `error-string: "<text>"`（错误文字）
  - `repeated-command: "<command>" ≥ <n> times`（命令重复次数）
  - `repeated-file: <path pattern> read ≥ <n> times`（匹配指定模式的路径被重复读取的次数）
  - `compaction-then: <behavior described in one line>`（压缩后出现的行为）
  - `missed-trigger: <skill> for requests matching "<text>"`（请求匹配指定文字时未触发的技能）
  - `free: <one-line description>`（自由描述，仅依据会话记录判断）

操作步骤如下。
1. 对 CANDIDATE 应用 `references/context-safety.md`。使用 CASE 记录的命令提取识别信息，包括会话 ID、工作目录、首条人工提示、首个时间戳、宿主版本和模型。
2. 对每个标记，先用只取行号的命令定位证据，再从指定行提取截短的字段。有 `path:line` 证据时记为 `hit`，搜索后未找到记为 `miss`，会话记录缺少所需字段时记为 `unknown`，并说明缺少哪个字段。
3. 严格按以下格式返回。

```
candidate: <session id> — <absolute path>
identity: <harness> <version>, <first timestamp>, "<first prompt, 100 chars>"
match: yes | partial | no
markers:
- <marker>: hit — <path>:<line> — "<quote ≤ 120 chars>"
- <marker>: miss — checked <what>
- <marker>: unknown — <missing field>
```

全部命中记为 `yes`，部分命中且至少命中一项记为 `partial`，没有命中记为 `no`。
