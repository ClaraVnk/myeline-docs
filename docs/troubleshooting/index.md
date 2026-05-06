# Dépannage

Guide de résolution des problèmes courants. Si vous ne trouvez pas
votre situation ici, contactez [hello@myeline.io](mailto:hello@myeline.io)
en joignant :

- La sortie de `curl -s https://votre-domaine/health`
- Les 100 dernières lignes des logs du conteneur `web` (`podman logs --tail 100 myeline-web`)
- La version installée : `curl -s https://votre-domaine/healthz | jq .version`

## Sections

- [Erreurs de licence](license-errors.md) — `LicenseRevokedError`,
  `LicenseExpiredError`, signature invalide, horloge décalée.
- [Problèmes Ollama](ollama-issues.md) — modèle non chargé, OOM,
  latence excessive, GPU non détecté.
- [Échecs de synchronisation](sync-failures.md) — OAuth refusé,
  token expiré, 429 du provider, fichiers ignorés.

## Diagnostic rapide

```bash
# 1. État global
curl -s https://votre-domaine/health | jq

# 2. Logs récents
podman logs --tail 50 myeline-web
podman logs --tail 50 myeline-cron
podman logs --tail 50 myeline-worker

# 3. Connectivité dépendances
podman exec myeline-web flask diagnose

# 4. Mode de déploiement détecté
podman exec myeline-web python -m app.utils.deployment_mode
```

`flask diagnose` (commande interne) tente une connexion DB / Redis /
Ollama / mailer / provider IA et imprime un rapport synthétique.

## Quand redémarrer ?

| Symptôme                                     | Action                                |
|----------------------------------------------|---------------------------------------|
| Web ne répond plus, healthz timeout          | `podman restart myeline-web`          |
| Workers RQ bloqués, queue grossit            | `podman restart myeline-worker`       |
| Cron skippé sans raison                      | `podman restart myeline-cron`         |
| Embedding lent + GPU plein                   | `podman restart ollama`               |
| Modif de `.env`                              | `podman-compose down && podman-compose up -d` (cf. CLAUDE.md) |
| Migration DB après `git pull`                | `podman exec myeline-web flask db upgrade` |

Un simple `podman-compose restart web` **ne reload pas** les
variables d'env — toujours `down + up` après modification du `.env`.
