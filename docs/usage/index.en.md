# Usage

This section covers usage **on the end-user side** — features
exposed via the web interface.

- [Library and uploads](upload-documents.md) — how the personal
  library is fed (cloud + scrapers) vs direct upload at org level,
  supported formats, indexing.
- [RAG search](rag-search.md) — ask a question, refine, cite the
  sources, strict mode.
- [Multi-turn conversations](conversations.md) — persistent
  discussion thread, history, export.
- [Watch alerts](watch-alerts.md) — monitor a keyword across your
  sources and receive a digest.

## Search scope

Three **scopes**, selectable via the contextual menu:

| Scope          | Content                                                  |
|----------------|-----------------------------------------------------------|
| **Personal**   | your uploaded documents + your connected cloud drives     |
| **Organisation** | shared library across your org (members + admin)        |
| **Public**     | platform public sources (RSS, admin scrapers)             |

Users can combine scopes (tick several boxes) — RRF (Reciprocal
Rank Fusion) merges results across selected ChromaDB collections.

## Quotas

See [Quotas and plans](../admin/quotas.md). In on-prem editions, all
Pro+ features are enabled by default (multi-turn, conversations,
alerts, single-document chat).
