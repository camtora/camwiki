# CLAUDE.md — camwiki Operating Contract

This file governs how you operate this wiki. Read it in full at the start of every
session before taking any action. Do not skip sections. If this file and a user
instruction conflict, ask for clarification before proceeding.

---

## 1. What This Wiki Is

camwiki is a personal home-server engineering knowledge base maintained exclusively
by Claude. The human provides sources and direction. Claude does all writing,
cross-referencing, and bookkeeping.

The wiki is a persistent, compounding artifact. Every ingest makes it richer.
Every query may produce a new page. Every lint pass makes it more consistent.
Nothing is re-derived from scratch at query time — the synthesis is already there.

The wiki is designed to scale from a personal home-server knowledge base to an
org-level system covering multiple teams, business entities, and codebases.

---

## 2. Directory Layout

```
camwiki/
├── CLAUDE.md          ← this file; read first every session
├── index.md           ← master catalog; update on every ingest or new page
├── log.md             ← append-only record; append on every operation
│
├── raw/               ← IMMUTABLE. Source documents. Never modify anything here.
│   ├── articles/      ← clipped web articles (Obsidian Web Clipper output)
│   ├── docs/          ← READMEs, vendor docs, official references
│   ├── code/          ← code files, scripts, config dumps
│   ├── repos/         ← symlinks to live git repos (read-only access)
│   └── assets/        ← images downloaded alongside articles
│
└── wiki/              ← LLM-maintained pages. You own this layer entirely.
    ├── projects/      ← one page per home-server project
    ├── infrastructure/← hosts, networks, services, storage topology
    ├── integrations/  ← external APIs, data flows, service connections
    ├── decisions/     ← ADRs and significant choices
    ├── business/      ← contracts, clients, metrics, entities
    ├── concepts/      ← technology reference knowledge
    └── sources/       ← one summary page per raw source ingested
```

**Hard rules:**
- Never create, modify, or delete anything under `raw/`.
- Never nest wiki pages more than one level deep inside a category directory.
  **Exception:** `wiki/decisions/` may contain one level of subdirectories named
  after the project or domain (e.g., `infra/`, `haymaker/`). ADR files live
  directly inside those subdirs — no further nesting.
- If a page genuinely spans multiple categories, place it in `concepts/` and
  cross-reference it from the relevant category pages.

---

## 3. Linked Git Repos

The following live git repos are symlinked under `raw/repos/`. You can read their
files and shell out to git commands directly — never write to them.

| Symlink | Project |
|---------|---------|
| `raw/repos/clarity` | Android app (AndroidStudioProjects/clarity) |
| `raw/repos/camerontora.ca` | Personal website |
| `raw/repos/docker-services` | Home server Docker services |
| `raw/repos/watchmap` | Watchmap (nested repo inside docker-services) |
| `raw/repos/donormap` | Donormap project |
| `raw/repos/deployment` | globalfaces-app backend deployment |
| `raw/repos/haymaker` | Haymaker project |
| `raw/repos/infrastructure` | Infrastructure configuration |
| `raw/repos/rotosync` | Rotosync project |
| `raw/repos/sbca` | SBCA project |
| `raw/repos/whosup` | Whosup project |

**Commands for reading repo state:**
```bash
git -C raw/repos/<name> log --oneline -20          # recent commits
git -C raw/repos/<name> log --since="2 weeks ago" --format="%h %s"  # recent activity
git -C raw/repos/<name> diff HEAD~1                 # last change
git -C raw/repos/<name> show HEAD                   # latest commit full diff
```

When ingesting a repo for the first time, read its README, scan recent commit
history, and explore the top-level directory structure to inform the project page.

---

## 4. Naming Conventions

**Filenames:** kebab-case, lowercase, no spaces, `.md` extension.
- Good: `nginx-reverse-proxy.md`, `home-nas-setup.md`, `fastapi-over-nextjs.md`
- Bad: `Nginx Reverse Proxy.md`, `homeNasSetup.md`
- ADR files in `wiki/decisions/<subdir>/` do **not** need a `decision-` prefix —
  the subdir provides context. Exception: the existing infra ADRs keep their
  `decision-` prefix since they predate this convention.

**Wiki-internal links:** Use Obsidian wiki-link format with the full path from
the repo root, no extension.
- Format: `[[wiki/projects/home-media-server]]`
- Display alias when helpful: `[[wiki/projects/home-media-server|Media Server]]`
- Never use relative paths — they break across directory moves.

---

## 5. Frontmatter Fields

Every wiki page under `wiki/` must have a YAML frontmatter block.

```yaml
---
title: "Human-readable title"
type: project | infrastructure | integration | decision | business | concept | source
tags: [tag1, tag2, tag3]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 0
---
```

Field rules:
- `title`: Title case, matches the H1 heading on the first line of the body.
- `type`: Exactly one value from the enum above.
- `tags`: At minimum one tag matching the category (e.g. `project`, `infra`).
- `created`: Set once when the page is first created. Never change.
- `updated`: Update on every substantive edit.
- `source_count`: Number of raw sources that contributed to this page. Increment
  when adding content from a new source. Never decrement.

---

## 6. Page Templates

Use these templates when creating a new page. Write `_No entries yet._` in any
empty section — never leave a section header with nothing below it.

### 6.1 Project Page (`wiki/projects/`)

```markdown
---
title: "Project Name"
type: project
tags: [project, <tech-tag>, <status-tag>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 0
---

# Project Name

One-paragraph description of what this project is and what problem it solves.

## Status

Current status: `active | paused | complete | abandoned`
Last meaningful change: YYYY-MM-DD

## Architecture

How the project is structured — components, dependencies, data flows.

## Infrastructure

Which hosts, services, containers, or network segments this project uses.
Link to relevant pages: [[wiki/infrastructure/example-host]]

## Integrations

External services or APIs this project connects to.
Link to integration pages: [[wiki/integrations/example-service]]

## Key Decisions

Significant choices made during this project with brief rationale.
Link to ADR pages where they exist: [[wiki/decisions/decision-example]]

## Change Log

Reverse-chronological list of significant changes.
- YYYY-MM-DD: Description of change.

## Open Questions

Things not yet resolved. Delete entries when resolved.

## Sources

- [[wiki/sources/source-slug]] — one-line description
```

### 6.2 Infrastructure Page (`wiki/infrastructure/`)

```markdown
---
title: "Infrastructure Component Name"
type: infrastructure
tags: [infra, <component-type-tag>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 0
---

# Infrastructure Component Name

What this is: one sentence.

## Specification

| Property | Value |
|----------|-------|
| ...      | ...   |

## Role in the Stack

What this component does, what depends on it, what it depends on.

## Configuration Notes

Non-obvious configuration decisions and why they were made.

## Connected Projects

- [[wiki/projects/example-project]]

## Known Issues / Limitations

Current problems, constraints, or workarounds.

## Sources

- [[wiki/sources/source-slug]] — one-line description
```

### 6.3 Integration Page (`wiki/integrations/`)

```markdown
---
title: "Service or Integration Name"
type: integration
tags: [integration, <service-tag>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 0
---

# Service or Integration Name

What this service is and why it is used.

## Connection Details

How the home server connects to this service (protocol, auth method, direction).

## Data Flows

What data moves, in which direction, and at what frequency.

## Dependent Projects

- [[wiki/projects/example-project]]

## API / Auth Notes

Key API surface, authentication approach, rate limits or quotas.

## Sources

- [[wiki/sources/source-slug]] — one-line description
```

### 6.4 Decision / ADR Page (`wiki/decisions/`)

```markdown
---
title: "Decision: Short Title"
type: decision
tags: [decision, adr, <topic-tag>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 0
---

# Decision: Short Title

**Date:** YYYY-MM-DD
**Status:** `proposed | accepted | superseded | deprecated`
**Superseded by:** [[wiki/decisions/newer-decision]] _(if applicable)_

## Context

What situation prompted this decision. What constraints existed.

## Options Considered

1. **Option A** — brief description, pros and cons.
2. **Option B** — brief description, pros and cons.

## Decision

What was decided and why.

## Consequences

What changes as a result. What becomes easier. What becomes harder.

## Related Pages

- [[wiki/projects/related-project]]

## Sources

- [[wiki/sources/source-slug]] — one-line description
```

### 6.5 Business Entity Page (`wiki/business/`)

```markdown
---
title: "Entity Name"
type: business
tags: [business, <entity-type-tag>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 0
---

# Entity Name

What this entity is: client, contract, metric definition, vendor, etc.

## Definition

Precise definition or description.

## Key Details

| Field  | Value |
|--------|-------|
| ...    | ...   |

## Relationships

Other entities, projects, or integrations this relates to.

## Notes

_No entries yet._

## Sources

- [[wiki/sources/source-slug]] — one-line description
```

### 6.6 Concept Page (`wiki/concepts/`)

```markdown
---
title: "Concept Name"
type: concept
tags: [concept, <domain-tag>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 0
---

# Concept Name

Clear, concise definition.

## How It Is Used Here

How this concept applies specifically to this wiki's domain.

## Key Properties / Mechanics

_No entries yet._

## Relationships to Other Concepts

_No entries yet._

## External References

_No entries yet._
```

### 6.7 Source Summary Page (`wiki/sources/`)

```markdown
---
title: "Source: Document Title"
type: source
tags: [source, <topic-tag>]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 1
---

# Source: Document Title

**Raw file:** `raw/<subdir>/filename` _(or `raw/repos/<name>` for linked repos)_
**Ingested:** YYYY-MM-DD
**Type:** article | readme | doc | code | repo | config

## Summary

2-4 paragraph synthesis of the source.

## Key Facts / Data Points

_No entries yet._

## Relevance to Wiki

Which existing pages this source informed or updated.

## Contradictions or Tensions

Anything in this source that conflicts with existing wiki content.

## Pages Updated

- [[wiki/projects/example-project]] — what changed
```

---

## 7. Workflows

### 7.1 Ingest Workflow

Use this workflow every time a new source is added.

**Step 1 — Read.** Read the entire source. For a linked repo: read the README,
scan recent commits (`git log --oneline -20`), and browse the top-level structure.

**Step 2 — Discuss.** Briefly summarize to the user: what the source is, its 3-5
most important facts, and which wiki pages you expect it to affect. Ask if the
user wants to adjust scope or emphasis before writing anything.

**Step 3 — Write source summary page.** Create `wiki/sources/<slug>.md` using
template 6.7.

**Step 4 — Update or create topic pages.** For each wiki page affected:
- If the page exists: add new information, update `updated` date, increment
  `source_count`, add a link under Sources, note contradictions.
- If the page does not exist: create it from the appropriate template.
- A single source may touch 5-15 pages. Err toward more cross-referencing.

**Step 4b — Create ADR pages for significant decisions.** For every project
repo ingest, scan for architecture decisions and create ADR pages in
`wiki/decisions/<project-name>/` using template 6.4. Look for decisions in:
- `docs/` folder — architecture docs, stack comparisons, migration guides
- `README.md` — technology choices, constraints, trade-offs
- Commit history — commits explaining *why* (not just what) signal a real decision
- Config files — `package.json`, `build.gradle`, `docker-compose.yaml`,
  `requirements.txt` — technology choices are often only visible here

Aim for 3–6 ADRs per project covering: platform/language choice, key external
services, non-obvious architectural patterns, and significant constraints. Add a
`## Key Decisions` section to the project page linking to each ADR.

**Step 5 — Update index.md.** Add entries for new pages; update descriptions
for pages that changed significantly.

**Step 6 — Append to log.md.** One entry per the format in section 9.

**Step 7 — Report.** Summarize: source summary path, pages created, pages
updated, anything unresolved.

### 7.2 Query Workflow

**Step 1 — Read index.md.** Identify candidate pages.

**Step 2 — Read candidate pages.** Follow cross-reference links one level deep
if needed.

**Step 3 — Synthesize.** Compose an answer grounded in wiki content with
wiki-link citations. Note if the wiki lacks sufficient information.

**Step 4 — Offer to file.** If the answer involves non-trivial synthesis, offer
to create a new page from it. If the user agrees, create the page, update
index.md, and append to log.md with type `query`.

### 7.3 Lint Workflow

Run when the user asks for a health check, or proactively when the wiki has
grown significantly.

Check in order:
1. **Orphan pages** — no inbound links from any other wiki page.
2. **Missing cross-references** — topic mentioned by name without a link.
3. **Contradictions** — conflicting claims between pages.
4. **Stale claims** — dated statements superseded by newer sources.
5. **Stub sections** — sections still reading `_No entries yet._` on mature pages.
6. **Missing pages** — topics mentioned across multiple pages but lacking their own page.
7. **Index drift** — pages not in index.md, or index entries pointing to deleted pages.

Report findings, ask user which to fix, fix them, append lint entry to log.md.

---

## 8. Index Entry Format

Entries are one line per page, organized by category section, alphabetical order:

```
- [[wiki/category/page-slug|Page Title]] — one-line description. `N sources`
```

Example:
```
- [[wiki/projects/docker-services|Docker Services]] — Home server Docker stack with all self-hosted services. `3 sources`
```

---

## 9. Log Entry Format

Append-only. Never edit or delete past entries. Always append at the bottom.

```markdown
## [YYYY-MM-DD] type | title

- **Source:** `raw/subdir/filename` _(ingest only)_
- **Pages created:** list or "none"
- **Pages updated:** list or "none"
- **Notes:** anything notable or unresolved
```

`type` must be one of: `ingest`, `query`, `lint`, `edit`, `bootstrap`

Grep patterns:
- All ingests: `grep "^\#\# \[" log.md | grep "ingest"`
- Last 5 ops:  `grep "^\#\# \[" log.md | tail -5`

---

## 10. Session Startup Checklist

At the start of every Claude Code session in this repo:

1. Read this file (`CLAUDE.md`) in full.
2. Read `index.md` to orient to current wiki state.
3. Read the last 5 entries in `log.md` to understand recent activity.
4. Ask the user what they want to do today.

Do not skip step 1 even if you believe you remember the conventions.

---

## 11. General Principles

- **The human curates; Claude maintains.** Do not add opinions beyond what is
  defensible from the sources. Flag uncertainty explicitly.
- **Cite everything.** Every factual claim should be traceable to a source page
  or a decision page.
- **Cross-reference aggressively.** A fact relevant to another page should be
  linked. The graph is the value.
- **Never summarize a source without reading it completely.**
- **Prefer updating existing pages over creating new ones.**
- **When in doubt, ask.** Pausing to confirm is cheaper than undoing 20 edits.
- **Preserve the log's integrity.** Never edit past log entries.
