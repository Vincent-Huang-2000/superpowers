# 技能编写最佳实践

> 学习如何编写代理能够发现并成功使用的技能。

好的技能简洁、结构清晰，并经过实际使用检验。本指南说明编写时如何取舍，帮助你写出代理能发现并有效使用的技能。

有关技能如何运作的基本概念，见[技能概览](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)。

## 核心原则

### 保持简洁

[上下文窗口](https://platform.claude.com/docs/en/build-with-claude/context-windows)是共享资源。技能需要与代理要知道的所有其他信息共用这个窗口，包括：

* 系统提示词
* 对话历史
* 其他技能的元数据
* 用户实际发出的请求

技能占用的 token 并非都立即计入上下文。启动时只预加载所有技能的元数据（名称和描述）。当技能适用于任务时，代理才读取 SKILL.md，并按需读取附加文件。SKILL.md 仍应保持简洁，因为代理加载它之后，每个 token 都会与对话历史和其他上下文争夺空间。

**默认假设**：代理已经具备很强的能力。

只补充代理尚不了解的背景。逐项问自己：

* 「代理真的需要这段说明吗？」
* 「可以假设代理已经知道这一点吗？」
* 「这段话值得占用这么多 token 吗？」

**好的示例：简洁**（约 50 个 token，按英文原例计）：

````markdown  theme={null}
## 提取 PDF 文本

使用 pdfplumber 提取文本：

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**不佳的示例：太冗长**（约 150 个 token，按英文原例计）：

```markdown  theme={null}
## 提取 PDF 文本

PDF（便携式文档格式）是一种常见文件格式，可包含文本、图像和其他内容。
你需要用一个库从 PDF 中提取文本。有许多库可用于处理 PDF，这里推荐
pdfplumber，因为它很好用，也能处理大多数情况。首先，需要用 pip 安装它。
接下来可以使用以下代码……
```

简洁的版本假设代理知道 PDF 是什么，也知道库的基本用法。

### 设定适当的自由度

操作越容易出错、做法越固定，指令就应越具体。

**高自由度**（文字指引）

适用于：

* 有多种可行做法。
* 决策取决于上下文。
* 可以依靠经验规则选择做法。

示例：

```markdown  theme={null}
## 代码审查流程

1. 分析代码的结构和组织方式。
2. 检查潜在缺陷及边缘情况。
3. 提出改善可读性和可维护性的建议。
4. 验证代码是否符合项目约定。
```

**中等自由度**（伪代码或带参数的脚本）

适用于：

* 有优先采用的模式。
* 允许一定程度的变化。
* 配置会影响行为。

示例：

````markdown  theme={null}
## 生成报告

使用以下模板，并按需调整：

```python
def generate_report(data, format="markdown", include_charts=True):
    # 处理数据
    # 生成指定格式的输出
    # 按需加入可视化内容
```
````

**低自由度**（明确指定脚本，且参数很少或没有参数）

适用于：

* 操作容易出错。
* 一致性非常重要。
* 必须遵循特定顺序。

示例：

````markdown  theme={null}
## 数据库迁移

严格按以下命令运行脚本：

```bash
python scripts/migrate.py --verify --backup
```

不得修改命令或添加参数。
````

可以把代理想象成在探索路线的机器人：

* **两侧都是悬崖的窄桥**只有一条安全路线，需要明确的约束和精确指令（低自由度）。数据库迁移就是一例，必须按规定顺序运行。
* **没有危险的开阔地**有多条可行路线。给出大致方向，让代理自己找到最佳做法（高自由度）。代码审查就是一例，最佳做法取决于上下文。

### 使用计划支持的所有模型进行测试

技能是对模型的补充，其效果取决于底层模型。用计划支持的每个模型测试技能。

各模型的测试重点：

* **Claude Haiku**（快速、经济）：技能是否提供了足够的指引？
* **Claude Sonnet**（均衡）：技能是否清楚、易于使用？
* **Claude Opus**（推理能力强）：技能是否避免了过度解释？

对 Opus 恰到好处的指引，Haiku 可能还需要更多细节。如果计划在多个模型上使用技能，尽量让指令对所有目标模型都有效。

## 技能结构

<Note>
  **YAML 前置元数据**：SKILL.md 的前置元数据要求两个字段：

  * `name` - 便于人阅读的技能名称（最多 64 字符）
  * `description` - 用一行说明技能的用途和适用条件（最多 1024 字符）

  完整的技能结构见[技能概览](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### 命名约定

保持命名方式一致，方便在文档和讨论中引用技能。英文名推荐用**动名词形式**（动词加 `-ing`），以直接说明该技能提供的能力。

推荐的动名词形式：

* 「Processing PDFs」（处理 PDF）
* 「Analyzing spreadsheets」（分析电子表格）
* 「Managing databases」（管理数据库）
* 「Testing code」（测试代码）
* 「Writing documentation」（编写文档）

也可以采用：

* 名词短语，如「PDF Processing」「Spreadsheet Analysis」。
* 动作导向的短语，如「Process PDFs」「Analyze Spreadsheets」。

应避免：

* 含糊的名称，如「Helper」「Utils」「Tools」。
* 过于宽泛的名称，如「Documents」「Data」「Files」。
* 同一个技能库中混用不一致的命名方式。

统一命名有助于在文档和对话中引用技能，快速理解技能用途，组织和搜索多个技能，以及维护风格一致的技能库。

### 编写有效的描述

`description` 字段用于发现技能。按 Anthropic 的指南，其中应同时说明技能的用途和适用条件。

<Warning>
  **始终使用第三人称。** 描述会注入系统提示词，视角不统一会影响技能发现。

  * **正确：**「处理 Excel 文件并生成报告」
  * **避免：**「我可以帮你处理 Excel 文件」
  * **避免：**「你可以用这个技能处理 Excel 文件」
</Warning>

**写具体内容，并包含关键词。** 写明技能能做什么，以及哪些触发条件或情境适合使用。

每个技能只有一个描述字段。代理可能要从 100 多项技能中作出选择，因此描述对选中正确的技能至关重要。描述应提供足够的信息，让代理知道何时选择该技能，实施细节则写在 SKILL.md 的其余部分。

有效的示例：

**PDF 处理技能**

```yaml  theme={null}
description: 适用于处理 PDF 文件，或用户提到 PDF、表单或文档提取的情况。可从 PDF 中提取文本和表格、填写表单及合并文档。
```

**Excel 分析技能**

```yaml  theme={null}
description: 适用于分析 Excel 文件、电子表格、表格数据或 .xlsx 文件。可分析 Excel 电子表格、创建数据透视表及生成图表。
```

**Git 提交辅助技能**

```yaml  theme={null}
description: 适用于用户请求协助编写提交信息或审查已暂存的改动。可分析 Git 差异并生成能说明改动内容的提交信息。
```

避免以下含糊的描述：

```yaml  theme={null}
description: 帮助处理文档
```

```yaml  theme={null}
description: 处理数据
```

```yaml  theme={null}
description: 对文件做一些操作
```

### 渐进式披露的组织模式

SKILL.md 提供概览，并像入门指南的目录一样，按需将代理引向详细资料。有关渐进式披露的原理，见概览中的[技能如何运作](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

实际建议：

* 为获得最佳效果，把 SKILL.md 正文控制在 500 行以内。
* 接近上限时，将内容拆到单独的文件。
* 用以下模式有效组织指令、代码和资源。

#### 图示：从简单到复杂

最基础的技能可以只有一个包含元数据和指令的 SKILL.md 文件：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="简单的 SKILL.md 文件，含 YAML 前置元数据和 Markdown 正文" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

技能扩充后，可以附带其他内容，让代理按需加载：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="随技能附带 reference.md 和 forms.md 等参考文件" data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

完整的技能目录结构可能如下：

```
pdf/
├── SKILL.md              # 主要指令（触发时加载）
├── FORMS.md              # 表单填写指南（按需加载）
├── reference.md          # API 参考（按需加载）
├── examples.md           # 用法示例（按需加载）
└── scripts/
    ├── analyze_form.py   # 工具脚本（执行时无需载入上下文）
    ├── fill_form.py      # 表单填写脚本
    └── validate.py       # 验证脚本
```

#### 模式 1：提供概览，链接参考资料

````markdown  theme={null}
---
name: PDF Processing
description: 适用于处理 PDF 文件，或用户提到 PDF、表单或文档提取的情况。可提取文本和表格、填写表单及合并文档。
---

# PDF 处理

## 快速入门

用 pdfplumber 提取文本：
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## 高级功能

**填写表单**的完整指南见 [FORMS.md](FORMS.md)。
**API 参考**的完整方法列表见 [REFERENCE.md](REFERENCE.md)。
**示例**中的常见模式见 [EXAMPLES.md](EXAMPLES.md)。
````

代理只在需要时加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

#### 模式 2：按领域组织

如果技能涉及多个领域，就按领域组织内容，避免载入无关上下文。用户询问销售指标时，代理只需读取销售相关的数据结构。将财务、营销资料拆开，可以减少 token 用量，让上下文围绕当前任务。

```
bigquery-skill/
├── SKILL.md（概览与导航）
└── reference/
    ├── finance.md（营收、账单指标）
    ├── sales.md（商机、销售漏斗）
    ├── product.md（API 用量、功能）
    └── marketing.md（营销活动、归因）
```

````markdown SKILL.md theme={null}
# BigQuery 数据分析

## 可用数据集

**财务**涉及营收、ARR 和账单。见 [reference/finance.md](reference/finance.md)。
**销售**涉及商机、销售漏斗和客户。见 [reference/sales.md](reference/sales.md)。
**产品**涉及 API 用量、功能和采用情况。见 [reference/product.md](reference/product.md)。
**营销**涉及营销活动、归因和电子邮件。见 [reference/marketing.md](reference/marketing.md)。

## 快速搜索

用 grep 查找特定指标：

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### 模式 3：按条件提供细节

直接展示基础内容，用链接提供进阶内容：

```markdown  theme={null}
# DOCX 处理

## 创建文档

用 docx-js 创建新文档。见 [DOCX-JS.md](DOCX-JS.md)。

## 编辑文档

简单编辑可直接修改 XML。

**修订跟踪**见 [REDLINING.md](REDLINING.md)。
**OOXML 的细节**见 [OOXML.md](OOXML.md)。
```

代理只在用户需要对应功能时读取 REDLINING.md 或 OOXML.md。

### 避免多层嵌套引用

若引用文件再去引用其他文件，代理可能只读一部分内容。遇到嵌套引用时，代理可能用 `head -100` 等命令预览，而不读取完整文件，导致漏掉实际需要的信息。

**参考资料与 SKILL.md 之间只保留一层。** 所有参考文件都应从 SKILL.md 直接链接，确保代理按需完整读取。

**不佳的示例：嵌套层级太深**

```markdown  theme={null}
# SKILL.md
见 [advanced.md](advanced.md)……

# advanced.md
见 [details.md](details.md)……

# details.md
具体信息写在这里……
```

**好的示例：只保留一层**

```markdown  theme={null}
# SKILL.md

**基本用法**：[instructions in SKILL.md]
**高级功能**：见 [advanced.md](advanced.md)
**API 参考**：见 [reference.md](reference.md)
**示例**：见 [examples.md](examples.md)
```

### 为较长的参考文件添加目录

参考文件超过 100 行时，在顶部加上目录。这样，代理即使只读开头，也能知道有哪些信息可用。

示例：

```markdown  theme={null}
# API 参考

## 目录
- 身份验证与配置
- 核心方法（创建、读取、更新、删除）
- 高级功能（批量操作、Webhook）
- 错误处理模式
- 代码示例

## 身份验证与配置
...

## 核心方法
...
```

代理可以按需阅读全文，或跳到指定章节。

有关这种基于文件系统的架构如何实现渐进式披露，见下文高级部分的[运行环境](#runtime-environment)。

## 工作流程和反馈循环

### 为复杂任务设计工作流程

把复杂操作拆成清晰、有序的步骤。流程特别复杂时，提供一份代理能复制到回复中、随进度逐项勾选的检查清单。

**示例 1：汇总研究资料的工作流程**（适用于不含代码的技能）

````markdown  theme={null}
## 汇总研究资料的工作流程

复制此清单并跟踪进展：

```
研究进度：
- [ ] 第 1 步：阅读所有来源文档
- [ ] 第 2 步：识别主要主题
- [ ] 第 3 步：交叉核对主张
- [ ] 第 4 步：撰写结构化摘要
- [ ] 第 5 步：验证引文
```

**第 1 步：阅读所有来源文档**

阅读 `sources/` 目录中的每份文档，记录主要论点和支持证据。

**第 2 步：识别主要主题**

找出各来源之间的共同点。哪些主题反复出现？哪些方面意见一致，哪些方面有分歧？

**第 3 步：交叉核对主张**

逐一确认每项主要主张都有资料依据，记录各论点分别得到了哪个来源的支持。

**第 4 步：撰写结构化摘要**

按主题组织发现，包含：
- 主要主张
- 来源中的支持证据
- 有分歧的观点（如有）

**第 5 步：验证引文**

确认每项主张都引用了正确的来源。如果有引文遗漏，回到第 3 步。
````

示例说明了如何用工作流程开展无需编写代码的分析任务。检查清单也适用于其他复杂的多步骤流程。

**示例 2：填写 PDF 表单的工作流程**（适用于含代码的技能）

````markdown  theme={null}
## 填写 PDF 表单的工作流程

复制此清单，每完成一项就打勾：

```
任务进度：
- [ ] 第 1 步：分析表单（运行 analyze_form.py）
- [ ] 第 2 步：创建字段映射（编辑 fields.json）
- [ ] 第 3 步：验证映射（运行 validate_fields.py）
- [ ] 第 4 步：填写表单（运行 fill_form.py）
- [ ] 第 5 步：验证输出（运行 verify_output.py）
```

**第 1 步：分析表单**

运行 `python scripts/analyze_form.py input.pdf`。

此命令提取表单字段及其位置，并将结果保存到 `fields.json`。

**第 2 步：创建字段映射**

编辑 `fields.json`，为每个字段添加值。

**第 3 步：验证映射**

运行 `python scripts/validate_fields.py fields.json`。

修复所有验证错误后才能继续。

**第 4 步：填写表单**

运行 `python scripts/fill_form.py input.pdf fields.json output.pdf`。

**第 5 步：验证输出**

运行 `python scripts/verify_output.py output.pdf`。

如果验证失败，回到第 2 步。
````

明确的步骤可防止代理跳过关键验证。检查清单能让你和代理都知道多步骤流程进行到了哪里。

### 建立反馈循环

**常见做法**是运行验证器、修复错误，再重复这一循环。这能显著提高输出质量。

**示例 1：遵循风格指南**（适用于不含代码的技能）

```markdown  theme={null}
## 内容审查流程

1. 按照 STYLE_GUIDE.md 的指引起草内容。
2. 对照清单审查：
   - 检查术语是否一致。
   - 验证示例是否符合标准格式。
   - 确认所有必需章节都已包含。
3. 如果发现问题：
   - 记录每个问题及其所在的具体章节。
   - 修改内容。
   - 再次按照清单审查。
4. 满足所有要求后才能继续。
5. 定稿并保存文档。
```

此例的验证循环使用参考文档。代理通过阅读并与 STYLE\_GUIDE.md 对照，完成验证。

**示例 2：编辑文档**（适用于含代码的技能）

```markdown  theme={null}
## 编辑文档的流程

1. 编辑 `word/document.xml`。
2. **立即验证**：`python ooxml/scripts/validate.py unpacked_dir/`
3. 如果验证失败：
   - 仔细阅读错误消息。
   - 修复 XML 中的问题。
   - 再次运行验证。
4. **只有通过验证才能继续。**
5. 重新打包：`python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. 测试输出文档。
```

验证循环能及早发现错误。

## 内容编写指南

### 避免有时效性的信息

避免加入以后会过时的信息。

**不佳的示例：有时效性**（以后会出错）

```markdown  theme={null}
如果你在 2025 年 8 月前执行此操作，请使用旧版 API。
2025 年 8 月之后，请使用新版 API。
```

**好的示例**（设置「旧用法」一节）

```markdown  theme={null}
## 当前用法

使用 v2 API 端点 `api.example.com/v2/messages`。

## 旧用法

<details>
<summary>旧版 v1 API（2025-08 弃用）</summary>

v1 API 使用 `api.example.com/v1/messages`。

此端点已停止支持。
</details>
```

「旧用法」一节可以交代历史背景，又让主要内容保持简洁。

### 统一术语

选定一个术语后，在整个技能中始终使用它。

**正确，保持一致：**

* 一律使用「API 端点」。
* 一律使用「字段」。
* 一律使用「提取」。

**不佳，混用术语：**

* 混用「API 端点」「URL」「API 路由」「路径」。
* 混用「字段」「输入框」「元素」「控件」。
* 混用「提取」「拉取」「获取」「检索」。

统一术语有助于代理理解并遵循指令。

## 常见模式

### 模板模式

提供输出格式模板，按需要决定对格式要求有多严格。

**对格式要求严格**（如 API 响应或数据格式）

````markdown  theme={null}
## 报告结构

始终严格使用以下模板结构：

```markdown
# [Analysis Title]

## 执行摘要
[One-paragraph overview of key findings]

## 主要发现
- 发现 1 及支持数据
- 发现 2 及支持数据
- 发现 3 及支持数据

## 建议
1. 具体可执行的建议
2. 具体可执行的建议
```
````

**提供灵活的指引**（适合按情况调整）

````markdown  theme={null}
## 报告结构

下面是合理的默认格式，可根据分析结果自行调整：

```markdown
# [Analysis Title]

## 执行摘要
[Overview]

## 主要发现
[Adapt sections based on what you discover]

## 建议
[Tailor to the specific context]
```

根据分析类型按需调整各节。
````

### 示例模式

如果技能的输出质量取决于代理是否看到示例，就像编写普通提示词一样，提供成对的输入与输出：

````markdown  theme={null}
## 提交信息格式

按以下示例生成提交信息：

**示例 1**
输入：添加了基于 JWT 的用户身份验证
输出：
```
feat(auth): 实现基于 JWT 的身份验证

添加登录端点和令牌验证中间件
```

**示例 2**
输入：修复了报告中日期显示错误的问题
输出：
```
fix(reports): 修正时区转换中的日期格式

统一在生成报告时使用 UTC 时间戳
```

**示例 3**
输入：更新了依赖并重构了错误处理
输出：
```
chore: 更新依赖并重构错误处理

- 将 lodash 升级到 4.17.21
- 统一各端点的错误响应格式
```

遵循 type(scope): brief description 的格式，再写详细说明。
````

示例比单靠描述更容易让代理掌握所需的风格和详细程度。

### 条件式工作流程

在决策点指明代理应走哪条路径：

```markdown  theme={null}
## 文档修改工作流程

1. 判断修改类型：

   **创建新内容？** → 使用下文的「创建流程」
   **编辑现有内容？** → 使用下文的「编辑流程」

2. 创建流程：
   - 使用 docx-js 库。
   - 从头创建文档。
   - 导出为 .docx 格式。

3. 编辑流程：
   - 解包现有文档。
   - 直接修改 XML。
   - 每次修改后验证。
   - 完成后重新打包。
```

<Tip>
  如果流程变长、步骤复杂，可以拆到单独的文件中，并告诉代理如何根据当前任务选择文件。
</Tip>

## 评估和迭代

### 先编写评估

**在编写大量文档之前先创建评估。** 这样能让技能解决实际存在的问题。

按以下步骤开展评估驱动开发：

1. **发现缺口。** 让代理在未加载技能时执行有代表性的任务，记录具体失败或缺失的上下文。
2. **创建评估。** 编写三个场景来测试这些缺口。
3. **建立基线。** 衡量代理在没有技能时的表现。
4. **编写最精简的指令。** 只写解决这些缺口并通过评估所需的内容。
5. **迭代。** 运行评估，与基线比较，再完善指令。

这样可以围绕已发生的问题开展工作，避免提前猜测可能永远不会出现的需求。

评估结构示例：

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "提取此 PDF 文件中的全部文本，并保存到 output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "使用合适的 PDF 处理库或命令行工具成功读取 PDF 文件",
    "提取文档所有页面的文本内容，没有遗漏页面",
    "将提取的文本以清晰易读的格式保存到名为 output.txt 的文件"
  ]
}
```

<Note>
  本例展示了如何用简单的评分规则做数据驱动的评估。Anthropic 目前没有提供运行此类评估的内置功能，用户可以创建自己的评估系统。衡量技能效果时，以实际评估为依据。
</Note>

### 与代理共同迭代技能

在技能开发中让代理参与进来，效果最好。用一个实例（「代理 A」）创建技能，再交给其他实例（「代理 B」）使用。代理 A 帮你设计和完善指令，代理 B 则在实际任务中测试。底层模型既知道如何编写有效的代理指令，也了解代理工作时需要哪些信息。

**创建新技能**

1. **先完成一项没有技能的任务。** 用普通提示词与代理 A 一起解决问题。过程中，你会自然地补充背景、说明偏好并分享操作经验。留意哪些信息反复出现。

2. **找出可复用的模式。** 完成任务后，想想哪些背景对将来类似的任务也有帮助。

   **例如**，进行 BigQuery 分析时，你可能提供过表名、字段定义、过滤规则（如「始终排除测试账号」）和常用查询模式。

3. **请代理 A 编写技能。**「把我们刚才的 BigQuery 分析模式编成一项技能，包含表结构、命名约定和过滤测试账号的规则。」

   <Tip>
     现代代理原生理解技能的格式和结构。直接请代理编写技能即可，它能生成结构正确的 SKILL.md，包含恰当的前置元数据和正文。无需特殊的系统提示词或专门用来编写技能的技能。
   </Tip>

4. **检查是否简洁。** 确认代理 A 没有加入多余解释。可以说「删掉关于赢单率含义的解释，代理已经知道这个概念。」

5. **改进内容组织。** 请代理 A 用更有效的方式组织内容。例如「把表结构放进独立的参考文件。我们以后可能还会加表。」

6. **用相似任务测试。** 让全新的代理 B 实例加载技能，执行相关任务。观察它能否找到所需信息、正确应用规则并成功完成任务。

7. **根据观察改进。** 如果代理 B 遇到困难或漏掉了某些事项，带着具体情况回到代理 A。例如「代理使用这项技能时，忘了按第四季度的日期范围过滤。是否需要加一节讲日期过滤的常用模式？」

**迭代现有技能**

改进技能时，仍沿用这套分工：

* 与负责完善技能的代理 A 合作。
* 让代理 B 使用技能完成实际工作。
* 观察代理 B 的行为，把发现反馈给代理 A。

1. **在真实流程中使用技能。** 把实际任务交给已加载技能的代理 B。

2. **观察代理 B 的行为。** 记录它在哪些地方遇到困难、顺利完成工作或作出了出乎意料的选择。

   **例如**，「我让代理 B 编写区域销售报告时，它写出了查询，却忘记排除测试账号。技能里已经写了这条规则。」

3. **请代理 A 改进。** 把当前的 SKILL.md 和观察到的情况交给代理 A。例如「我让代理 B 编写区域报告时，它忘了过滤测试账号。技能提到了过滤要求，可能需要写得更醒目？」

4. **审阅代理 A 的建议。** 它可能建议重新组织内容，让规则更醒目，改用强度更高的措辞，例如把「始终过滤」改为「必须过滤」，或者调整工作流程一节。

5. **应用并测试改动。** 按代理 A 的建议改进技能，再让代理 B 用类似请求测试。

6. **随使用情况继续迭代。** 遇到新场景时，继续观察、完善和测试。每轮迭代都以代理的实际行为为依据。

**收集团队反馈**

1. 与同事共享技能，观察他们怎么用。
2. 询问技能是否在预期时机触发、指令是否清楚，以及缺少哪些内容。
3. 吸收反馈，弥补自己使用方式中的盲点。

代理 A 了解代理需要哪些信息，你了解相关领域，代理 B 通过实际使用暴露缺口。每轮改进都以观察到的行为为依据。

### 观察代理如何查阅技能

迭代时留意代理在实际工作中如何使用技能，重点观察：

* **出乎意料的查阅路径**：代理是否按意想不到的顺序读取文件？如果是，文件组织可能不够直观。
* **遗漏的引用**：代理是否错过了指向重要文件的引用？链接可能需要更明确、更醒目。
* **反复读取的章节**：如果代理总是读取同一份文件，可以考虑把这些内容放到主要的 SKILL.md 中。
* **从不读取的内容**：如果代理从不访问附带的某个文件，该文件可能没必要保留，或者主要指令没有明确提示何时读取。

按观察结果迭代。技能元数据中的 `name` 和 `description` 尤其重要。代理根据这两个字段判断当前任务是否应触发技能，因此要清楚说明技能的用途和适用条件。

## 应避免的反模式

### 避免 Windows 风格的路径

文件路径始终用正斜杠，即使在 Windows 上也一样：

* ✓ **正确**：`scripts/helper.py`、`reference/guide.md`
* ✗ **避免**：`scripts\helper.py`、`reference\guide.md`

Unix 风格的路径可以跨平台使用，Windows 风格的路径会在 Unix 系统上出错。

### 避免提供太多选项

仅在确有必要时提供多种做法：

````markdown  theme={null}
**不佳的示例：选项太多，容易困惑**
「可以用 pypdf、pdfplumber、PyMuPDF、pdf2image，或者……」

**好的示例：提供默认方案和备用方案**
「用 pdfplumber 提取文本：
```python
import pdfplumber
```

如果遇到需要 OCR 的扫描版 PDF，就使用 pdf2image 和 pytesseract。」
````

## 进阶：含可执行代码的技能

以下各节介绍含可执行脚本的技能。如果技能只包含 Markdown 指令，可以直接跳到[有效技能的检查清单](#checklist-for-effective-skills)。

### 在脚本中处理错误

为技能编写脚本时，应在脚本中处理错误情况，避免把问题留给代理解决。

**好的示例：显式处理错误**

```python  theme={null}
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # 文件不存在时，用默认内容创建文件
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # 提供可用的备用结果
        print(f"Cannot access {path}, using default")
        return ''
```

**不佳的示例：把错误留给代理处理**

```python  theme={null}
def process_file(path):
    # 直接报错，让代理自行处理
    return open(path).read()
```

配置参数也应说明取值依据，避免难以解释的常量（Ousterhout 定律）。如果你都不知道该取什么值，代理又如何判断？

**好的示例：直接说明取值依据**

```python  theme={null}
# HTTP 请求通常在 30 秒内完成
# 较长的超时能容纳速度慢的连接
REQUEST_TIMEOUT = 30

# 重试三次，在可靠性和速度之间取得平衡
# 多数间歇性故障到第二次重试时就已恢复
MAX_RETRIES = 3
```

**不佳的示例：含义不明的数字**

```python  theme={null}
TIMEOUT = 47  # 为什么是 47？
RETRIES = 5   # 为什么是 5？
```

### 提供工具脚本

即使代理能够自己编写脚本，随技能提供现成脚本也有优势：

* 比即时生成的代码可靠。
* 节省 token，因为无需把代码载入上下文。
* 节省时间，因为无需生成代码。
* 保证每次使用的行为一致。

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="将可执行脚本与指令文件一同打包" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

上图展示了可执行脚本如何与指令文件配合。指令文件（forms.md）引用脚本，代理可以直接执行，无需把脚本内容载入上下文。

在指令中明确告诉代理应如何使用脚本：

* **执行脚本**（最常见），例如「运行 `analyze_form.py` 提取字段」。
* **作为参考读取脚本**（适用于复杂逻辑），例如「字段提取算法见 `analyze_form.py`」。

大多数工具脚本适合直接执行，可靠性和效率都更高。脚本执行方式见下文的[运行环境](#runtime-environment)。

示例：

````markdown  theme={null}
## 工具脚本

**analyze_form.py**：提取 PDF 中的所有表单字段。

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

输出格式：
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**：检查边界框是否重叠。

```bash
python scripts/validate_boxes.py fields.json
# 返回 "OK"，或列出冲突
```

**fill_form.py**：将字段值写入 PDF。

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### 使用视觉分析

如果输入可以渲染为图像，可以让代理分析图像：

````markdown  theme={null}
## 分析表单布局

1. 将 PDF 转成图像：
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. 分析每页图像，识别表单字段。
3. 代理可以直接从图像判断字段的位置和类型。
````

<Note>
  此例中需要自行编写 `pdf_to_images.py` 脚本。
</Note>

代理的视觉能力可以帮助理解布局和结构。

### 生成可验证的中间产物

代理执行复杂的开放式任务时，可能出错。先让代理用结构化格式生成计划，再用脚本验证计划，通过后才执行，可以及早发现问题。

例如，让代理根据电子表格更新 PDF 中的 50 个表单字段。如果没有验证，代理可能引用不存在的字段、产生冲突的值、遗漏必填字段，或用错误的方式应用更新。

可以沿用上文填写 PDF 表单的流程，再加入一个中间文件 `changes.json`，在应用改动前验证。流程变为分析 → **生成计划文件** → **验证计划** → 执行 → 验证。

这种做法具有以下特点：

* **及早发现错误**，在应用更改之前就能发现问题。
* **可由机器验证**，脚本可依据客观规则校验。
* **可以反复调整计划**，代理无需动原始文件。
* **方便排查**，错误消息能指向具体问题。

适用于批量操作、破坏性改动、验证规则复杂的任务，以及后果重大的操作。

编写验证脚本时，报错应具体，例如「找不到字段 'signature_date'。可用字段：customer_name、order_total、signature_date_signed」。代理可以据此修复问题。

### 软件包依赖

技能在代码执行环境中运行，不同平台的限制有所不同：

* **claude.ai**：可从 npm 和 PyPI 安装软件包，也可从 GitHub 仓库拉取内容。
* **Anthropic API**：无法访问网络，也无法在运行时安装软件包。

在 SKILL.md 中列出所需的软件包，并通过[代码执行工具文档](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool)确认它们是否可用。

<a id="runtime-environment"></a>

### 运行环境

技能在代码执行环境中运行，可以访问文件系统、运行 bash 命令并执行代码。相关架构的概念见技能概览中的[技能架构](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#the-skills-architecture)。

代理按以下方式访问技能：

1. **预加载元数据。** 启动时，所有技能 YAML 前置元数据中的名称和描述都会载入系统提示词。
2. **按需读取文件。** 代理在需要时使用文件读取工具从文件系统访问 SKILL.md 和其他文件。
3. **高效执行脚本。** 工具脚本可以通过 bash 执行，无需将完整内容载入上下文。只有脚本输出会消耗 token。
4. **大型文件按需占用上下文。** 参考文件、数据和文档在实际读取之前不会消耗上下文 token。

编写时，应注意以下几点：

* **文件路径。** 代理像浏览文件系统一样浏览技能目录。路径一律使用正斜杠（`reference/guide.md`）。
* **文件名要能说明内容。** 例如用 `form_validation_rules.md`，避免用 `doc2.md`。
* **方便查找。** 按领域或功能组织目录。
  * 正确：`reference/finance.md`、`reference/sales.md`。
  * 不佳：`docs/file1.md`、`docs/file2.md`。
* **附带完整资料。** 可以附上完整的 API 文档、大量示例和大型数据集。读取之前不占用上下文。
* **确定性的操作优先用脚本。** 例如直接提供 `validate_form.py`，免去代理生成校验代码。
* **明确脚本的使用方式。**
  * 「运行 `analyze_form.py` 提取字段」（执行）。
  * 「提取算法见 `analyze_form.py`」（作为参考读取）。
* **测试文件的访问路径。** 用真实请求确认代理能顺利浏览目录结构。

示例：

```
bigquery-skill/
├── SKILL.md（概览，指向参考文件）
└── reference/
    ├── finance.md（营收指标）
    ├── sales.md（销售漏斗数据）
    └── product.md（使用情况分析）
```

用户询问营收时，代理读取 SKILL.md，看到指向 `reference/finance.md` 的引用，再调用 bash 读取该文件。sales.md 和 product.md 会留在文件系统中，只有实际读取时才占用上下文 token。基于文件系统的结构实现了渐进式披露，让代理按每项任务的需要选择要加载的资料。

完整的技术架构见技能概览中的[技能如何运作](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

### 引用 MCP 工具

如果技能使用 MCP（Model Context Protocol）工具，始终使用带服务器前缀的完整工具名，以避免报找不到工具的错误（`tool not found`）。

格式为 `ServerName:tool_name`。

示例：

```markdown  theme={null}
使用 BigQuery:bigquery_schema 工具获取表结构。
使用 GitHub:create_issue 工具创建议题。
```

`BigQuery` 和 `GitHub` 是 MCP 服务器名。`bigquery_schema` 和 `create_issue` 是相应服务器中的工具名。

省略服务器前缀时，代理可能找不到工具，尤其是在有多个 MCP 服务器可用时。

### 不要假定工具已经安装

明确说明需要哪些软件包。

````markdown  theme={null}
**不佳的示例：假定软件包已安装**
「用 pdf 库处理文件。」

**好的示例：明确说明依赖**
「安装必需的软件包：`pip install pypdf`

接下来按以下方式使用：
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```
」
````

## 技术说明

### YAML 前置元数据要求

SKILL.md 的前置元数据必须包含 `name`（最多 64 字符）和 `description`（最多 1024 字符）。完整结构见[技能概览](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#skill-structure)。

### token 预算

为获得最佳效果，将 SKILL.md 正文控制在 500 行以内。如果内容超出此上限，按上文的渐进式披露模式拆分到单独的文件。架构细节见[技能概览](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work)。

<a id="checklist-for-effective-skills"></a>

## 有效技能的检查清单

共享技能之前，请逐项核对。

### 基本质量

* [ ] 描述具体且包含关键词
* [ ] 描述写明技能用途和适用条件
* [ ] SKILL.md 正文少于 500 行
* [ ] 按需将额外细节放进独立文件
* [ ] 无时效性信息，或已移到「旧用法」一节
* [ ] 全文术语一致
* [ ] 示例具体
* [ ] 从 SKILL.md 到引用文件仅有一层
* [ ] 恰当地使用渐进式披露
* [ ] 工作流程有明确的步骤

### 代码和脚本

* [ ] 脚本自行处理问题
* [ ] 错误处理明确，能帮助代理解决问题
* [ ] 所有常量都说明了取值依据
* [ ] 指令列出所需软件包，并验证过可用性
* [ ] 脚本有清晰的文档
* [ ] 路径全部用正斜杠
* [ ] 关键操作有验证步骤
* [ ] 质量要求高的任务有反馈循环

### 测试

* [ ] 创建了至少三个评估
* [ ] 用 Haiku、Sonnet 和 Opus 测试过
* [ ] 用真实场景测试过
* [ ] 已纳入团队反馈（如适用）

## 后续步骤

<CardGroup cols={2}>
  <Card title="Agent Skills 入门" icon="rocket" href="https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart">
    创建你的第一个技能
  </Card>

  <Card title="在 Claude Code 中使用技能" icon="terminal" href="https://code.claude.com/docs/en/skills">
    在 Claude Code 中创建和管理技能
  </Card>

  <Card title="通过 API 使用技能" icon="code" href="https://platform.claude.com/docs/en/build-with-claude/skills-guide">
    通过程序上传和使用技能
  </Card>
</CardGroup>
