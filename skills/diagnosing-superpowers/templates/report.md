# 会话诊断：<session-id>

报告路径：~/.superpowers/diagnosing-superpowers/<session-id>/report.md
编写时间：<ISO timestamp>

## 1. 问题陈述（必填）

<从案例文件复制。>

## 2. 分析结论（必填）

<用文字说明证据显示报告的问题前后发生了什么。每项陈述后附上 `path:line`。标注置信度（high / medium / low），并说明哪些信息能提高置信度。不要就 Superpowers 应采取的行动发表意见。>

## 3. 环境（必填）

- 操作系统
- 宿主和版本
- 记录中出现的模型
- Superpowers 安装根目录、版本和 Git SHA
- 读取或注入的技能文件（案例文件中的 SHA-1 表）
- 其他插件、扩展、MCP 服务器
- 已存在的指令文件（仅路径）

逐一标注环境字段和技能观察的证据来源类型，并记录支持证据的位置。可选类型为 `historical evidence`（历史证据）、`unverified snapshot`（未验证的快照）、`current observation`（当前观察）和 `unknown`（未知）。

## 4. 检查过的会话（必填）

| 角色 | 会话 ID | 绝对路径 | 行数 | 字节数 |
|---|---|---|---|---|

已排除的候选：<id — path — 排除原因>，无则填 `none`。

## 5. 时间线（必填）

每条人工提示一行。「事件」栏列出技能调用、子代理派发、上下文压缩、错误、会话恢复和中止。

| 轮次 | 行号 | 时间 | 请求（单行） | 事件 |
|---|---|---|---|---|

## 6. 发现（必填，每个维度一小节）

每条发现使用以下格式。

```
- finding: <one sentence>
  evidence: <path:line> — "<short quote>"
  turns: <first>–<last>
  confidence: high | medium | low
```

没有发现的维度填 `none found — checked: <what was checked>`，并注明核查过的内容。

### 6.1 技能时间线
### 6.2 计划执行情况
### 6.3 重复工作
### 6.4 执行受阻
### 6.5 质量证据
### 6.6 请求冲突
### 6.7 费用和耗时
### 6.8 使用的其他插件和技能

## 7. Superpowers 参与程度（必填）

not indicated | possible | likely

证据行：<path:line list>。本节仅说明 Superpowers 的参与程度，不认定缺陷，也不提出改动。

## 8. 覆盖范围说明（必填）

- 未读取：<范围、文件和原因>
- 不可用的宿主功能：<清单，无则填 none>
- 读取时会话是否仍在进行：yes/no
- 需要对方复核的事项：<清单，无则填 none>

## 9. 相似会话（仅在请求时填写）

| 会话 ID | 路径 | 日期 | 宿主 | 匹配 | 未匹配 |
|---|---|---|---|---|---|
