# Library

Myeline has **no manual file-upload feature**. A user's library
(and an organisation's library) is fed exclusively by two automatic
channels:

- **Cloud connectors** — Google Drive, OneDrive, Dropbox, kDrive,
  Zotero, S3 / S3-compatible, WebDAV
- **RSS / web scrapers**

This is a design choice: keep the source of truth on drives the
customer already controls, with no content duplication inside
Myeline.

## Scopes

`/user/ma-bibliotheque` aggregates your personal documents.
`/org/<slug>/workspace` aggregates those shared inside an
organisation. Each scope has its own ChromaDB collection:

| Scope             | ChromaDB collection     | Visibility           |
|-------------------|-------------------------|----------------------|
| Personal          | `user_<id>_personal`    | You only             |
| Organisation      | `org_<id>_shared`       | Org members          |
| Public platform   | `shared`                | Every user           |

## Cloud connectors

Connect your drive from `/user/cloud` (or `/org/<slug>/cloud` on
the organisation side):

- Google Drive, OneDrive, Dropbox, kDrive (sovereign-hybrid only,
  see [Cloud connectors](../admin/cloud-connectors.md))
- S3 / S3-compatible (all editions)
- WebDAV — Nextcloud, ownCloud (all editions)
- Zotero (sovereign-hybrid only, indexes bibliographic metadata)

The `check_cloud_sync` cron (every 4 h, tier-dependent floor)
detects new files and adds them to your collection. You can also
trigger a manual sync (max once per hour).

### Indexed formats

| Format              | Extensions             | Notes                                |
|---------------------|------------------------|--------------------------------------|
| **PDF**             | `.pdf`                 | text extraction via `pdfplumber`     |
| **Word**            | `.docx`                | via `python-docx`                    |
| **OpenDocument**    | `.odt`                 | via `odfpy`                          |
| **Plain text**      | `.txt`, `.md`          | UTF-8 expected                       |
| **HTML**            | `.html`, `.htm`        | sanitised via `bleach`               |

Files in other formats (XLSX, PPTX, ZIP, video, image…) are silently
ignored by the indexer. Size limit: 50 MB per file (configurable
via `MAX_UPLOAD_SIZE_MB`). OCR on scanned PDFs is disabled by
default (enable via `RAG_OCR_ENABLED=true` in sovereign-hybrid only,
Tesseract dependency).

## RSS / web scrapers

`/user/scrapers` (Pro+) lets you watch RSS feeds or web pages — each
fetched article is added to your personal library. An organisation
member can "push" a scraper to the shared library (🏢 button on the
scrapers page).

The `check_user_scrapers` cron runs 3 times per day at off-peak
hours (03:00 / 07:00 / 23:00 UTC) to avoid saturating the server
during business hours.

## Indexing pipeline

```
New file detected (cloud sync) or article fetched (scraper)
  → MIME detection (magic bytes)
  → Text extraction
  → Chunking (~500 tokens, overlap 50)
  → bge-m3 embedding (local Ollama)
  → ChromaDB insert into the target collection
  → Document RAG-queryable
```

Indexing is **async** (RQ worker). On a 100-page PDF, count 30-90
seconds after sync.

## Removal and re-indexing

- **Unindex a document** from the library: removes ChromaDB chunks.
  The file stays in the source drive.
- **Re-index a full collection**: `/admin/library/reindex` on the
  global admin side (costly, useful after embedding model upgrade).

## Single-document chat

Clicking a document (personal or org) opens a chat **scoped to that
single document** — useful to drill into a long PDF without
polluting other sources. See
[RAG search § single-document](rag-search.md#single-document).

## Why no manual upload?

Three reasons:

1. **Single source of truth** — when a document evolves on the
   customer side (new PDF in Drive), Myeline picks it up
   automatically at the next sync. No drift between the uploaded
   version and the "official" version.
2. **Compliance** — the document stays in the customer's
   classification system (Drive, SharePoint, Nextcloud…) which
   already handles permissions, backups, retention.
3. **Simplicity** — a single ingestion path to maintain and
   monitor (cloud sync), rather than doubling up with an upload
   form that has its own bugs and limits.

If your use case still requires a manual upload (e.g. one-off
documents unrelated to a drive), reach out to us — technically
doable, to arbitrate per context.
