---
title: "Decision: Pre-synthesized wiki pages over RAG"
type: decision
tags: [decision, adr, rag, llm, architecture]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Decision: Pre-synthesized wiki pages over RAG

**Date:** 2026-04-17 (reconstructed from CLAUDE.md and camwiki project page)
**Status:** `accepted`

## Context

An LLM-maintained knowledge base can be architected two ways: generate answers at query time from raw documents (RAG), or pre-synthesize raw documents into structured pages that are maintained over time and queried directly.

## Options Considered

1. **RAG (Retrieval-Augmented Generation)** — store raw documents, embed them, retrieve relevant chunks at query time, and generate an answer. Standard pattern for "chat with your docs." But: expensive per query, inconsistent (same question may produce different answers), cross-document synthesis is shallow, and the system doesn't compound — every query starts from scratch.
2. **Pre-synthesized wiki** — Claude reads sources once, writes structured pages with cross-references, and maintains them as sources evolve. Query reads existing pages rather than re-deriving from raw.

## Decision

Pre-synthesized wiki pages maintained by Claude. Key reasons:

- **Compounding value:** Every ingest makes the wiki richer. Every query may produce a new page. The synthesis compounds — the wiki gets smarter over time.
- **Consistency:** The same query always finds the same pre-written answer. No hallucination variance between queries.
- **Cross-referencing as value:** Pre-synthesized pages can aggressively link to each other. The graph of links is itself a knowledge artifact — not possible with RAG chunks.
- **Query speed:** Reading a wiki page is instant; RAG re-derives from scratch on every query.
- **Scalability:** Designed to scale to org-level systems covering multiple teams and codebases — a curated knowledge graph is more trustworthy than on-demand LLM re-derivation at that scale.

## Consequences

- Sources must be actively re-ingested when they change — the wiki can go stale if not maintained. RAG always reads the latest raw documents.
- Initial ingest cost is higher — Claude writes many pages per source. RAG indexes incrementally.
- The wiki is only as good as its maintenance discipline — requires the ingest and lint workflows to be run regularly.

## Related Pages

- [[wiki/projects/camwiki]]
- [[wiki/decisions/camwiki/claude-md-operating-contract]]

## Sources

- `CLAUDE.md` — Section 1 (What This Wiki Is)
- `wiki/projects/camwiki.md` — Architecture section
