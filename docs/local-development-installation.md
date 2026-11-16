# 本地安装多个 Superpowers 开发版本

每个版本保留一份自己的本地目录。下面用 `personal` 和一个 `experiment` 版本举例，示例路径请替换成自己的实际目录。单独的 clone 或 worktree 都可以，目录里保留完整的插件入口和 `skills/`。

```powershell
$personalRoot = 'D:\dev\superpowers'
$experimentRoot = 'D:\dev\superpowers-experiment'
```

**平时只想试一版**，优先用宿主的临时加载或项目配置，直接指向那个目录。**想把两版都留在插件列表里**，按下表给它们不同的安装标识。两版装好后，同一会话只启用一版，避免同名技能和启动钩子一起生效。

## Codex 示例

[`.agents/plugins/marketplace.json`](../.agents/plugins/marketplace.json) 里 `plugins[0].source` 的 `{"source":"url","url":"./"}` 已经指向仓库自身，[`tests/codex/test-marketplace-manifest.sh`](../tests/codex/test-marketplace-manifest.sh) 会校验这个值，不用改。个人版市场名留作 `superpowers-dev`，实验版的顶层 `name` 用 `superpowers-experiment`。桌面版可同时改实验版的 `interface.displayName`，方便在界面识别。

```powershell
codex plugin marketplace add "$personalRoot"
codex plugin marketplace add "$experimentRoot"
codex plugin add superpowers@superpowers-dev
codex plugin add superpowers@superpowers-experiment
```

CLI 用 `/plugins` 切换。桌面版也可以在注册市场后重启应用，到 **Plugins** 选对应来源安装。切换后开新聊天。Codex 加载安装缓存，继续修改本地代码后，在插件界面刷新安装副本再验证。 [Codex 本地插件安装](https://developers.openai.com/plugins/build/plugins#install-a-local-plugin-manually) · [CLI 管理命令](https://learn.chatgpt.com/docs/developer-commands#codex-plugin)

## 各宿主速查

表里的 `$root` 指要安装的那份本地目录。例如先运行 `$root = $experimentRoot`，再执行相应的安装命令。聊天命令里的 `<绝对路径>` 直接填实际路径。

| 宿主 | 怎样从本地安装并避免冲突 |
| --- | --- |
| Codex App | 按上面的示例注册两个不同名称的市场，然后在 **Plugins** 从对应市场安装。 |
| Codex CLI | 按上面的命令安装成两个 `superpowers@市场名`，用 `/plugins` 切换。 |
| Claude Code | 临时使用 `claude --plugin-dir "$root"`。两版都要保留时，给实验版的 [`.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json) 一个不同的顶层 `name`，对每个目录运行 `/plugin marketplace add <绝对路径>`，再用 `/plugin install superpowers@市场名` 安装。用 `/plugin` 切换。 [Claude 插件管理](https://code.claude.com/docs/en/discover-plugins) |
| Antigravity | 给实验版的 [`.antigravity-plugin/plugin.json`](../.antigravity-plugin/plugin.json) 一个不同的 `name`，分别运行 `agy plugin install "$root"`，用 `agy plugin enable` 或 `disable` 加插件名切换。 [Antigravity 插件管理](https://antigravity.google/docs/marketplace) |
| Cursor | 给实验版的 [`.cursor-plugin/plugin.json`](../.cursor-plugin/plugin.json) 一个不同的 `name`。先确认设置里允许本地插件导入（企业版默认关闭），再把两个插件目录分别放在 `~/.cursor/plugins/local/` 下，运行 **Developer: Reload Window**。需要切换时，可以把暂时不用的目录移出 `local/` 再重载。指向 `local/` 之外的符号链接不会被加载，同名市场版也会优先于本地副本。 [Cursor 本地插件](https://cursor.com/docs/plugins#test-plugins-locally) |
| Devin CLI | 给实验版的 [`.devin-plugin/plugin.json`](../.devin-plugin/plugin.json) 一个不同的 `name`，分别运行 `devin plugins install --local "$root"`。`--local` 才把本地目录按链接方式装到本机，改动即时生效；停用一版时用 `devin plugins remove <插件名> --local` 取消注册。 [Devin 本地安装](https://docs.devin.ai/cli/extensibility/plugins/overview) |
| Factory Droid | 用两个不同名称的本地目录，分别运行 `droid plugin marketplace add "$root"`。用 `droid plugin marketplace list` 取注册名，再运行 `droid plugin install "superpowers@注册名"`。在 `/plugins` 的 **Installed** 页切换。 [Droid 市场管理](https://docs.factory.com/harness/plugins) |
| Gemini CLI | 给实验版的 [`gemini-extension.json`](../gemini-extension.json) 一个不同的 `name`，分别运行 `gemini extensions link "$root"`。用 `gemini extensions enable` 或 `disable` 加扩展名切换。 [Gemini 扩展命令](https://geminicli.com/docs/cli/cli-reference/#extensions-management) |
| GitHub Copilot CLI | 复用 Claude Code 的两个市场名称。分别运行 `copilot plugin marketplace add "$root"`，再用 `copilot plugin install superpowers@市场名` 安装。用 `copilot plugin enable` 或 `disable` 加插件名切换，两版同名时带上市场名（`superpowers@市场名`）。 [Copilot 插件管理](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-finding-installing) |
| Grok Build CLI | 启动时用 `grok --plugin-dir "$root"` 选择一版。要常驻两版时，给实验版的 [Claude 兼容插件清单](../.claude-plugin/plugin.json) 一个不同的 `name`，将两版分别放入 `~/.grok/plugins/`，在 `/plugins` 管理。 [Grok 插件加载](https://docs.x.ai/build/features/skills-plugins-marketplaces) |
| Kimi Code | 给实验版的 [`.kimi-plugin/plugin.json`](../.kimi-plugin/plugin.json) 一个不同的 `name`。分别运行 `/plugins install <绝对路径>`，在 `/plugins` 切换后开新会话。改了本地代码后重新安装该版。 [Kimi 插件管理](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/plugins.html) |
| OpenCode | 在目标项目的 `opencode.json` 里用 `"plugins": ["D:/path/to/superpowers"]` 指向选中的本地目录，V1 用单数键 `plugin`。确认全局配置里没有同时加载另一版。换路径后重启。 [OpenCode 安装说明](../.opencode/INSTALL.md) |
| Pi | 用 `pi -e "$root"` 每次选一版，或在目标项目用 `pi install --local "$root"` 固定一版。本地包按绝对路径识别，可以保留原包名。 [Pi 本地包配置](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/packages.md) |
| Qwen Code | 复用 Claude Code 的两个市场。实验版同时改 [Claude 插件清单](../.claude-plugin/plugin.json)的 `name` 和[市场清单](../.claude-plugin/marketplace.json)的 `plugins[0].name`。用 `qwen extensions sources add "$root"` 注册各目录，再到 `/extensions manage` 的 **Discover** 安装、在 **Installed** 切换。安装的是副本，改了本地代码后运行 `qwen extensions update <扩展名>`。 [Qwen 扩展管理](https://qwenlm.github.io/qwen-code-docs/en/users/extension/introduction/) |
| Hermes Agent | 用 `hermes profile create <名称>` 为每版建一个 profile，用 `hermes profile show <名称>` 查其主目录。各自在主目录的 `plugins/superpowers/` 放入该版 `.hermes-plugin/` 中的 `plugin.yaml`、`__init__.py` 和仓库的 `skills/`。运行 `hermes -p <名称> plugins enable superpowers`，用 `hermes -p <名称> chat` 选择版本。 [Hermes 本地插件](https://hermes-agent.nousresearch.com/docs/user-guide/features/plugins) · [profile 管理](https://hermes-agent.nousresearch.com/docs/reference/profile-commands/) |
| Muse | 给实验版的 [`.muse-plugin/plugin.json`](../.muse-plugin/plugin.json) 一个不同的 `name`，分别运行 `muse plugins install "$root"`。用 `muse plugins approve <插件名>` 启用该版启动钩子。用 `muse plugins enable` 或 `disable` 加插件名切换。 [Muse 插件命令](https://meta-models.github.io/muse-code-sdk/next/guides/plugins/reference/cli/) |

装完后启用要测试的版本，开新会话，调用一个在两版内容不同的技能来确认加载来源。复制式安装要刷新插件副本才会读到后续改动，链接式安装则跟随各自的本地目录。
