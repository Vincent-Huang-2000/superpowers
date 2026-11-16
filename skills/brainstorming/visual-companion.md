# 可视化辅助指南

在浏览器中展示界面样稿、图示和选项，辅助构思。

## 何时使用

逐个问题判断。**给用户直接看，是否比请用户读文字更容易理解？**

**用浏览器**展示视觉内容。

- **UI 样稿**，如线框图、布局、导航结构和组件设计
- **架构图**，如系统组件、数据流和关系图
- **并排对比**，如两套布局、配色或设计方向
- **设计细节**，如观感、间距和视觉层级
- **空间关系**，如以图示表现状态机、流程和实体关系

**用终端**讨论文本或表格内容。

- **需求与范围**，如「X 是什么意思」「哪些功能在范围内」
- **概念上的 A/B/C 选择**，用文字描述各方案
- **取舍**，如优缺点和对比表
- **技术决策**，如 API 设计、数据建模和架构方案
- **澄清问题**，需要的是文字说明，或答案与视觉偏好无关

即使讨论 UI，也要根据问题选择媒介。「你想要什么样的向导？」适合在终端讨论。「这些向导布局里哪个更合适？」适合在浏览器里展示。

## 工作原理

服务器监视一个目录，在浏览器中提供其中最新的 HTML 文件。把 HTML 写入 `screen_dir`，用户就能在浏览器里查看并点击选项。用户的操作会记录在 `state_dir/events` 中，在下一轮读取。

如果 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器直接提供该页面，只注入辅助脚本。其他文件会自动套用框架模板，添加页眉、CSS 主题、连接状态和交互功能。**默认编写内容片段。** 只有需要完整控制页面时，才编写完整文档。

## 启动会话

```bash
# 用户同意使用可视化辅助后再启动。--open 在首屏就绪时自动打开浏览器
# --project-dir 保存样稿，并在重启时复用端口
bash scripts/start-server.sh --project-dir /path/to/project --open

# 返回：{"type":"server-started","port":52341,
#        "url":"http://localhost:52341/?key=ab12…",
#        "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#        "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

保存响应中的 `screen_dir` 和 `state_dir`。使用 `--open` 时，写入首个页面后浏览器会自动打开。仍需把链接发给用户，以便无界面或远程环境中的用户手动打开。

**链接含有会话密钥（`?key=…`）。** 服务器会拒绝缺少密钥的请求。给用户完整的 `url` 字段值，保留查询字符串。仅包含主机和端口的 `http://host:port` 无法访问会话。密钥控制 HTTP 和 WebSocket 访问，避免其他标签页或网络上的其他设备读取页面或提交事件。首次加载后，浏览器通过 Cookie 记住密钥，重新加载和访问 `/files/*` 资源时无需重复提供。

**获取连接信息。** 服务器把启动 JSON 写入 `$STATE_DIR/server-info`。若后台启动时未获得标准输出，从该文件读取链接和端口。使用 `--project-dir` 时，可在 `<project>/.superpowers/brainstorm/` 中找到会话目录。

**传入项目根目录作为 `--project-dir`。** 样稿会保存在 `.superpowers/brainstorm/`，服务器重启后仍然可用。省略参数时，文件写入 `/tmp`，随后会被清理。如果 `.gitignore` 尚未包含 `.superpowers/`，提醒用户添加。

**按宿主启动服务器。**

**Claude Code**

```bash
# 默认模式可用，脚本会自行将服务器放到后台。
bash scripts/start-server.sh --project-dir /path/to/project --open
```

在 Windows 上，脚本会自动切换为前台模式，因而阻塞工具调用。调用 Bash 工具时设置 `run_in_background: true`，让服务器跨对话轮次保持运行。下一轮从 `$STATE_DIR/server-info` 获取链接和端口。

**Codex**

```bash
# Codex 会回收后台进程。脚本检测到 CODEX_CI 时会自动切换为前台模式。
# 直接运行即可，无需额外参数。
bash scripts/start-server.sh --project-dir /path/to/project --open
```

**Gemini CLI**

```bash
# 使用 --foreground，并在 Shell 工具调用中设置 is_background: true，
# 使进程跨轮次保持运行。
bash scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**Copilot CLI**

```bash
# 用 Copilot CLI 的非阻塞或后台 Shell 机制启动，确保跨轮次运行。
# 保留 --foreground，由宿主负责后台运行。用 Bash 执行 .sh 文件，
# Windows 上从 PowerShell 工具调用 Git Bash 的 bash.exe。
bash scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**其他环境。** 服务器必须在对话轮次之间保持运行。如果环境会回收分离进程，使用 `--foreground`，并通过平台提供的后台执行机制启动。

如果浏览器无法访问链接（远程环境和容器中很常见），绑定非回环地址。

```bash
bash scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

`--url-host` 控制返回的 JSON 链接所显示的主机名。

## 交互流程

1. **确认服务器还在运行**，然后在 `screen_dir` 中新建 HTML 文件。
   - **给出链接或推送页面前，必须确认服务器可用。** 检查 `$STATE_DIR/server-info` 存在且 `$STATE_DIR/server-stopped` 不存在。若已停止，使用相同的 `--project-dir` 重新运行 `start-server.sh`。服务器会复用原端口，用户打开的标签页会自动重连，停机时会显示暂停遮罩，无需重新发链接。服务器空闲 4 小时后自动退出，可通过 `--idle-timeout-minutes` 调整。
   - 用有语义的文件名，例如 `platform.html`、`visual-style.html`、`layout.html`。
   - 每个页面写入新文件，禁止复用文件名。
   - 使用文件创建工具。禁止用 `cat` 或 heredoc 写入页面，以免把内容刷到终端。
   - 服务器自动展示最新文件。

2. **告知用户页面内容并结束当前轮次。**
   - 每一步都附上链接。
   - 简述页面内容，例如「页面展示了 3 种首页布局」。
   - 请用户在终端回复，例如「请看看这几种方案，告诉我你的想法。也可以直接点击选项。」

3. **用户在终端回复后的下一轮。**
   - 如果 `$STATE_DIR/events` 存在，读取其中的点击和选择记录，每行是一条 JSON。
   - 结合终端中的文字反馈理解用户的意见。
   - 以终端消息为主，`state_dir/events` 提供结构化的交互数据。

4. **迭代当前页面或继续下一题。** 如果反馈导致当前页面需要调整，新建文件，例如 `layout-v2.html`。等用户确认当前步骤后，再进入下一个问题。

5. **回到终端时清空旧页面。** 如果下一步不需要浏览器，例如要提澄清问题或讨论取舍，推送等待页面。

   ```html
   <!-- 文件名：waiting.html（或 waiting-2.html 等） -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">请回到终端继续……</p>
   </div>
   ```

   这样用户就不会在话题推进后仍看到已讨论完的选项。遇到下一个视觉问题时，照常推送新的内容文件。

6. 重复以上流程，直到讨论完成。

## 编写内容片段

只写页面内的内容。服务器会自动套用框架模板，提供页眉、CSS 主题、连接状态和交互功能。

**最小示例**

```html
<h2>哪种布局更合适？</h2>
<p class="subtitle">请考虑可读性和视觉层级</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>单栏布局</h3>
      <p>阅读体验简洁，便于集中注意力</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>双栏布局</h3>
      <p>侧栏负责导航，主栏展示内容</p>
    </div>
  </div>
</div>
```

服务器会提供 `<html>`、CSS 和脚本。

## 可用的 CSS 类

框架模板提供以下 CSS 类。

### 选项（A/B/C）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>标题</h3>
      <p>说明</p>
    </div>
  </div>
</div>
```

**多选。** 给容器添加 `data-multiselect`。每次点击会切换该项的选中样式。

```html
<div class="options" data-multiselect>
  <!-- 选项标记相同，用户可以同时选择或取消多个选项 -->
</div>
```

### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- 样稿内容 --></div>
    <div class="card-body">
      <h3>名称</h3>
      <p>说明</p>
    </div>
  </div>
</div>
```

### 样稿容器

```html
<div class="mockup">
  <div class="mockup-header">预览：仪表板布局</div>
  <div class="mockup-body"><!-- 放入样稿的 HTML --></div>
</div>
```

### 并排视图

```html
<div class="split">
  <div class="mockup"><!-- 左边 --></div>
  <div class="mockup"><!-- 右边 --></div>
</div>
```

### 优缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>优点</h4><ul><li>收益</li></ul></div>
  <div class="cons"><h4>缺点</h4><ul><li>代价</li></ul></div>
</div>
```

### 线框图组件

```html
<div class="mock-nav">标志 | 首页 | 关于 | 联系方式</div>
<div style="display: flex;">
  <div class="mock-sidebar">导航</div>
  <div class="mock-content">主内容区</div>
</div>
<button class="mock-button">操作按钮</button>
<input class="mock-input" placeholder="输入内容">
<div class="placeholder">占位区域</div>
```

### 排版与区块

- `h2` 用于页面标题
- `h3` 用于区块标题
- `.subtitle` 用于标题下的辅助说明
- `.section` 为内容块提供下边距
- `.label` 用于较小的标签文字，拉丁字母显示为大写

## 浏览器事件格式

用户在浏览器里点击选项时，服务器会把交互记录到 `$STATE_DIR/events`，每行一个 JSON 对象。推送新页面后，文件会自动清空。

```jsonl
{"type":"click","choice":"a","text":"方案 A - 简洁布局","timestamp":1706000101}
{"type":"click","choice":"c","text":"方案 C - 复杂网格","timestamp":1706000108}
{"type":"click","choice":"b","text":"方案 B - 混合布局","timestamp":1706000115}
```

完整的事件流记录了用户查看选项的过程。用户定下来之前可能会点击多个选项。最后一条包含 `choice` 的事件通常代表最终选择，而点击顺序可能透露值得询问的顾虑或偏好。

如果 `$STATE_DIR/events` 不存在，说明用户没有在浏览器中操作，使用终端里的回复即可。

## 设计建议

- 让样稿精细程度匹配问题。讨论布局时用线框图，讨论视觉细节时再细化样稿。
- 每个页面都要提出具体问题，例如「哪种布局看起来更专业？」
- 如果当前页面需要调整，先写入新版本再继续。
- 每个页面只放 2 到 4 个选项。
- 内容会影响判断时，使用真实素材。例如摄影作品集可使用真实照片（Unsplash），占位内容可能掩盖设计问题。
- 保持样稿简洁，优先展示布局和结构。

## 文件命名

- 用有语义的名称，例如 `platform.html`、`visual-style.html`、`layout.html`。
- 每个页面都写入新文件。
- 迭代时添加版本后缀，例如 `layout-v2.html`、`layout-v3.html`。
- 服务器按修改时间展示最新文件。

## 清理

```bash
bash scripts/stop-server.sh $SESSION_DIR
```

使用 `--project-dir` 时，样稿保留在 `.superpowers/brainstorm/`，以便日后查阅。停止服务器时只会删除 `/tmp` 下的会话。

## 参考

- 框架模板（CSS 参考）`scripts/frame-template.html`
- 客户端辅助脚本 `scripts/helper.js`
