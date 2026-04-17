---
title: "Decision: Google Gemini (Vertex AI) over Anthropic Claude for AI extraction"
type: decision
tags: [decision, adr, gemini, vertex-ai, ai, llm]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Google Gemini (Vertex AI) over Anthropic Claude for AI extraction

**Date:** 2026-04-17 (reconstructed from commit 95a6658)
**Status:** `accepted`

## Context

Rotosync uses an LLM to extract structured data (workstreams, action items, blockers, key decisions) from meeting transcripts. The original implementation used Anthropic Claude via API key. The question was whether to migrate to Google Gemini via Vertex AI.

## Options Considered

1. **Anthropic Claude API** — original implementation (`claudeExtractor.ts`). Requires `ANTHROPIC_API_KEY` secret stored in Firebase secrets (`firebase functions:secrets:set`).
2. **Google Gemini via Vertex AI** — uses Application Default Credentials (ADC) from the same GCP project as Cloud Functions. No additional secret required.

## Decision

Migrate to Gemini 2.0 Flash via Vertex AI (`geminiExtractor.ts`). Rename file, keep identical TypeScript extraction interface and prompt structure.

Key reasons:

- **Zero-config secrets:** Vertex AI uses Application Default Credentials — the same GCP service account that runs Cloud Functions. No API key to rotate, store, or manage.
- **Same GCP ecosystem:** Vertex AI is in the same GCP project as Firestore, Cloud Functions, and Cloud Scheduler — unified billing, monitoring, and quotas.
- **Cost efficiency:** Gemini 2.0 Flash is faster and more cost-effective for token-heavy transcript summarisation.
- **Prompt portability:** Extraction logic is prompt-based, not model-specific. Same prompts produce equivalent output across models.

## Consequences

- Model behaviour differences between Claude and Gemini may affect extraction quality in edge cases.
- Vertex AI requires the Cloud Functions service account to have the `Vertex AI User` role — one-time IAM setup.
- `ANTHROPIC_API_KEY` secret removed from Firebase project.

## Related Pages

- [[wiki/projects/rotosync]]
- [[wiki/decisions/rotosync/firebase-unified-backend]]

## Sources

- `raw/repos/rotosync` — functions/src/geminiExtractor.ts, functions/package.json, commit 95a6658
