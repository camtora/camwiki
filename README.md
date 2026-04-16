# camwiki

A personal engineering knowledge base maintained by Claude. Covers home-server infrastructure, self-hosted services, project architecture, and technical decisions. Designed to scale to an org-level system spanning multiple teams, business entities, and codebases.

---

## Concept

Traditional documentation gets stale. RAG re-derives answers from raw sources at query time — expensive and inconsistent. This wiki takes a third approach: **persistent compiled synthesis**.

Claude reads raw sources (git repos, markdown docs, config files), synthesizes them into structured wiki pages, and maintains those pages as the authoritative knowledge layer. Every ingest makes the wiki richer. Every query can produce a new page. The synthesis is already there — no re-derivation at query time.

The human curates and directs. Claude does all writing, cross-referencing, and bookkeeping.

---

## How It Was Built

1. Designed an operating contract (`CLAUDE.md`) that Claude reads at the start of every session — defines the directory layout, naming conventions, frontmatter schema, page templates, and three core workflows (ingest, query, lint)
2. Scaffolded the directory structure and symlinked 11 live git repos under `raw/repos/` so Claude can shell out to `git log`, `git diff`, and read source files directly
3. Ingested repos one by one (or in batches): Claude reads the README and recent commit history, synthesizes project and infrastructure pages, cross-references them, updates the index, and appends to the log
4. Ran a retroactive decision pass on the infrastructure repo to capture ADRs from commit history

The wiki now covers ~30 pages across 11 ingested repos, with full cross-referencing and an append-only operation log.

---

## Directory Structure

```
camwiki/
├── CLAUDE.md          ← operating contract; Claude reads this every session
├── index.md           ← master catalog; updated on every ingest
├── log.md             ← append-only record of all operations
│
├── raw/               ← IMMUTABLE source documents (Claude reads, never writes)
│   ├── articles/      ← clipped web articles
│   ├── docs/          ← READMEs, vendor docs, official references
│   ├── code/          ← code files, scripts, config dumps
│   ├── repos/         ← symlinks to live git repos
│   └── assets/        ← images
│
└── wiki/              ← LLM-maintained pages
    ├── projects/      ← one page per project
    ├── infrastructure/← hosts, networks, services, storage
    ├── integrations/  ← external APIs and data flows
    ├── decisions/     ← architecture decision records (ADRs)
    ├── business/      ← contracts, clients, metrics, vendors
    ├── concepts/      ← cross-cutting technical reference
    └── sources/       ← one summary page per raw source ingested
```

---

## Linked Repos

Live git repos are symlinked under `raw/repos/`. Claude reads them directly — never writes to them.

| Symlink | Project |
|---------|---------|
| `raw/repos/clarity` | Android fundraising app (GlobalFaces) |
| `raw/repos/camerontora.ca` | Personal website |
| `raw/repos/docker-services` | Home server Docker stack |
| `raw/repos/watchmap` | Plex stream map + tvOS app |
| `raw/repos/donormap` | Donor visualization map |
| `raw/repos/deployment` | Clarity FastAPI backend |
| `raw/repos/haymaker` | Health/habit/finance tracker |
| `raw/repos/infrastructure` | Infrastructure configuration |
| `raw/repos/rotosync` | Standup facilitation app |
| `raw/repos/sbca` | Cottage association mobile app |
| `raw/repos/whosup` | Social activity app |

---

## Browsing

Open this repo as an **Obsidian vault**. Recommended settings:

- New link format: Absolute path in vault
- Use `[[Wikilinks]]`: On
- Attachment folder: `raw/assets`
- Community plugins: **Dataview**, **Obsidian Git** (auto-pull on startup)

The graph view shows the cross-reference structure across all wiki pages.

---

## Adding Knowledge

Start a Claude Code session in this directory. Claude reads `CLAUDE.md`, `index.md`, and the last 5 log entries automatically.

**To ingest a new source:**
- Drop a file into `raw/docs/` or `raw/articles/` and tell Claude to ingest it
- For a linked repo, just name it: "ingest the infrastructure repo"

Claude will: read the source → summarize what it found → write a source summary page → create or update topic pages → update `index.md` → append to `log.md` → report what changed.

**To query the wiki:**
- Ask Claude any question about the documented systems
- Claude reads the index, follows cross-references, and synthesizes an answer grounded in wiki content
- Non-trivial answers can be filed as new pages

**To lint:**
- Ask Claude to run a health check
- Checks for orphan pages, missing cross-references, contradictions, stale claims, index drift

---

## Page Schema

Every wiki page has YAML frontmatter:

```yaml
---
title: "Human-readable title"
type: project | infrastructure | integration | decision | business | concept | source
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
source_count: 1
---
```

`source_count` tracks how many raw sources contributed to a page — useful for Dataview queries to find well-sourced vs thin pages.

---

## Sync

This repo is pushed to GitHub (`git@github.com:camtora/camwiki.git`). Obsidian Git pulls automatically on vault open, keeping the Mac in sync with any changes made via Claude Code on the server.
