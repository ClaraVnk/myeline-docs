# Quotas and plans

In on-prem editions (sovereign and sovereign-hybrid), the **licence
covers the operator** — there is no notion of an internal user plan.
All features are enabled: multi-turn, conversations, alerts,
history, single-document chat.

Quotas still exist, but as **technical guard-rails** (preventing one
user from monopolising the server), not as a commercial lever.

## Default quotas

| Metric                     | Limit              | Scope            |
|----------------------------|--------------------|------------------|
| RAG queries                | 300 / day          | per user         |
| RSS / scraper sources      | 50 / org           | per organisation |
| Personal documents         | 10 GB / user       | per user         |
| Active cloud connections   | 10 / user          | per user         |
| Conversation history       | unlimited          | —                |

Values are defined in `app/services/quotas.py` and can be overridden
via environment variables (see
[Server prerequisites](../install/prerequisites.md)).

## Monitoring

The `check_quotas` cron runs daily at 04:00 UTC:

- **80 %** usage reached → warning email to the org owner
- **100 %** reached → soft block (HTTP 429 until the next day)
- In pure sovereign, the mailer is log-only — check `logs/mailer/`
  or wire an internal MTA.

## Admin override

A global admin can **bump a quota on the fly** from `/admin/users/<id>`
(field "override quota", expires after 24 h). Action recorded in the
audit log.

## Exposed metrics

`/metrics` (Prometheus) exposes:

- `myeline_quota_usage{user_id, metric}` (gauge)
- `myeline_rag_queries_total{user_id, status}` (counter)
- `myeline_quota_blocks_total` (counter)

Plug Grafana or your obs stack for dashboards.
