# EvoWiki 🧠✨

> **Knowledge shouldn't be retrieved. It should evolve.**
> 
> *知识不该被检索，它应该进化。*

<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/Powered%20by-LLM-8A2BE2?style=flat-square&logo=openai&logoColor=white" alt="LLM Powered"></a>
  <a href="#"><img src="https://img.shields.io/badge/Markdown-000000?style=flat-square&logo=markdown&logoColor=white" alt="Markdown"></a>
  <a href="#"><img src="https://img.shields.io/badge/Obsidian%20Ready-7B68EE?style=flat-square&logo=obsidian&logoColor=white" alt="Obsidian Ready"></a>
  <a href="#"><img src="https://img.shields.io/badge/Git%20Managed-F05032?style=flat-square&logo=git&logoColor=white" alt="Git"></a>
  <a href="#"><img src="https://img.shields.io/badge/License-MIT-yellow?style=flat-square" alt="License"></a>
</p>

<p align="center">
  <b>自演进个人知识库 · Self-Evolving Personal Knowledge Base</b>
</p>

---

## 📖 What is EvoWiki?

EvoWiki is a **self-evolving, LLM-maintained personal knowledge base** inspired by [Andrej Karpathy's llm-wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

Unlike traditional RAG systems where the LLM rediscovers knowledge from scratch on every query, EvoWiki **compiles knowledge once and keeps it current**. When you add a new source, the LLM doesn't just index it—it reads it, extracts key information, and integrates it into an interconnected web of markdown pages. Cross-references are already there. Contradictions are already flagged. The synthesis already reflects everything you've read.

**The wiki is a persistent, compounding artifact.**

### Why Not RAG?

| | Traditional RAG | EvoWiki |
|---|---|---|
| **Mental Model** | Search engine | Compiler |
| **Per Query** | Re-read raw sources → Answer | Read compiled wiki → Answer |
| **Knowledge State** | Scattered, ephemeral | Structured, persistent |
| **Cross-Document Reasoning** | Re-derived every time | Pre-compiled into relations |
| **Maintenance Cost** | Zero (no maintenance) | Near-zero (LLM does it all) |

RAG is **interpreted execution**. EvoWiki is **compiled execution**.

---

## 🏗️ Architecture

EvoWiki has three layers:

```
┌─────────────────────────────────────────┐
│  Schema Layer: AGENTS.md                │
│  (The "constitution" — conventions,     │
│   workflows, page formats)              │
├─────────────────────────────────────────┤
│  Wiki Layer: wiki/                      │
│  (LLM-generated, interlinked markdown   │
│   — entities, concepts, relations)      │
├─────────────────────────────────────────┤
│  Raw Layer: raw/                        │
│  (Immutable source documents — papers,  │
│   articles, notes, images)              │
└─────────────────────────────────────────┘
```

### Directory Structure

```
EvoWiki/
├── AGENTS.md                  # LLM behavior constitution
├── README.md                  # This file — human documentation
├── raw/                       # 📥 Immutable source documents
│   └── assets/                # Images and attachments from sources
├── wiki/                      # 🧠 LLM-maintained knowledge network
│   ├── index.md               # Content catalog (updated per ingest)
│   ├── log.md                 # Append-only chronological log
│   ├── domains/               # Dynamic domain tags
│   │   └── _registry.md       # Domain registry
│   ├── sources/               # Source summaries (1 raw → 1 source page)
│   ├── entities/              # Named entities (people, papers, models, orgs...)
│   ├── concepts/              # Abstract concepts (algorithms, theories, patterns...)
│   ├── artifacts/             # Tangible outputs (code, datasets, tools...)
│   └── relations/             # Synthesized relations (comparisons, evolutions, debates...)
└── tools/                     # 🛠️ Optional helper scripts
```

Every wiki page carries **YAML frontmatter** with metadata:

```yaml
---
title: "Attention Is All You Need"
type: source | entity | concept | artifact | relation | domain
created: 2026-05-17
updated: 2026-05-17
domains: [nlp, deep_learning, architecture]      # Auto-managed by LLM
sources: [raw/2026-05-17_paper_attention_is_all_you_need.md]
status: draft | reviewed | mature | stale
confidence: established | working-hypothesis | speculative
---
```

---

## 🚀 Workflows

### 1. Ingest — Add Knowledge

Drop a new source into `raw/` and tell your LLM agent to **ingest** it.

**Phase 1: Discuss** (Human + LLM)
- LLM reads the raw source
- Extracts candidate entities, concepts, and relations
- Proposes discussion questions: *"Should I create a new concept page for 'speculative decoding'?"*
- You confirm, modify, or veto

**Phase 2: Write** (LLM Auto)
- Creates/updates `wiki/sources/`
- Creates/updates `wiki/entities/` and `wiki/concepts/`
- Creates/updates `wiki/relations/` for cross-cutting insights
- Updates `wiki/domains/` (auto-creates new domain tags when needed)
- Updates `wiki/index.md`
- Appends to `wiki/log.md`

> **One raw source often explodes into 5–15 wiki pages.**

### 2. Query — Ask Questions

Ask questions **against the wiki**, not against raw sources.

1. LLM reads `wiki/index.md` to locate relevant pages
2. Drills into specific pages for detail
3. Synthesizes an answer with citations (`[[Page Name]]`)
4. **Valuable answers are filed back into `wiki/relations/`** — your explorations compound

### 3. Lint — Health Check

Periodically ask the LLM to **lint** the wiki:

- 🔍 **Contradictions**: Page A says X, Page B says ¬X
- 🕰️ **Stale claims**: Newer sources have superseded old ones
- 🔗 **Orphan pages**: No inbound links — should they exist?
- 🕳️ **Concept gaps**: Important ideas mentioned but lacking dedicated pages
- 🔨 **Broken links**: `[[Non-existent Page]]`

**Lint Frequency**: Mini-lint runs automatically after each ingest. Full lint is suggested every ~10 ingests.

---

## 🏷️ Domain Tags — Auto-Managed

EvoWiki has **no preset domains**. The LLM dynamically discovers and manages them:

- Every wiki page declares its `domains:` in frontmatter
- `wiki/domains/_registry.md` lists all known domains with definitions
- When encountering a truly new domain, the LLM:
  1. Checks `_registry.md` for near-duplicates
  2. Creates `wiki/domains/<new-domain>.md`
  3. Registers it in `_registry.md`
  4. Logs the creation in `wiki/log.md`

You review `_registry.md` periodically to merge near-duplicate domains.

---

## 📦 Getting Started

### Prerequisites

- Git
- Any text editor (VS Code, Neovim, or Obsidian for the full graph-view experience)
- An LLM agent with file-editing capabilities (Claude Code, Codex, Kimi CLI, etc.)

### Quick Start

```bash
# Clone the repo
git clone https://github.com/YuXiang-ZhuanSun/EvoWiki.git
cd EvoWiki

# Your knowledge base is ready. Start ingesting:
# 1. Drop a source into raw/
# 2. Tell your LLM agent: "Ingest raw/my_new_article.md"
# 3. Discuss, confirm, let the LLM write
# 4. Browse wiki/ to see the network grow
```

### Obsidian Migration (Optional)

EvoWiki uses standard Markdown with `[[WikiLinks]]`. To open in Obsidian:
1. Open `EvoWiki/` as a vault
2. Settings → Files and links → Turn on `[[WikiLinks]]`
3. Enjoy the graph view as your knowledge network evolves

---

## 🧬 Design Philosophy

> *"The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero."*
>
> — Andrej Karpathy

**Human's job**: Curate sources, direct analysis, ask good questions, think about what it all means.

**LLM's job**: Everything else — summarizing, cross-referencing, filing, bookkeeping, linting.

---

## 🗺️ Roadmap

- [x] Core architecture and schema (`AGENTS.md`)
- [x] Multi-domain support with auto-managed tags
- [x] Hybrid ingest workflow (discuss → write → review)
- [ ] `qmd` integration for large-scale semantic search
- [ ] Marp slide deck generation from wiki content
- [ ] GitHub Actions for automated lint scheduling
- [ ] MCP server for native tool integration

---

## 🤝 Contributing

This is a personal knowledge base, but the pattern is universal.

If you have ideas for improving the schema, the workflows, or the tooling, open an issue or PR. The `AGENTS.md` is a living document — it evolves as we learn what works.

---

## 📜 License

MIT License — use the pattern, build your own wiki, evolve your own knowledge.

---

## 🙏 Acknowledgments

- **[Andrej Karpathy](https://karpathy.ai/)** for the [llm-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) that started this journey
- **[Vannevar Bush](https://en.wikipedia.org/wiki/Vannevar_Bush)** for the Memex vision (1945) — a personal, curated knowledge store with associative trails

---

<p align="center">
  <i>Build once. Evolve forever. 🔥</i>
</p>
