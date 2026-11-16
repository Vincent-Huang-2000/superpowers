## 派发子代理需要多代理支持

在 Codex 配置文件（`~/.codex/config.toml`）中加入以下配置。

```toml
[features]
multi_agent = true
```

这会启用 `dispatching-parallel-agents` 和 `subagent-driven-development` 等技能所用的多代理工具。可用工具取决于模型预设选择的多代理版本。当前预设使用 V2，较旧的预设使用 V1。说明与当前环境有出入时，以实际工具列表为准。

- **派发。** 用 `spawn_agent {fork_turns: "none"}` 为子代理提供干净的上下文。默认值 `"all"` 会复制整个对话记录。Codex 0.145 及以上版本可通过 `agent_type` 将 `~/.codex/agents/` 下的角色文件应用到隔离会话。继承完整记录的会话也支持覆盖 `model` 和 `reasoning_effort`，只会拒绝 `agent_type`。子代理驱动开发默认用隔离会话，以控制传入的上下文。
- **修复轮次。** 使用 `followup_task` 恢复原实施代理。它会传递消息、触发新一轮执行，并在宿主已卸载该代理时自动重新加载。在 V2 中可以继续给同一代理发消息，修复时复用原代理。
- **生命周期。** V2 会在需要名额时自动卸载已完成的子代理，无需手动关闭。只有 V1 会话提供 `close_agent`，收到审查结果时关闭审查代理，任务通过审查后关闭对应的实施代理。
- **模型名称。** 将技能、表格或旧会话中的模型名称传给 `spawn_agent` 前，先对照当前允许派发的模型列表。V2 只接受支持 V2 的预设，其他值会直接报错。

## 等待子代理

`wait_agent` 订阅事件。只要收到子代理的消息，长时间等待会和短时间等待一样立即唤醒。短间隔反复等待每次都会多一次工具调用和上下文计费。对会话的测量显示，约三分之二的等待调用是超时结束的短轮询。

- 自己还有工作时，继续执行。已完成的子代理的最终答复会送到消息队列，在你下一轮时收到。
- 确实空闲且还有子代理在工作时，分段等待。对 `wait_agent` 设置 `timeout_ms` 为 300000–600000（5–10 分钟）。每段等待结束后，无论是被唤醒还是超时，都发一行状态更新，调用 `list_agents`，并追查那些已结束却未报告的子代理。每段至少五分钟，事件订阅仍会立即唤醒。
- 完成消息只会投递到队列，无法主动触发闲置的主控会话执行一轮。`wait_agent` 用来覆盖这段空闲时间。若等待超时且没有消息，核对各子代理的状态，下一段仍按原等待间隔。

## 派发时选择模型

每次调用 `spawn_agent` 时，都按所用技能的模型选择规则显式设置 `model` 和 `reasoning_effort`。子代理发起并行派发时也一样。只设置 `model` 时，子代理的推理强度会回到该模型的默认值。

请用户在 `~/.codex/config.toml` 中设置整台机器通用的默认值。遗漏显式配置时，派发仍会落在有意选定的档位，避免继承当前会话成本最高的模型。

```toml
[agents]
default_subagent_model = "<从允许派发的模型中选择一个中档模型>"
default_subagent_reasoning_effort = "medium"
```

## 检测环境

创建 worktree 或收尾分支的技能，应先用只读 Git 命令检测环境。

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` 表示已在关联 worktree 中，跳过创建。
- `BRANCH` 为空表示 HEAD 已分离，无法从沙箱创建分支、推送或发起 PR。

参阅 `using-git-worktrees` 的步骤 0 和 `finishing-a-development-branch` 的步骤 2，了解各技能如何使用这些结果。

## 在 Codex 应用中收尾

若工作区由外部管理、HEAD 已分离，且沙箱阻止创建分支或推送，代理先提交所有工作，再请用户使用应用的原生控件。

- **「Create branch」** 为分支命名，再通过应用界面提交、推送或创建 PR。
- **「Hand off to local」** 将工作转移到用户的本地检出。

代理仍可运行测试和暂存文件，也可以给出分支名、提交消息和 PR 描述供用户复制。
