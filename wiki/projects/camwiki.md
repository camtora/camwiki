---
title: "camwiki"
type: project
tags: [project, wiki, llm, knowledge-base, obsidian, claude]
created: 2026-04-16
updated: 2026-04-17
source_count: 1
---

# camwiki

A personal engineering knowledge base maintained by Claude. Sources are git repos, markdown docs, and config files. Claude synthesizes them into structured wiki pages and maintains them over time. Browsed in Obsidian. Designed to scale to an org-level system covering multiple teams, business entities, and codebases.

## Status

Current status: `active`
Last meaningful change: 2026-04-17 (known issues pass, desktop doc ingests, history pages)

## Architecture

Three layers:

```
raw/           ← IMMUTABLE source documents (Claude reads, never writes)
wiki/          ← LLM-maintained pages (Claude owns entirely)
CLAUDE.md      ← operating contract (auto-read by Claude Code each session)
```

**Why not RAG?** RAG re-derives answers from raw sources at query time — expensive and inconsistent. This wiki pre-synthesizes: Claude reads sources once, writes structured pages, and maintains them. The synthesis is already there at query time. Every ingest compounds the value.

**Why CLAUDE.md?** Claude Code auto-reads `CLAUDE.md` at session start. This makes the operating contract self-activating — no manual setup needed each session. The file defines directory layout, naming conventions, frontmatter schema, 7 page templates, and 3 workflows.

## Infrastructure

- Git repo: `git@github.com:camtora/camwiki.git`
- Hosted on the home server at `/home/camerontora/camwiki`
- Obsidian installed on the server; vault synced to Mac and iPhone via **Obsidian Sync**
- Mac previously used Obsidian Git plugin — decommissioned in favour of Obsidian Sync
- Claude Code sessions run on the server via SSH

## Linked Repos

11 live git repos symlinked under `raw/repos/` — Claude reads via `git log`, `git diff`, and direct file reads. Never writes to them.

| Symlink | Project |
|---------|---------|
| `raw/repos/clarity` | Android fundraising app |
| `raw/repos/camerontora.ca` | Personal website |
| `raw/repos/docker-services` | Home server Docker stack |
| `raw/repos/watchmap` | Plex stream map + tvOS |
| `raw/repos/donormap` | Donor visualization map |
| `raw/repos/deployment` | Clarity FastAPI backend |
| `raw/repos/haymaker` | Health/habit/finance tracker |
| `raw/repos/infrastructure` | Infrastructure configuration |
| `raw/repos/rotosync` | Standup facilitation app |
| `raw/repos/sbca` | Cottage association app |
| `raw/repos/whosup` | Social activity app |

## Workflows

**Ingest** — read source → discuss scope → write source summary page → create/update topic pages → update index → append log → report.

**Query** — read index → read candidate pages → synthesize answer grounded in wiki content → optionally file answer as new page.

**Lint** — check orphans, missing cross-references, contradictions, stale claims, stub sections, missing pages, index drift.

## Page Structure

Every wiki page has YAML frontmatter (`title`, `type`, `tags`, `created`, `updated`, `source_count`). Seven page types: project, infrastructure, integration, decision, business, concept, source. All internal links use full paths (`[[wiki/projects/camwiki]]`) to prevent collisions.

`source_count` tracks how many raw sources contributed to a page — queryable via Obsidian Dataview.

## Wiki Directories

| Directory | Purpose |
|-----------|---------|
| `wiki/projects/` | One page per project |
| `wiki/infrastructure/` | Hosts, networks, services, storage |
| `wiki/integrations/` | External APIs and data flows |
| `wiki/decisions/` | ADRs, one subdir per project |
| `wiki/concepts/` | Cross-cutting technology reference (10 pages) |
| `wiki/sources/` | One summary page per raw source ingested |
| `wiki/history/` | Auto-generated git log per repo — 11 pages |

## Coverage (as of 2026-04-17)

| Category | Count |
|----------|-------|
| Project pages | 13 |
| Infrastructure pages | 11 |
| Decision / ADR pages | 35+ |
| Concept pages | 10 |
| Source pages | 14 |
| History pages | 11 |
| Known Issues sections | 9 projects |

## Key Decisions

- [[wiki/decisions/camwiki/pre-synthesis-over-rag|Pre-synthesis over RAG]] — Pages written once, maintained over time; compounding value; consistent answers.
- [[wiki/decisions/camwiki/claude-md-operating-contract|CLAUDE.md as Operating Contract]] — Auto-read by Claude Code; all rules in one self-activating file.
- [[wiki/decisions/camwiki/raw-immutable|raw/ is Immutable]] — Source documents never modified; enables auditability and clean re-ingest.
- [[wiki/decisions/camwiki/full-path-wikilinks|Full-path Wiki-links]] — Collision-safe and refactoring-safe as wiki scales to org-level.
- [[wiki/decisions/camwiki/append-only-log|Append-only log.md]] — Immutable audit trail of every operation; grep-searchable.

## Change Log

- 2026-04-17: Known Issues sections added to 9 project pages (scrape of all repos); 3 desktop docs ingested (drive layout, ops reference, SSO guide); 11 git history pages created under wiki/history/; CLAUDE.md updated with Step 4c (history regeneration) and Step 4b (ADR creation); 10 concept pages written; ADR deep dives for rotosync, donormap, watchmap, camwiki (23 ADRs)
- 2026-04-17: Obsidian Sync set up — server → Mac + iPhone; replaced Obsidian Git plugin on Mac; `raw/` excluded from sync to prevent node_modules storm across all linked repos
- 2026-04-16: Initial scaffold, 11 repo ingests, infrastructure ADR retro pass, docs/ folder ingest (13 docs)

## Open Questions

- What's the right cadence for re-ingesting repos as they evolve?
- Should business/ be populated with client/contract info, or kept separate?

## Sources

- [[wiki/sources/infrastructure-repo]] — the infrastructure repo was the first and most deeply ingested source; its docs/ folder drove the majority of infrastructure page content
