# Watch alerts

A **watch alert** continuously monitors your sources (RSS, scrapers,
cloud drives) and emails you whenever new content mentions a keyword
you've defined.

## Create an alert

`/user/watch-alerts`:

1. **Keyword or expression**: the text to watch (e.g. `NIS2 directive`,
   `cybersecurity workforce act`).
2. **Scope**: personal / org / public, or a combination.
3. **Delivery cadence**: immediate / daily / weekly.
4. **Sensitivity** (`high` / `medium` / `low`): minimum relevance
   threshold for a match to trigger an email.

Keywords are **encrypted at rest** (`watch_alert.keyword_enc`).

## Mechanism

The `check_watch_alerts` cron (08:00 + 20:00 UTC, user timezone)
walks new contents since the previous run, computes a similarity
score against active alerts, and triggers a digest email if the
threshold is reached.

```
New articles / docs since last run
  ↓
bge-m3 embedding of content vs keyword embedding
  ↓
Relevance score (cosine similarity)
  ↓
If score ≥ threshold: add to user digest
  ↓
Bulk send at end of cron (1 email per user, max 50 hits)
```

## Management

- **Pause / resume**: temporarily disable without losing the config.
- **Modify** the keyword or threshold — optional backlog recompute
  (max once per 24 h).
- **Delete**: immediate purge of the alert and its hit history.

## Limits

- Up to **20 active alerts** per user (configurable via
  `WATCH_ALERTS_PER_USER`).
- Detection lag: 8 h max (cron runs twice a day). No real-time
  notifications — this is monitoring for awareness, not paging.
- In pure sovereign: emails are **log-only** by default (mailer
  disabled). Configure an internal MTA to actually receive them
  (see `docs/EMAIL.md` in the application repo).

## For whom?

- Competitive intelligence: watch a competitor's name across your
  article feed.
- Regulatory watch: get alerted whenever a new directive or text
  mentions a sensitive topic.
- Brand monitoring: detect mentions of your organisation.
- Academic research: new papers on a precise theme (combined with
  the Zotero connector).
