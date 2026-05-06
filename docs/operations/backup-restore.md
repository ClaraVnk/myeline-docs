# Sauvegarde et restauration

Myeline tourne avec **trois sources de vérité** à sauvegarder :

1. **MariaDB** — comptes, organisations, métadonnées documents,
   journal d'audit, conversations.
2. **ChromaDB** — vecteurs d'embedding + métadonnées (sources des
   réponses RAG).
3. **Volumes de fichiers** — uploads utilisateur (`data/uploads/`),
   logs (`logs/`), assets statiques personnalisés (`static/custom/`).

Sans les trois, une restauration est incomplète.

## Backup automatique quotidien

Le cron `backup_databases` s'exécute tous les jours à **02:30 UTC**
(off-peak, modifiable dans `scripts/crontab`). Il produit dans
`/var/lib/myeline/backups/` :

```
backups/
├── mariadb-2026-05-06.sql.gz       # mysqldump compressé
├── chroma-2026-05-06.tar.zst       # snapshot ChromaDB
└── volumes-2026-05-06.tar.zst      # uploads + logs custom
```

**Rétention locale** : 14 jours (variable `BACKUP_RETENTION_DAYS`).
Au-delà, les fichiers sont supprimés du host.

## Backup off-host (recommandé)

Pour une vraie résilience, **synchroniser** `/var/lib/myeline/backups/`
vers un stockage **distinct** (autre datacenter, autre fournisseur).

### Option A — `rclone` (souverain-hybride uniquement)

```bash
# Configuration interactive
rclone config
# Puis dans /etc/cron.d/myeline-backup-offsite :
30 3 * * *  root  rclone sync /var/lib/myeline/backups remote:myeline-backups --transfers=4 --checksum
```

### Option B — MinIO interne (souverain pur)

```bash
mc alias set local-minio https://minio.dmz.local:9000 ACCESS SECRET
30 3 * * *  root  mc mirror /var/lib/myeline/backups local-minio/myeline-backups
```

### Option C — `borgbackup` (chiffrement client)

Recommandé si la cible est partiellement de confiance — chiffrement
AES-256 client + déduplication.

## Restauration complète

```bash
# 1. Stopper la stack
podman-compose down

# 2. Restaurer MariaDB
zcat backups/mariadb-2026-05-06.sql.gz | podman exec -i myeline-db mariadb -u root -p$DB_ROOT myeline

# 3. Restaurer ChromaDB
tar --use-compress-program=unzstd -xf backups/chroma-2026-05-06.tar.zst -C data/

# 4. Restaurer les volumes
tar --use-compress-program=unzstd -xf backups/volumes-2026-05-06.tar.zst -C data/

# 5. Redémarrer
podman-compose up -d
podman-compose exec web flask db upgrade
```

## Test de restauration

**Recommandation forte** : tester la procédure de restauration sur
un environnement vierge **au moins une fois par trimestre**. Un
backup non testé = pas de backup.

## Rotation des secrets après une fuite

Si une sauvegarde a fuité, **rotater immédiatement** :

- `SECRET_KEY` (déconnecte toutes les sessions)
- `CLOUD_TOKEN_KEY` (invalide tous les tokens OAuth chiffrés —
  les utilisateurs devront reconnecter leurs drives)
- Mots de passe DB, Redis
- Tokens API externes (Mistral, Anthropic, OpenAI, Gemini)
- Clé de licence (contacter Myeline pour réémission)

Voir [Erreurs de licence](../troubleshooting/license-errors.md) pour
le détail de la rotation de licence.
