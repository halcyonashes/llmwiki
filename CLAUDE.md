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
        ├── output/        ← analyses, comparisons, responses filed as pages
        └── context/       ← codebase context map (file index + module deep-dives)
            ├── graph.md   ← master map: every file → purpose → key exports
            └── <module>.md ← deep page per complex module (on demand)
```

---

## Session Start Protocol (MANDATORY)

At the start of every session, before doing anything else:

1. Read `.claude/wiki/log.md` (last 20 entries) to understand recent activity.
2. Read `.claude/wiki/index.md` to know what pages exist.
3. If `.claude/wiki/context/graph.md` exists, read it — load the file index into context before doing any code work.
4. Report to the user: what was done last session, what's in the wiki, what's ready to work on.

---

## Page Format

Every wiki page (except `index.md` and `log.md`) uses this frontmatter:

```yaml
---
title: "Page Title"
type: entity | concept | source | output | context
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
- End every `entity`, `concept`, `source`, and `output` page with a `## Sources` section listing the raw sources it draws from. Context pages are exempt — their source is the live codebase.
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
2. If the question is about code, a bug, or a specific file — also read `.claude/wiki/context/graph.md` and any relevant `context/<module>.md` deep-dive pages before answering.
3. Read the identified pages and synthesize an answer.
4. Cite wiki pages inline with `[[Page Title]]`.
5. If the answer is valuable enough to keep (complex synthesis, comparison, analysis) → **file it** in `.claude/wiki/output/<slug>.md`.
6. Append to `.claude/wiki/log.md`:
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

### 3. Map the Codebase (`/map`)

When the user says "map" or "/map":

1. Glob all source files in the project (excluding `.claude/`, `node_modules/`, build artifacts).
2. For each file: read it, extract its purpose, key exports/functions/classes, and notable dependencies.
3. Write `.claude/wiki/context/graph.md` — the master context map (see template below).
4. For any module that is large, complex, or central to the system: create `.claude/wiki/context/<module>.md` with a deep-dive (data flow, entry points, gotchas).
5. Update `.claude/wiki/index.md` — add or refresh the Context Map section.
6. Append to `.claude/wiki/log.md`:
   ```
   ## [YYYY-MM-DD] map | Codebase context map
   Files indexed: N
   Deep pages created: [list or "none"]
   ```

**Context map template** (`.claude/wiki/context/graph.md`):
```markdown
---
title: "Codebase Context Map"
type: context
tags: [codebase, index]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Codebase Context Map

> Consult this before touching any file. Update entries for files you modify.

## File Index

| File | Purpose | Key Exports / Entry Points | Dependencies |
|------|---------|---------------------------|--------------|
| `src/index.ts` | App entry point | `main()` | config, server |
| `src/server.ts` | HTTP server setup | `createServer()` | express, routes |
| ... | ... | ... | ... |

## Module Deep-Dives
- [[context/server]] — request lifecycle, middleware order
- [[context/auth]] — token flow, session handling

## Architecture Notes
<high-level data flow or system diagram in text>
```

---

### 4. Lint the Wiki

When the user says "lint" or "health check":

1. Read all pages in `.claude/wiki/`.
2. Check for and report:
   - **Contradictions**: claims that conflict across pages
   - **Orphan pages**: pages with no inbound `[[links]]`
   - **Stale claims**: pages referencing "recent" events that may be outdated
   - **Missing pages**: entities or concepts mentioned in `[[links]]` that have no page
   - **Gaps**: important topics not yet covered
   - **Data gaps**: questions that a web search could answer
   - **Stale context entries**: rows in `context/graph.md` whose files no longer exist in the codebase — remove them
   - **Unmapped files**: source files in the project not listed in `context/graph.md` — flag them for a `/map` refresh
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

Update this file on every `ingest`, `/map`, and any operation that creates or removes wiki pages. Keep it sorted by category:

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

## Context Map
| Page | Coverage | Last updated |
|------|----------|-------------|
| [[context/graph]] | Master file index | YYYY-MM-DD |
| [[context/module]] | Module deep-dive | YYYY-MM-DD |
```

---

## Log Format (`.claude/wiki/log.md`)

Append only. Never edit past entries. Format each entry as:

```
## [YYYY-MM-DD] <operation> | <title>
<1-3 lines of detail>
```

Operations: `ingest`, `query`, `lint`, `map`, `update`, `note`

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

## Context Map — Freshness Rule (MANDATORY)

After **any** operation that modifies codebase files:

1. Identify every file you created, edited, or deleted.
2. Update only those entries in `.claude/wiki/context/graph.md` — do not regenerate the whole map.
   - New file → add a row to the File Index.
   - Modified file → update its Purpose, Key Exports, and Dependencies columns.
   - Deleted file → remove its row and note the removal in the log.
3. If the change affects a module that has a deep-dive page (`context/<module>.md`), update that page too.
4. Append to `.claude/wiki/log.md`:
   ```
   ## [YYYY-MM-DD] update | context map
   Files updated: [list]
   ```

**Do not skip this step.** A stale context map is worse than no map — it sends future sessions in the wrong direction.

---

## What You Never Do

- Modify any file in `.claude/raw/`.
- Delete wiki pages without telling the user why.
- Leave a `[[link]]` pointing to a non-existent page without flagging it.
- Skip the log entry for any operation.
- Modify codebase files without updating `context/graph.md` afterward (see Freshness Rule).
- Run `/map` and skip updating `index.md`.
