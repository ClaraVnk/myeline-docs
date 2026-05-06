# Monitoring

Myeline exposes several monitoring endpoints to plug into your obs
stack (Prometheus / Grafana, Uptime Kuma, Zabbix, Datadog, Sentry…).

## Endpoints

### `/healthz` — liveness

Always 200 if the Flask process responds. Doesn't touch any external
dependency. **Ideal for Kubernetes / Podman health probes** or a
load balancer.

```bash
$ curl -i https://your-domain/healthz
HTTP/1.1 200 OK
Content-Type: application/json

{"status": "ok", "version": "v2026.05.01"}
```

### `/health` — readiness

Checks the state of critical dependencies:

```json
{
  "status": "ok",
  "checks": {
    "db": "ok",
    "redis": "ok",
    "ollama": "ok",
    "chromadb": "ok",
    "mistral": "ok"      // sovereign-hybrid only
  }
}
```

Returns **503** if a critical dependency (DB or Redis) is down,
**200 with `degraded`** if a non-critical dependency (Ollama or AI
provider) is down. Use this for readiness probes.

### `/status` — public page

`https://your-domain/status` shows:

- Current state of each dependency (green / orange / red)
- **30-day uptime** as bars (1 per 1-hour slot)
- Incident history > 5 minutes

Fed by the `record_status` cron (every 5 min). Handy to share with
your internal users.

### `/metrics` — Prometheus

Standard Prometheus text format, scrapable every 15-60 s:

```
# HTTP
myeline_http_requests_total{method,route,status}
myeline_http_request_duration_seconds{method,route}

# RAG
myeline_rag_queries_total{user_id,status,scope}
myeline_rag_latency_seconds{stage}      # embed / retrieve / rerank / synth

# Resources
myeline_users_total{tier,is_admin}
myeline_orgs_total{plan}
myeline_conversations_active

# Quotas / audit
myeline_quota_usage{user_id,metric}
myeline_audit_events_total{action}

# Enterprise provisioning
myeline_enterprise_stacks{status}
```

No auth on `/metrics` by default — restrict via the reverse proxy
(IP allow-list of your Prometheus scraper).

## Logs

- **Application logs**: stdout of the `web` container (captured by
  podman / journald). Structured JSON format (slog-like).
- **Mailer logs** (in pure sovereign): `data/logs/mailer/<date>.log`
  — every simulated send is traced in clear text to allow manual
  recovery of links (confirmation, reset, invitation).
- **Cron logs**: `data/logs/cron/<task>.log`.

Plug Loki / Vector / journald → Elastic according to your standard.
No PII is logged outside `data/logs/mailer/` (and that folder must
be treated as sensitive — see [GDPR](../compliance/gdpr.md)).

## Recommended alerts

| Alert                                        | Severity  | Threshold                 |
|----------------------------------------------|-----------|---------------------------|
| `/healthz` not 200 for 1 min                 | critical  | likely downtime           |
| `/health` `degraded` > 10 min                | warning   | external dependency down  |
| `db` or `redis` down                         | critical  | direct user impact        |
| HTTP 5xx > 1 % over 5 min                    | warning   | likely application bug    |
| RAG p95 > 30 s over 5 min                    | warning   | LLM slow / saturated      |
| Quota usage > 90 %                           | info      | scaling to anticipate     |
| Free disk < 20 %                             | warning   | backup / chroma to purge  |

Grafana templates available on request (`hello@myeline.io`).
