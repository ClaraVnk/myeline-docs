# Troubleshooting

Resolution guide for common issues. If you don't find your situation
here, contact [hello@myeline.io](mailto:hello@myeline.io) and attach:

- Output of `curl -s https://your-domain/health`
- Last 100 lines of the `web` container logs
  (`podman logs --tail 100 myeline-web`)
- Installed version: `curl -s https://your-domain/healthz | jq .version`

## Sections

- [Licence errors](license-errors.md) — `LicenseRevokedError`,
  `LicenseExpiredError`, invalid signature, clock skew.
- [Ollama issues](ollama-issues.md) — model not loaded, OOM, excess
  latency, GPU not detected.
- [Sync failures](sync-failures.md) — OAuth refused, expired token,
  provider 429, ignored files.

## Quick diagnostic

```bash
# 1. Global state
curl -s https://your-domain/health | jq

# 2. Recent logs
podman logs --tail 50 myeline-web
podman logs --tail 50 myeline-cron
podman logs --tail 50 myeline-worker

# 3. Dependency connectivity
podman exec myeline-web flask diagnose

# 4. Detected deployment mode
podman exec myeline-web python -m app.utils.deployment_mode
```

`flask diagnose` (internal command) attempts a connection to DB /
Redis / Ollama / mailer / AI provider and prints a synthesised report.

## When to restart?

| Symptom                                        | Action                                |
|------------------------------------------------|---------------------------------------|
| Web no longer responds, healthz timeout        | `podman restart myeline-web`          |
| RQ workers stuck, queue grows                  | `podman restart myeline-worker`       |
| Cron skipped without reason                    | `podman restart myeline-cron`         |
| Slow embedding + GPU full                      | `podman restart ollama`               |
| `.env` change                                  | `podman-compose down && podman-compose up -d` (cf. CLAUDE.md) |
| DB migration after `git pull`                  | `podman exec myeline-web flask db upgrade` |

A simple `podman-compose restart web` **does not reload** environment
variables — always `down + up` after a `.env` change.
