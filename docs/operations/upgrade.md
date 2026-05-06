# Mise à jour

Les éditions on-prem reçoivent les mises à jour de Myeline via une
**image OCI** publiée sur notre registre. Aucune mise à jour
automatique : vous décidez du moment.

## Cycle de release

- **Patch** (`x.y.Z`) : correctifs sécurité + bugs critiques —
  tous les 1-2 semaines.
- **Minor** (`x.Y.0`) : nouvelles fonctionnalités, migrations DB
  rétrocompatibles — tous les 1-2 mois.
- **Major** (`X.0.0`) : breaking changes (rares, communiqués 30
  jours à l'avance).

Un email arrive à l'admin de chaque déploiement à la sortie d'un
patch / minor (en souverain pur, log-only — abonnez-vous à la
mailing-list `releases@myeline.io` côté humain).

## Procédure standard

```bash
cd /opt/myeline

# 1. Sauvegarder avant tout (le cron a tourné cette nuit, mais une
#    sauvegarde fraîche évite la régression de la journée).
podman-compose exec web flask run-cron backup_databases

# 2. Pull la nouvelle image
podman-compose pull web cron worker

# 3. Recréer les conteneurs
podman-compose up -d

# 4. Appliquer les migrations DB (idempotent — rien si déjà à jour)
podman-compose exec web flask db upgrade

# 5. Vérifier la santé
curl -s https://votre-domaine/health | jq
```

`/health` doit renvoyer 200 et tous les composants `ok`. Si l'un est
`degraded` ou `down`, voir [Dépannage](../troubleshooting/index.md).

## Rollback

Toutes les images sont taguées avec leur version (`v2026.05.01`,
`v2026.04.20`, etc.) — `latest` n'est qu'un alias mobile. Pour
revenir à la version précédente :

```bash
# 1. Editer .env et fixer le tag explicite
echo 'MYELINE_VERSION=v2026.04.20' >> .env

# 2. Recréer
podman-compose up -d

# 3. Si une migration descendante est nécessaire :
podman-compose exec web flask db downgrade
```

⚠️ Une downgrade de schéma peut être **destructive** si la version
montante a ajouté des colonnes obligatoires. Préférer la
restauration depuis backup (voir
[Sauvegarde et restauration](backup-restore.md)) en cas de doute.

## Migrations en plusieurs étapes

Pour les changements à risque (drop de colonne, renommage massif),
nous appliquons le pattern **expand → migrate → contract** sur deux
releases :

1. Release N : ajout de la nouvelle colonne / structure (ancienne
   restant en place).
2. Backfill silencieux côté code applicatif.
3. Release N+1 : suppression de l'ancienne structure.

Aucun upgrade ne devrait jamais nécessiter un downtime > 30 secondes
(le temps du `podman-compose up -d`). Les migrations `flask db
upgrade` sont conçues pour s'appliquer en quelques secondes même
sur des bases de plusieurs Go.

## Mise à jour de la licence

Sans rapport avec la mise à jour de l'image. Voir
[Renouvellement de licence](license-renewal.md).
