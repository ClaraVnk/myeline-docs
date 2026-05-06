# Supervision

Myeline expose plusieurs endpoints de supervision pour brancher votre
stack obs (Prometheus / Grafana, Uptime Kuma, Zabbix, Datadog,
Sentry…).

## Endpoints

### `/healthz` — liveness

Toujours 200 si le process Flask répond. Ne touche aucune dépendance
externe. **Idéal pour les health probes Kubernetes / Podman** ou un
load balancer.

```bash
$ curl -i https://votre-domaine/healthz
HTTP/1.1 200 OK
Content-Type: application/json

{"status": "ok", "version": "v2026.05.01"}
```

### `/health` — readiness

Vérifie l'état des dépendances critiques :

```json
{
  "status": "ok",
  "checks": {
    "db": "ok",
    "redis": "ok",
    "ollama": "ok",
    "chromadb": "ok",
    "mistral": "ok"      // souverain-hybride uniquement
  }
}
```

Renvoie **503** si une dépendance critique (DB ou Redis) est down,
**200 avec `degraded`** si une dépendance non critique (Ollama ou
provider IA) est down. À utiliser pour les readiness probes.

### `/status` — page publique

`https://votre-domaine/status` affiche :

- L'état actuel de chaque dépendance (vert / orange / rouge)
- L'**uptime sur 30 jours** sous forme de barres (1 par tranche
  d'1 heure)
- L'historique des incidents > 5 minutes

Alimentée par le cron `record_status` (toutes les 5 min). Pratique
à partager avec vos utilisateurs internes.

### `/metrics` — Prometheus

Format texte Prometheus standard, scrapable toutes les 15-60 s :

```
# HTTP
myeline_http_requests_total{method,route,status}
myeline_http_request_duration_seconds{method,route}

# RAG
myeline_rag_queries_total{user_id,status,scope}
myeline_rag_latency_seconds{stage}      # embed / retrieve / rerank / synth

# Ressources
myeline_users_total{tier,is_admin}
myeline_orgs_total{plan}
myeline_conversations_active

# Quotas / audit
myeline_quota_usage{user_id,metric}
myeline_audit_events_total{action}

# Provisioning Enterprise
myeline_enterprise_stacks{status}
```

Pas d'auth par défaut sur `/metrics` — restreindre via le
reverse proxy (allow-list IP du scraper Prometheus).

## Logs

- **Logs applicatifs** : sortie standard du conteneur `web` (capturée
  par podman / journald). Format JSON structuré (slog-like).
- **Logs mailer** (en souverain pur) : `data/logs/mailer/<date>.log`
  — chaque envoi simulé y est tracé en clair pour permettre la
  récupération manuelle des liens (confirmation, reset, invitation).
- **Logs cron** : `data/logs/cron/<task>.log`.

Brancher Loki / Vector / journald → Elastic selon votre standard.
Aucun PII n'est loggé hors `data/logs/mailer/` (et ce dossier doit
être traité comme sensible — voir [RGPD](../compliance/gdpr.md)).

## Alertes recommandées

| Alerte                           | Sévérité | Seuil                     |
|----------------------------------|----------|---------------------------|
| `/healthz` non 200 pendant 1 min | critique | downtime probable         |
| `/health` `degraded` > 10 min    | warning  | dépendance externe down   |
| `db` ou `redis` down             | critique | impact direct utilisateurs|
| 5xx HTTP > 1 % sur 5 min         | warning  | bug applicatif probable   |
| RAG p95 > 30 s sur 5 min         | warning  | LLM lent / saturé         |
| Quota usage > 90 %               | info     | scaling à anticiper       |
| Disque libre < 20 %              | warning  | backup / chroma à purger  |

Templates Grafana fournis sur demande (`hello@myeline.io`).
