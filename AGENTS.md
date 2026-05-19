# EvoWiki — LLM Agent Constitution

> This file governs how the LLM agent maintains the EvoWiki knowledge base.
> It is the schema, the conventions, and the workflows.
> The human curates sources and asks questions. The LLM handles everything else.
> Update this file as workflows evolve.

---

## 1. Philosophy

EvoWiki is a **compiled knowledge base**, not a retrieval system.

- **Raw sources** (`raw/`) are immutable truth. Never modify them.
- **The wiki** (`wiki/`) is the LLM's domain. Create, update, cross-reference, and maintain it without human micromanagement.
- **The human** curates sources, directs analysis, asks questions, and reviews. The LLM does all bookkeeping.

Key principle: **One raw source should explode into multiple wiki pages.** Extract every entity, concept, and relationship worth keeping. Do not compress knowledge into a single summary page.

---

## 2. Directory Structure

```
EvoWiki/
├── AGENTS.md              # This file
├── README.md              # Human-facing project docs
├── raw/                   # Immutable source documents
│   └── assets/            # Images and attachments
├── wiki/                  # LLM-maintained knowledge network
│   ├── index.md           # Content catalog
│   ├── log.md             # Chronological log
│   ├── domains/           # Domain tag pages (auto-managed)
│   │   └── _registry.md   # Domain registry
│   ├── sources/           # Source summaries (1 raw → 1 source)
│   ├── entities/          # Named entities (people, papers, models, orgs...)
│   ├── concepts/          # Abstract concepts (algorithms, theories...)
│   ├── artifacts/         # Tangible outputs (code, datasets, tools...)
│   └── relations/         # Synthesized relations (comparisons, evolutions...)
└── tools/                 # Helper scripts (if any)
```

---

## 3. Page Types & Responsibilities

Every wiki page MUST have a `type` in its frontmatter.

| Type | Slug Location | Purpose | Example |
|------|--------------|---------|---------|
| `source` | `wiki/sources/` | Summary of one raw document | `wiki/sources/2026-05-17_attention_is_all_you_need.md` |
| `entity` | `wiki/entities/` | Named, identifiable things | `wiki/entities/ashish_vaswani.md`, `wiki/entities/gpt_4.md` |
| `concept` | `wiki/concepts/` | Abstract ideas and mechanisms | `wiki/concepts/transformer.md`, `wiki/concepts/rlhf.md` |
| `artifact` | `wiki/artifacts/` | Code, data, tools, products | `wiki/artifacts/llama_cpp.md` |
| `relation` | `wiki/relations/` | Cross-cutting syntheses | `wiki/relations/transformer_vs_rnn.md` |
| `domain` | `wiki/domains/` | Domain tag pages | `wiki/domains/nlp.md` |

### 3.1 Source Pages

- One page per raw document.
- Preserve nuance. Include key claims, methodology, results, and limitations.
- Link to all extracted entities and concepts.
- Status typically starts as `draft`, moves to `reviewed` after human check.

### 3.2 Entity Pages

- One page per significant named entity.
- Include: definition, key attributes, related entities, history.
- Use `aliases:` frontmatter for alternate names (e.g., "GPT-4" / "gpt-4-turbo").

### 3.3 Concept Pages

- One page per significant abstract concept.
- Include: definition, intuition, mathematical formulation (if applicable), key papers, related concepts.
- Distinguish between "what it is" and "how it works".

### 3.4 Relation Pages

- Created when cross-document or cross-concept synthesis is needed.
- Common patterns: `X_vs_Y`, `evolution_of_X`, `X_and_Y_debate`, `survey_of_X`.
- MUST cite source pages via `[[...]]`.
- Use confidence markers:
  - ✅ **Established**: Multiple sources agree
  - 🟡 **Working Hypothesis**: Single source or LLM inference
  - ❓ **Open Question**: Explicitly unresolved

### 3.5 Domain Pages

- Auto-created when a new domain tag is first used.
- Contains: domain definition, list of member pages (auto-maintained), related domains.
- Registered in `wiki/domains/_registry.md`.

---

## 4. YAML Frontmatter Specification

Every wiki page MUST start with:

```yaml
---
title: "Human-Readable Title"
type: source | entity | concept | artifact | relation | domain
created: YYYY-MM-DD
updated: YYYY-MM-DD
domains: [domain_tag_1, domain_tag_2]  # Use kebab-case or snake_case consistently
sources: [raw/filename.md]             # For non-source pages, list contributing sources
aliases: ["Alt Name", "Abbreviation"]  # Optional, for entities/concepts
status: draft | reviewed | mature | stale
confidence: established | working-hypothesis | speculative
---
```

### Field Rules

- `domains`: Use `snake_case`. If unsure about domain name, check `_registry.md` first. Create new domain only when genuinely novel.
- `sources`: Always link back to `raw/` files or `wiki/sources/` pages. Enables provenance tracking.
- `status`:
  - `draft`: Just created, not yet human-reviewed
  - `reviewed`: Human has checked and approved
  - `mature`: Stable, cross-referenced, no known issues
  - `stale`: May be outdated, flagged by lint
- `confidence`:
  - `established`: Cross-validated by multiple sources
  - `working-hypothesis`: Supported by limited evidence or inference
  - `speculative`: Explicitly tentative, needs validation

---

## 5. Naming Conventions

### File Names

- Use `snake_case`.
- Prefix source files with date: `YYYY-MM-DD_descriptive_slug.md`
- Be descriptive but concise: `attention_is_all_you_need.md`, not `paper1.md`

### WikiLinks

- Use Obsidian-style `[[Page Name]]` for internal links.
- Use `[[Page Name|Display Text]]` for custom display text.
- Prefer linking to the canonical entity/concept page rather than source page when discussing general ideas.

### Domain Tags

- `snake_case`: `deep_learning`, `natural_language_processing`
- Keep them stable. Prefer broader domains over hyper-specific ones.

---

## 6. Workflows

### 6.1 INGEST — Add a New Source

**Trigger**: Human drops file into `raw/` and says "ingest this".

**Phase 1: Discuss (Human + LLM)**

1. Read the raw source completely.
2. Extract a **candidate list**:
   - Entities mentioned (people, models, organizations, papers...)
   - Concepts discussed (algorithms, theories, mechanisms...)
   - Artifacts referenced (code, datasets, tools...)
   - Potential relations (comparisons, contradictions, evolutions...)
   - Domain tags (what fields does this touch?)
3. Present findings to human:
   - "I found 3 new entities: X, Y, Z. Should I create pages for all?"
   - "Concept 'speculative decoding' is mentioned but not yet in wiki. Create?"
   - "This paper contradicts `wiki/relations/foo_vs_bar.md` on point X. Flag?"
4. Incorporate human feedback.

**Phase 2: Write (LLM Auto)**

1. Create `wiki/sources/YYYY-MM-DD_<slug>.md` with full summary.
2. For each approved entity: create or update `wiki/entities/<slug>.md`.
3. For each approved concept: create or update `wiki/concepts/<slug>.md`.
4. For each approved artifact: create or update `wiki/artifacts/<slug>.md`.
5. For each identified relation: create or update `wiki/relations/<slug>.md`.
6. Update all affected pages with new cross-references (`[[...]]`).
7. Handle domains:
   - For each `domain` in page frontmatters:
     - If exists: update `wiki/domains/<domain>.md` to include new page
     - If new: create `wiki/domains/<domain>.md`, register in `_registry.md`
8. Update `wiki/index.md`:
   - Add new pages under appropriate categories
   - Ensure one-line summary per page
9. Append to `wiki/log.md`:
   ```markdown
   ## [YYYY-MM-DD] ingest | <Source Title>
   - Added: [[source/page]], [[entity/page]], [[concept/page]] ...
   - Updated: [[entity/old_page]], [[concept/old_page]] ...
   - Domains touched: [domain1, domain2]
   ```

**Phase 3: Mini-Lint (Auto)**

After writing, perform a focused lint on affected pages:
- Check for contradictions with existing pages
- Check for orphan pages created in this ingest
- Check for broken `[[links]]`
- Report findings to human

### 6.2 QUERY — Answer Questions

**Trigger**: Human asks a question.

1. Read `wiki/index.md` to identify relevant pages.
2. Read relevant `wiki/` pages (not raw sources).
3. Synthesize answer with citations: `According to [[Entity Page]] and [[Concept Page]]...`
4. If the answer reveals a valuable new relation or insight:
   - Ask human: "This answer reveals an interesting connection. Should I save it as a relation page?"
   - If yes: create `wiki/relations/<slug>.md` and update index/log.

### 6.3 LINT — Health Check

**Trigger**: Human requests, or auto-triggered every ~10 ingests.

**Lint Checklist**:

- [ ] **Contradictions**: Scan for conflicting claims across pages. Flag with confidence markers.
- [ ] **Stale Claims**: Identify pages with `status: mature` but older `updated` dates than newer contradicting sources.
- [ ] **Orphan Pages**: Find pages with zero inbound `[[links]]`. Decide: integrate better, or mark for deletion.
- [ ] **Missing Concept Pages**: Find concepts mentioned in `[[...]]` but without dedicated pages.
- [ ] **Broken Links**: Find `[[Non-existent Page]]` references.
- [ ] **Domain Drift**: Check `_registry.md` for near-duplicate domains. Suggest merges.
- [ ] **Confidence Audit**: Ensure `speculative` claims haven't been validated (or refuted) by newer sources.

**Lint Output**:

Generate a markdown report and append summary to `wiki/log.md`:
```markdown
## [YYYY-MM-DD] lint | Full Health Check
- Contradictions found: N (list pages)
- Orphan pages: N (list pages)
- Missing concept pages: N (list suggested)
- Broken links: N (list)
- Suggested actions: ...
```

Present report to human. Execute fixes only after human confirmation.

---

## 7. Domain Management Rules

Domains are **emergent**, not preset.

1. **Before creating a new domain**: Check `wiki/domains/_registry.md` for existing near-matches.
2. **New domain threshold**: Only create if the concept space is genuinely distinct and will likely have multiple pages.
3. **On creation**:
   - Create `wiki/domains/<new_domain>.md` with definition and member list
   - Add entry to `_registry.md`
   - Log in `wiki/log.md`
4. **Merges**: When human identifies duplicate domains:
   - Pick canonical name
   - Update all affected page frontmatters
   - Delete redundant domain page
   - Update `_registry.md`

---

## 8. Writing Style

- **Concise but complete**: Prefer short paragraphs. Use bullet points for lists.
- **Progressive disclosure**: Start with a one-paragraph summary, then expand.
- **Cite everything**: Every claim should trace back to sources via `[[...]]`.
- **Use headers**: H2 for major sections, H3 for subsections. Keep pages scannable.
- **Confidence honesty**: Mark speculative content explicitly. Don't present inference as fact.
- **Cross-reference liberally**: If a concept is mentioned, link it: `[[concept_name]]`.

---

## 9. Prohibited Actions

- **NEVER modify `raw/` files.** They are immutable.
- **NEVER delete a page without human confirmation** (except obvious duplicates created in same session).
- **NEVER create empty stub pages.** Every page must have meaningful content on creation.
- **NEVER ignore contradictions.** Flag them explicitly.
- **NEVER use absolute file paths** in wiki content. Use relative `[[WikiLinks]]` or relative paths.

---

## 10. Human Review Points

The LLM MUST pause for human input at:

1. **Post-ingest discussion**: Present candidate entities/concepts before writing
2. **Contradiction discovery**: Always flag, never silently overwrite
3. **Relation page creation from queries**: Ask before filing exploratory answers
4. **Lint fix execution**: Present plan, confirm before bulk edits

---

## 11. Changelog

Track significant schema changes here:

| Date | Change |
|------|--------|
| 2026-05-17 | Initial schema established |
