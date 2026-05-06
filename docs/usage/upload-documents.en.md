# Library and uploads

Myeline distinguishes two indexing scopes: a user's **personal
library** and an organisation's **shared library**. They are fed
in different ways.

## Personal library

`/user/ma-bibliotheque` aggregates your personal documents. **There
is no manual upload**: the library is filled automatically through
two channels.

### Via cloud connectors

Connect your drive from `/user/cloud`:

- Google Drive, OneDrive, Dropbox, kDrive (sovereign-hybrid only,
  see [Cloud connectors](../admin/cloud-connectors.md))
- S3 / S3-compatible (all editions)
- WebDAV (Nextcloud, ownCloud — all editions)
- Zotero (sovereign-hybrid only, indexes bibliographic metadata)

The `check_cloud_sync` cron (every 4 h, tier-dependent floor) detects
new files and adds them to your ChromaDB collection. You can also
trigger a manual sync (1/h).

### Via RSS / web scrapers

`/user/scrapers` (Pro+) lets you watch RSS feeds or web pages — each
fetched article is added to your library.

## Organisation library

In an **organisation** context, the admin and members can **upload
files directly** from `/org/<slug>/workspace`:

| Format              | Extensions             | Notes                                |
|---------------------|------------------------|--------------------------------------|
| **PDF**             | `.pdf`                 | text extraction via `pdfplumber`     |
| **Word**            | `.docx`                | via `python-docx`                    |
| **OpenDocument**    | `.odt`                 | via `odfpy`                          |
| **Plain text**      | `.txt`, `.md`          | UTF-8 expected                       |
| **HTML**            | `.html`, `.htm`        | sanitised via `bleach`               |

### Limits

- **Max file size**: 50 MB (configurable via `MAX_UPLOAD_SIZE_MB`)
- **Per-organisation quota**: per plan (see
  [Quotas](../admin/quotas.md))
- **Rate limit**: 20 uploads / hour / user

### Indexing pipeline

```
Upload (POST /org/<slug>/docs/upload)
  → MIME check (magic bytes)
  → Text extraction
  → Chunking (~500 tokens, overlap 50)
  → bge-m3 embedding (local Ollama)
  → ChromaDB insert (org_<id>_shared collection)
  → Document visible in search
```

Indexing is **async** (RQ worker). On a 100-page PDF, count 30-90
seconds.

### Delete and re-index

- **Delete** a document: removes ChromaDB chunks + the source file.
  Admin-only action.
- **Re-index** a full collection: `/admin/library/reindex` global
  admin only, costly operation (useful after embedding model
  upgrade).

## Single-document chat

Clicking a document (personal or org) opens a chat **scoped to that
single document** — useful to drill into a long PDF without polluting
other sources. See
[RAG search § single-document](rag-search.md#single-document).
