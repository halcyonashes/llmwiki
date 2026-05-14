# LLM Wiki Schema

You are the maintainer of this wiki. You read sources, synthesize knowledge, and write/update markdown files. You never modify files in `.claude/raw/`. You own everything in `.claude/wiki/`. This document is your operating manual.

---

## Directory Structure

```
<project>/
├── CLAUDE.md              ← this file (schema + operating manual, auto-loaded by Claude Code)
├── README.md              ← optional project readme
└── .claude/               ← add this to .gitignore to keep the wiki private
    ├── raw/               ← immutable source documents (you read, never modify)
    │   └── assets/        ← images and attachments
    └── wiki/              ← you own this entire directory
        ├── index.md       ← master catalog of all wiki pages
        ├── log.md         ← append-only chronological record
        ├── overview.md    ← high-level synthesis of everything ingested so far
        ├── entities/      ← one page per named entity (person, org, product, etc.)
        ├── concepts/      ← one page per concept, idea, or theme
        ├── sources/       ← one summary page per ingested source
        └── output/        ← analyses, comparisons, responses filed as pages
```

---

## Session Start Protocol (MANDATORY)

At the start of every session, before doing anything else:

1. Read `.claude/wiki/log.md` (last 20 entries) to understand recent activity.
2. Read `.claude/wiki/index.md` to know what pages exist.
3. Report to the user: what was done last session, what's in the wiki, what's ready to work on.

---

## Page Format

Every wiki page (except `index.md` and `log.md`) uses this frontmatter:

```yaml
---
title: "Page Title"
type: entity | concept | source | output
tags: [tag1, tag2]
sources: [source-slug-1, source-slug-2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### Body conventions

- Use `[[Page Title]]` for internal wiki links (Obsidian-compatible).
- Use `> [!NOTE]` callouts for important caveats or contradictions.
- Use `> [!WARNING]` for claims that contradict other sources.
- End every page with a `## Sources` section listing the raw sources it draws from.
- End every entity/concept page with a `## See Also` section with related links.

---

## Operations

### 1. Ingest a Source

When the user says "ingest [file]" or drops a new file in `.claude/raw/`:

1. Read the source file from `.claude/raw/`.
2. Discuss key takeaways with the user (ask what to emphasize if unclear).
3. Create `.claude/wiki/sources/<slug>.md` — a structured summary page.
4. Identify all entities mentioned → update or create `.claude/wiki/entities/<name>.md`.
5. Identify all concepts → update or create `.claude/wiki/concepts/<name>.md`.
6. Update `.claude/wiki/overview.md` if the source shifts the overall picture.
7. Update `.claude/wiki/index.md` — add the new source page and any new entity/concept pages.
8. Append to `.claude/wiki/log.md`:
   ```
   ## [YYYY-MM-DD] ingest | <Source Title>
   Pages created: sources/<slug>.md, entities/<x>.md (updated), ...
   Key claims: <1-2 sentences>
   Contradictions: <any found, or "none">
   ```

**Source summary page template** (`.claude/wiki/sources/<slug>.md`):
```markdown
---
title: "<Source Title>"
type: source
tags: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <Source Title>

**Author**: ...  
**Date**: ...  
**Format**: article | paper | book | transcript | data | other  
**Source file**: `.claude/raw/<filename>`

## Summary
<2-4 paragraph synthesis>

## Key Claims
- Claim 1
- Claim 2

## Entities Mentioned
- [[Entity Name]] — role in this source
- [[Entity Name 2]] — role in this source

## Concepts
- [[Concept Name]] — how it appears here
- [[Concept Name 2]]

## Contradictions & Gaps
- Contradicts [[Other Source]] on: ...
- Gap: ...

## Raw Notes
<optional: verbatim excerpts or data worth preserving>

## Sources
- `.claude/raw/<filename>`
```

---

### 2. Answer a Query

When the user asks a question:

1. Read `.claude/wiki/index.md` to identify relevant pages.
2. Read those pages and synthesize an answer.
3. Cite wiki pages inline with `[[Page Title]]`.
4. If the answer is valuable enough to keep (complex synthesis, comparison, analysis) → **file it** in `.claude/wiki/output/<slug>.md`.
5. Append to `.claude/wiki/log.md`:
   ```
   ## [YYYY-MM-DD] query | <Question (truncated)>
   Answer filed: output/<slug>.md (or "not filed")
   Pages consulted: [list]
   ```

**Output page template** (`.claude/wiki/output/<slug>.md`):
```markdown
---
title: "<Question or Analysis Title>"
type: output
tags: []
sources: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <Title>

**Question**: <original question>  
**Date**: YYYY-MM-DD

## Answer
<synthesis>

## Sources Consulted
- [[Wiki Page 1]]
- [[Wiki Page 2]]
```

---

### 3. Lint the Wiki

When the user says "lint" or "health check":

1. Read all pages in `.claude/wiki/`.
2. Check for and report:
   - **Contradictions**: claims that conflict across pages
   - **Orphan pages**: pages with no inbound `[[links]]`
   - **Stale claims**: pages referencing "recent" events that may be outdated
   - **Missing pages**: entities or concepts mentioned in `[[links]]` that have no page
   - **Gaps**: important topics not yet covered
   - **Data gaps**: questions that a web search could answer
3. For each issue found, either fix it inline or flag it to the user.
4. Append to `.claude/wiki/log.md`:
   ```
   ## [YYYY-MM-DD] lint
   Issues found: <count>
   Fixed: [list]
   Flagged for user: [list]
   ```

---

## Index Format (`.claude/wiki/index.md`)

Update this file on every ingest. Keep it sorted by category:

```markdown
# Wiki Index

Last updated: YYYY-MM-DD | Sources ingested: N | Total pages: N

## Overview
- [[overview]] — High-level synthesis of all ingested knowledge

## Sources
| Page | Summary | Date |
|------|---------|------|
| [[sources/slug]] | One-line summary | YYYY-MM-DD |

## Entities
| Page | Summary |
|------|---------|
| [[entities/name]] | One-line description |

## Concepts
| Page | Summary |
|------|---------|
| [[concepts/name]] | One-line description |

## Output (Queries & Analyses)
| Page | Question / Topic | Date |
|------|-----------------|------|
| [[output/slug]] | One-line summary | YYYY-MM-DD |
```

---

## Log Format (`.claude/wiki/log.md`)

Append only. Never edit past entries. Format each entry as:

```
## [YYYY-MM-DD] <operation> | <title>
<1-3 lines of detail>
```

Operations: `ingest`, `query`, `lint`, `update`, `note`

Parseable with: `grep "^## \[" .claude/wiki/log.md | tail -10`

---

## Conventions

- **Slugs**: lowercase, hyphens, no spaces. `ai-safety.md`, `anthropic.md`, `transformer-architecture.md`
- **Entity pages**: one page per distinct named thing. Merge duplicates aggressively.
- **Concept pages**: one page per distinct idea. Split when a page exceeds ~600 lines.
- **Cross-references**: always link with `[[Page Title]]`, not bare file paths.
- **Contradictions**: mark with `> [!WARNING]` in the body, and note in both pages involved.
- **Images**: store in `.claude/raw/assets/`. Reference as `![[assets/filename.png]]` in wiki pages.
- **Frontmatter**: keep `updated:` current whenever you touch a page.

---

## What You Never Do

- Modify any file in `.claude/raw/`.
- Delete wiki pages without telling the user why.
- Leave a `[[link]]` pointing to a non-existent page without flagging it.
- Skip the log entry for any operation.
