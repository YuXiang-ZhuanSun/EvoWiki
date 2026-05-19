# EvoWiki 使用指南

> 本文档面向 EvoWiki 的用户，教你从零开始搭建并维护自己的自演进知识库。
>
> 如果你是开发者，想了解项目架构和设计理念，请阅读 [README.md](./README.md)。
> 如果你是 LLM Agent，想了解维护规范，请阅读 [AGENTS.md](./AGENTS.md)。

---

## 目录

- [1. 五分钟快速开始](#1-五分钟快速开始)
- [2. 核心概念](#2-核心概念)
- [3. 准备原始资料](#3-准备原始资料)
- [4. Ingest：把资料"编译"进知识库](#4-ingest把资料编译进知识库)
- [5. Query：向知识库提问](#5-query向知识库提问)
- [6. Lint：定期健康检查](#6-lint定期健康检查)
- [7. 目录结构详解](#7-目录结构详解)
- [8. 页面类型与规范](#8-页面类型与规范)
- [9. 领域标签（Domain Tags）](#9-领域标签domain-tags)
- [10. 与 Obsidian 配合使用](#10-与-obsidian-配合使用)
- [11. 最佳实践](#11-最佳实践)
- [12. 常见问题](#12-常见问题)

---

## 1. 五分钟快速开始

### 1.1 环境准备

你只需要：

- **Git**：版本控制
- **一个 LLM Agent**：支持文件读写的 AI 助手（如 Claude Code、Kimi CLI、OpenAI Codex 等）
- **文本编辑器**：VS Code、Obsidian，或任何能看 Markdown 的编辑器

### 1.2 克隆项目

```bash
git clone https://github.com/YuXiang-ZhuanSun/EvoWiki.git
cd EvoWiki
```

### 1.3 第一次 Ingest（示例流程）

**Step 1**：找一篇你最近在看的文章，保存为 Markdown 文件放入 `raw/`：

```bash
# 假设你有一篇文章
cp ~/Downloads/my_article.md raw/2026-05-17_my_article.md
```

**Step 2**：告诉你的 LLM Agent：

> "请 ingest 这份资料：raw/2026-05-17_my_article.md"

**Step 3**：LLM 会：
1. 读文章，提取关键信息
2. **和你讨论**："我发现三个实体、两个新概念，是否创建页面？"
3. 你确认后，自动写入 `wiki/`
4. 更新索引、日志，并做迷你检查

**Step 4**：打开 `wiki/` 查看生成的知识网络。

---

## 2. 核心概念

### 2.1 为什么不用 RAG？

传统 RAG（检索增强生成）每次提问都让 LLM 重新读原始文档、重新理解、重新组织答案。这就像**每次写报告都让实习生把图书馆重新读一遍**。

EvoWiki 的思路是**"编译"**：
- 读资料时，把知识**提取、结构化、交叉链接**
- 提问时，直接读**已整理好的 Wiki 页面**
- 知识越积越多，查询反而越来越快

### 2.2 三层架构

```
Schema 层（AGENTS.md）
    ↑ 定义规则
Wiki 层（wiki/）  ← 你提问时读这层
    ↑ 编译、维护
Raw 层（raw/）    ← 原始资料，只读不写
```

**你的角色**：策展人 + 提问者 + 审核员  
**LLM 的角色**：图书管理员 + 编辑 + 校对员

### 2.3 关键文件

| 文件 | 作用 | 谁维护 |
|------|------|--------|
| `wiki/index.md` | 知识库总目录，所有页面的入口 | LLM |
| `wiki/log.md` | 操作时间线，记录了每次 ingest/query/lint | LLM |
| `wiki/domains/_registry.md` | 领域标签注册表 | LLM + 你 |
| `AGENTS.md` | LLM 的行为规范 | 你 + LLM |

---

## 3. 准备原始资料

### 3.1 支持的格式

理论上任何文本都可以，但推荐：

- **Markdown**（`.md`）：最佳，原生支持
- **纯文本**（`.txt`）：可用
- **PDF**：建议先转成 Markdown（可用 `marker`、`pdftotext` 等工具）
- **网页文章**：可用 Obsidian Web Clipper 等浏览器插件转为 Markdown

### 3.2 命名规范

原始资料建议用日期前缀：

```
raw/2026-05-17_attention_is_all_you_need.md
raw/2026-05-18_某博客文章_关于RLHF的反思.md
```

这样 `log.md` 和 `wiki/sources/` 的页面可以一一对应。

### 3.3 图片处理

如果文章包含图片：

1. 把图片放入 `raw/assets/`
2. 在 Markdown 中用相对路径引用：`![图](assets/image_name.png)`
3. LLM 处理时会保留图片引用

> 注意：当前 LLM 读取 Markdown 时可能无法直接"看到"内嵌图片。如果图片包含关键信息，可以单独让 LLM 查看。

---

## 4. Ingest：把资料"编译"进知识库

这是 EvoWiki **最核心的工作流**。

### 4.1 完整流程

```
你放入 raw/ ──→ LLM 读取 ──→ 讨论要点 ──→ 你确认 ──→ LLM 写入 wiki/
                              ↑___________|
                                   |
                              人类介入点
```

### 4.2 Phase 1：讨论（人机协作）

LLM 读完资料后会向你汇报：

```
我从这篇文章中提取了以下内容：

【实体】（是否需要创建/更新页面？）
- Ashish Vaswani（作者）→ 已存在，更新？
- Google Brain（机构）→ 新实体，创建？
- Transformer（模型架构）→ 已存在，更新？

【概念】（是否需要创建/更新页面？）
- Self-Attention（新概念，创建？）
- Multi-Head Attention（新概念，创建？）

【关系】（是否需要创建关系页？）
- 与现有 wiki/concepts/rnn.md 的对比关系 → 更新 relations/？

【领域标签】
- [nlp, deep_learning, architecture]
```

**你的决策**：
- 同意：说"都创建"或"创建 A、B，跳过 C"
- 修改：说"把 X 改成 Y"
- 补充：说"还有一个重要概念 Z 别忘了"

### 4.3 Phase 2：自动写入

你确认后，LLM 会自动：

1. 创建 `wiki/sources/2026-05-17_xxx.md` —— 资料摘要
2. 创建/更新 `wiki/entities/` 下的页面
3. 创建/更新 `wiki/concepts/` 下的页面
4. 创建/更新 `wiki/relations/` 下的关系页
5. 更新 `wiki/index.md`
6. 追加 `wiki/log.md`

**一次 ingest 通常会产生 5~15 个 wiki 页面。**

### 4.4 Phase 3：迷你检查

写入完成后，LLM 会自动检查：

- 本次创建的页面是否与现有页面矛盾？
- 有没有产生"孤儿页面"（没有入链）？
- 链接是否有效？

然后向你汇报检查结果。

### 4.5 批量 Ingest

如果你有一堆资料要处理：

> "请批量 ingest raw/ 目录下所有未处理的文件。"

LLM 会逐篇处理，但讨论环节会压缩为摘要式汇报，提高效率。适合你已经熟悉的内容。

---

## 5. Query：向知识库提问

### 5.1 基本用法

直接问：

> "Transformer 和 RNN 的核心区别是什么？"

LLM 的执行过程：
1. 读 `wiki/index.md` 找到相关页面
2. 读 `wiki/concepts/transformer.md` 和 `wiki/concepts/rnn.md`
3. 读 `wiki/relations/transformer_vs_rnn.md`（如果有）
4. 综合回答，并标注引用：`根据 [[Transformer]] 中的定义...`

### 5.2 归档有价值的回答

如果 LLM 的回答产生了**新的洞察**（比如发现了一个新的关联、形成了一个对比），它会问你：

> "这个回答揭示了一个有趣的联系：A 和 B 在 C 点上其实是相通的。要不要把它保存为 `wiki/relations/xxx.md`，以后可以直接查？"

**建议你同意。** 这样你的探索也会成为知识库的一部分，不断复利增长。

### 5.3 查询格式

你可以要求不同形式的输出：

- **对比表**："请用表格对比 Transformer、RNN、CNN"
- **时间线**："梳理一下 GPT 系列的发展时间线"
- **待办清单**："基于当前知识库，列出我还需要深入了解的概念"
- **Marp 幻灯片**："把 Attention 机制做成一份幻灯片"

---

## 6. Lint：定期健康检查

### 6.1 什么时候需要 Lint？

- **迷你 Lint**：每次 ingest 后自动执行（只检查本次涉及的内容）
- **全面 Lint**：你主动说"检查一下 wiki 健康度"，或每 ingest 10 篇后 LLM 主动提醒

### 6.2 Lint 检查什么？

LLM 会扫描整个 wiki，检查：

| 检查项 | 说明 |
|--------|------|
| 🔍 矛盾 | A 页面说 X，B 页面说非 X |
| 🕰️ 过时 | 旧资料被新资料推翻，但 wiki 未更新 |
| 🔗 孤儿页 | 没有任何页面链接到它，可能冗余 |
| 🕳️ 概念缺口 | 多个页面提到某个概念，但它没有独立页面 |
| 🔨 断链 | `[[不存在的页面]]` |
| 📋 领域漂移 | 领域标签有重复或近义词 |
| ⚠️ 置信度 | `speculative`（推测）的内容是否已被证实/证伪 |

### 6.3 Lint 结果处理

LLM 会生成报告：

```markdown
## Lint 报告（2026-05-20）

- 发现 2 处矛盾
  - [[entities/gpt_4]] 说参数量 1.8T，[[relations/llm_scaling]] 说 8×220B
  - 建议：标注为"估计值存在争议"

- 发现 1 个孤儿页
  - [[concepts/speculative_decoding]] 没有任何入链
  - 建议：在 [[sources/xxx]] 和 [[concepts/transformer]] 中添加链接

- 发现 3 个缺失概念页
  - "Mixture of Experts" 被 5 个页面提及，但没有独立页面
  - 建议：创建 [[concepts/mixture_of_experts]]
```

**你需要确认**后，LLM 才会执行修复。

---

## 7. 目录结构详解

```
EvoWiki/
├── AGENTS.md          # LLM 的行为宪法（你和 LLM 共同维护）
├── README.md          # 项目介绍（给访客看）
├── GUIDE.md           # 本文件（给用户看）
├── raw/               # 📥 原始资料（你只放东西，LLM 只读）
│   ├── assets/        # 图片、附件
│   └── ...            # 你的文章、论文、笔记
├── wiki/              # 🧠 知识库（LLM 全权维护，你阅读和审核）
│   ├── index.md       # 总目录
│   ├── log.md         # 操作日志
│   ├── domains/       # 领域标签页
│   │   ├── _registry.md
│   │   └── nlp.md     # 示例：NLP 领域页
│   ├── sources/       # 资料摘要（1 raw → 1 source）
│   ├── entities/      # 实体页（人物、模型、机构...）
│   ├── concepts/      # 概念页（算法、理论...）
│   ├── artifacts/     # 产物页（代码、工具、数据集...）
│   └── relations/     # 关系页（对比、演进、争议...）
└── tools/             # 🛠️ 小工具（搜索脚本等，可选）
```

---

## 8. 页面类型与规范

### 8.1 每种页面干什么？

| 类型 | 例子 | 内容要点 |
|------|------|----------|
| **source** | `wiki/sources/2026-05-17_attention_is_all_you_need.md` | 这篇资料讲了什么、关键结论、方法论、局限性 |
| **entity** | `wiki/entities/ashish_vaswani.md` | 是谁/是什么、关键属性、历史、相关实体 |
| **concept** | `wiki/concepts/self_attention.md` | 定义、直觉理解、数学表达、关键论文、相关概念 |
| **artifact** | `wiki/artifacts/llama_cpp.md` | 是什么工具、用途、优缺点、相关资源 |
| **relation** | `wiki/relations/transformer_vs_rnn.md` | 对比维度、各自的优劣、适用场景、演进关系 |
| **domain** | `wiki/domains/nlp.md` | 领域定义、包含的页面列表、相关领域 |

### 8.2 YAML Frontmatter 规范

每个 wiki 页面开头必须有：

```yaml
---
title: "页面标题"
type: source | entity | concept | artifact | relation | domain
created: 2026-05-17
updated: 2026-05-17
domains: [domain1, domain2]
sources: [raw/filename.md]
aliases: ["别名", "缩写"]
status: draft | reviewed | mature | stale
confidence: established | working-hypothesis | speculative
---
```

**字段说明**：

- `type`：页面类型（必填）
- `domains`：领域标签（必填，自动管理）
- `sources`：来源追溯（必填，链接到 raw/ 或 source 页）
- `aliases`：别名（可选，用于实体和概念）
- `status`：页面成熟度
  - `draft`：刚创建，未审核
  - `reviewed`：你看过并认可
  - `mature`：稳定、交叉验证过
  - `stale`：可能过时，待更新
- `confidence`：内容置信度
  - `established`：多篇资料交叉验证
  - `working-hypothesis`：有限证据或推理
  - `speculative`：明确待验证

### 8.3 置信度标记（正文中）

在 `relation` 页面或任何包含推断的内容中，使用以下标记：

- ✅ **确立事实**：多篇 source 交叉验证
- 🟡 **工作假设**：单篇 source 或 LLM 推断
- ❓ **开放问题**：明确未解或存在争议

---

## 9. 领域标签（Domain Tags）

### 9.1 什么是领域标签？

领域标签是**动态生长**的分类系统。没有预设列表，完全由 LLM 根据内容自动识别。

### 9.2 如何使用？

**你不需要手动管理。** 每次 ingest 时，LLM 会自动：

1. 判断这篇资料涉及哪些领域
2. 检查 `wiki/domains/_registry.md` 是否已有该领域
3. 如果有，更新该领域页的页面列表
4. 如果没有，新建领域页并注册

### 9.3 领域合并

LLM 有时会产生近义词领域，比如 `nlp` 和 `natural_language_processing`。你需要定期（比如每月）查看 `wiki/domains/_registry.md`，把相近的领域合并。

**合并方法**：
1. 选一个作为规范名
2. 把所有页面的 `domains` 字段更新
3. 删除冗余的领域页
4. 更新 `_registry.md`

### 9.4 命名规范

- 使用 `snake_case`（下划线连接）
- 优先用学科通用名，不用生造词
- 宁宽勿窄：先用 `ai`，内容多了再拆成 `machine_learning`、`deep_learning`

---

## 10. 与 Obsidian 配合使用

EvoWiki 原生兼容 Obsidian，你可以随时迁移。

### 10.1 打开 Vault

1. 下载 [Obsidian](https://obsidian.md/)
2. "Open folder as vault" → 选择 `EvoWiki/`
3. Settings → Files and links → 启用 `"Use [[Wikilinks]]"`

### 10.2 推荐插件

| 插件 | 用途 |
|------|------|
| **Graph View** | 可视化知识网络，看哪些页面是枢纽、哪些是孤儿 |
| **Dataview** | 基于 frontmatter 查询页面（如"列出所有 status:stale 的页面"） |
| **Obsidian Web Clipper** | 一键把网页转为 Markdown 存入 `raw/` |
| **Marp** |
