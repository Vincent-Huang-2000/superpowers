# Superpowers 会话诊断包

会话：<session-id>
宿主：<name> <version> (<provenance label>)    Superpowers：<version> (<sha or "not a checkout">, <provenance label>)
脱敏级别：skeleton | evidence | full
制作时间：<ISO timestamp>

为页首的版本字段标明证据来源类型。可选类型为 `historical evidence`（历史证据）、`unverified snapshot`（未验证的快照）、`current observation`（当前观察）和 `unknown`（未知）。`environment.json` 对每个环境字段也记录相同的来源类型及支持证据的位置。

## 关于诊断包

本诊断包包含一份安装了 Superpowers 且出现问题的编程代理会话记录，已经脱敏。报告使用 `path:line` 证据说明发生了什么。未参与该会话的代理或人员可据此判断 Superpowers 是否参与了问题，以及是否需要改动和如何改动。报告只记录事实，将缺陷诊断和修复方案留给读者决定。

## 文件

- `report.md` 为会话诊断报告，包含问题陈述、分析结论、环境、会话、时间线、发现、Superpowers 参与程度和覆盖范围说明。
- `case.md` 为分析子代理使用的案例文件。
- `environment.json` 为环境一节的机器可读副本。
- `timeline.md` 为按轮次排列的时间线。
- `findings/<dimension>.md` 为各维度的原始分析发现。
- `transcripts/<session-id>.md` 为每个受查会话按轮次精简后的记录。严禁打包原始 JSONL。各级别对工具结果正文的处理如下。

  | 级别 | 工具结果正文 |
  |---|---|
  | skeleton | 限制正文，以 `[tool result: <tool>, <bytes> bytes, exit <code>]` 代替 |
  | evidence | 保留已引用事件的正文，包括支持发现所需的命令和结果 |
  | full | 全部保留 |
- `scrub-log.md` 列出使用过的每个占位符及其类别，严禁写入原值。

## 阅读顺序

先读 `report.md` §1–2，再读 §7（Superpowers 参与程度）及其引用的证据行，最后查看 `transcripts/` 中对应的轮次。`path:line` 引用指向报告者机器上的原始文件，精简记录中的 `[L<n>]` 标记保留了相同的行号。

## 脱敏

占位符示例包括 `<EMAIL-1>`、`<PERSON-2>`、`<SECRET-3>`、`<HOST-4>`、`<REPO-5>`、`<ORG-6>` 和 `<PROPRIETARY-7>`。主目录路径改写为 `~/…`。在同一诊断包内，同一占位符始终指向同一个原值。

## 制作说明

完成诊断包后，用实际结果替换本节。

脱敏后，仅使用诊断包内的内容核实每一条导出的实质性发现。根据引用找到诊断包所含的会话行或来源标记，阅读对应的命令、结果或引文，核实它是否支持该结论。仅检查路径和行号存在还不够。若脱敏级别或必要的隐去操作移除了证据，记录具体限制。

核对报告、案例、环境、发现、README 和本地议题草稿，使其一致。按最终文件更新脱敏日志的计数，排除日志自身。删除过时的导出陈述，并分别明确诊断包的准备状态和归档交付状态。保留历史行号与诊断包内证据的对应关系。

分别记录独立隐私审计和证据核查结果。
- 隐私审计：`CLEAN` 或尚未解决的遗漏。
- 证据支持：`supported` 或 `limited`，并注明受影响的发现和原因。

检查后如有内容变动，重新运行受影响的检查。将最终日志、文件清单和两项结果交给对方审批归档。只归档审查过的文件，并确认交付的归档与这些文件一致。在审查过的诊断包外记录归档交付情况。审批后不得改动包内容。脱敏检查无法保证涵盖所有隐私信息。
