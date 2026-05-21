# Installation souveraine (air-gap)

Walkthrough complet pour déployer Myeline en édition **Souveraine**
sur votre infrastructure, sans aucun appel API externe.

!!! info "Avant de commencer"
    Vous devez avoir reçu :

    - Une clé de licence par email après signature du devis (format
      `MYE-...`)
    - Un accès au repo Git Myeline (clé SSH déposée dans votre compte
      GitHub fourni par nous)

## 1. Pré-requis

Vérifier que votre serveur correspond aux
[pré-requis serveur](prerequisites.md). Pour le souverain :

- **Sans GPU** : 16 vCores, 32 GB RAM, 200 GB NVMe minimum
- **Avec GPU** : RTX 4090 24 GB ou L40S recommandé pour ≤ 200 utilisateurs

Vérifier que Podman 4.6+ et podman-compose sont installés :

```bash
podman --version       # → 4.6.x ou plus
podman-compose --version
```

Si manquant sur Rocky / AlmaLinux 9 :

```bash
sudo dnf install -y podman
sudo pip3 install podman-compose
```

Vérifier le **socket Podman rootless** (utilisé pour le provisionnement
Enterprise — pas obligatoire en sovereign mais utile à activer dès
maintenant) :

```bash
systemctl --user enable --now podman.socket
ls /run/user/$(id -u)/podman/podman.sock   # doit exister
```

## 2. Cloner le repo

```bash
git clone -b synapse git@github.com:ClaraVnk/myeline.git
cd myeline
```

## 3. Lancer l'installeur guidé

```bash
./scripts/install.sh
```

Le script vous accueille et vérifie le système :

```
╔═══════════════════════════════════════════════════════════╗
║    m y e l i n e    —  installateur guidé              ║
║    plateforme RAG self-hosted, FR / EU-hosted            ║
╚═══════════════════════════════════════════════════════════╝

── Pré-vérifications du système
[✓] Podman 4.6.0 détecté
[✓] 165 GB d'espace disque disponibles
[✓] 32 GB de RAM
[✓] 16 cœurs CPU
```

### 3.1 Choisir le mode

```
── Choix du mode de déploiement

  2) Installation souveraine        ← votre choix
  3) Installation souverain-hybride

    Votre choix [2/3] : 2
[✓] Mode : sovereign
```

### 3.2 Coller la clé de licence

```
── Clé de licence Myeline

    Clé de licence : MYE-eyJjdXN0b21lciI6IkFDTUUgQ29ycCIsImV4cGlyZXNfYXQiOiIyMDI3...
[✓] Format de licence valide (vérification cryptographique au boot)
```

La validation cryptographique réelle (signature Ed25519) a lieu au
premier démarrage de l'app — voir [Erreurs de licence](../troubleshooting/license-errors.md)
si l'app refuse de démarrer.

### 3.3 Configuration domaine

```
── Domaine public

    Domaine [myeline.local] : myeline.acme.local
[✓] URL publique : https://myeline.acme.local
```

Le domaine sert à construire les URLs absolues dans les emails et
les redirect URIs OAuth (non utilisés en sovereign mais préservés
pour cohérence). En air-gap strict, utilisez un domaine interne
résolu par votre DNS d'entreprise.

### 3.4 Compte administrateur initial

```
── Compte administrateur initial

    Email admin : admin@acme.local
    Mot de passe (≥ 12 caractères) : ************
    Confirmer : ************
[✓] Compte admin configuré
```

Conservez ce mot de passe en lieu sûr. Un récupération de password
est possible plus tard via la CLI `flask update-admin`.

### 3.5 Mailer (forcé en log-only)

```
── Emails transactionnels (Brevo / Sendinblue)

  En mode souverain, le mailer est forcé en log-only
  (les emails sont écrits dans logs/mailer/ au lieu d'être envoyés).

    Configurer Brevo maintenant ? [o/N] : N
[i] Mailer en mode dry-run (emails écrits dans les logs).
```

En souverain, vous pouvez plus tard configurer un MTA / SMTP interne
(Postfix sur votre réseau) en éditant `.env` et en désactivant le
log-only (`MAIL_DRY_RUN=false`). Chantier custom à discuter avec
nous selon votre infra.

### 3.6 Configuration IA (Ollama)

```
── Synthèse IA

  En souverain, toute la synthèse passe par Ollama local.

    URL Ollama [http://ollama:11434] :
    Modèle d'embedding [bge-m3] :
    Modèle LLM de synthèse [mistral-nemo] : llama3.1:70b
[i] Pensez à pull le modèle après l'install : ollama pull llama3.1:70b
```

L'URL `http://ollama:11434` pointe vers le service Ollama interne au
docker-compose. Si vous utilisez un Ollama externe (sur un autre
serveur GPU dédié), pointez vers son URL réelle.

#### Choix du modèle d'embedding

**`bge-m3`** est le défaut et reste **notre recommandation** pour la
majorité des déploiements : 1024 dimensions, multilingue (excellent en
FR/EN), 568M paramètres, équilibre qualité/vitesse imbattable. Mais si
votre corpus est mono-langue ou votre matériel particulier, vous pouvez
basculer sur un autre modèle Ollama. Tous sont testés et supportés —
adaptez `OLLAMA_EMBED_MODEL` dans `.env` puis [réindexez](#reindexer-apres-changement-dembedder).

| Modèle Ollama | Dim | Langues | Taille | Quand l'utiliser |
|---------------|----:|---------|-------:|------------------|
| **`bge-m3`** *(défaut)* | 1024 | FR/EN/multi excellents | 568M | Cas général, multilingue |
| `mxbai-embed-large` | 1024 | EN très fort, FR correct | 334M | Corpus anglais à dominante technique |
| `snowflake-arctic-embed-l` | 1024 | EN très fort | 334M | Corpus anglais — alternative à mxbai |
| `nomic-embed-text` | 768 | EN principalement | 137M | CPU faible, corpus EN, beaucoup d'inférences |
| `paraphrase-multilingual` | 768 | Multilingue (qualité ↓) | 278M | Léger, multilingue, qualité moindre |
| `all-minilm` | 384 | EN/limité | 23M | Hardware très contraint, démos |

!!! warning "Compatibilité dimensionnelle"
    `bge-m3`, `mxbai-embed-large` et `snowflake-arctic-embed-l` produisent
    tous des vecteurs en **1024 dimensions** — vous pouvez basculer entre
    eux sans erreur ChromaDB, mais le **rerank-by-similarity sera dégradé**
    tant que les anciens vecteurs ne sont pas réindexés (les espaces
    vectoriels diffèrent même à dimension égale). Les autres modèles
    (768 / 384) requièrent une réindexation **obligatoire** sinon les
    requêtes échouent avec un mismatch dimensionnel.

#### Réindexer après changement d'embedder

Après avoir modifié `OLLAMA_EMBED_MODEL` :

```bash
# 1. Pull le nouveau modèle
podman-compose exec ollama ollama pull mxbai-embed-large

# 2. Redémarrer le web pour relire .env
podman-compose down && podman-compose up -d

# 3. Lancer la réindexation (peut prendre plusieurs minutes selon corpus)
podman-compose exec web flask reindex-embeddings
```

Pendant la réindexation, **la qualité de recherche est dégradée** mais
l'application reste accessible. Lancez idéalement hors heures de
bureau.

### 3.7 Connecteurs cloud

```
── Connecteurs de stockage cloud

[i] Mode souverain : seuls S3 / WebDAV (token API) seront proposés
    à l'utilisateur. Aucune clé OAuth à configurer.
```

Les utilisateurs configureront ensuite eux-mêmes leur connexion à
votre MinIO interne ou Nextcloud interne via `/user/cloud`. Pas de
configuration globale ici.

### 3.8 Backups off-host

```
── Backups hors-site (rclone)

    Configurer un backup hors-site maintenant ? [o/N] : O
    Remote rclone (nom:chemin) : minio-internal:myeline-backups
    Chemin du rclone.conf [~/.config/rclone/rclone.conf] :
[i] Voir docs/BACKUP.md pour la procédure complète.
```

**Pré-requis** : avoir lancé `rclone config` au préalable et
configuré une remote pointant vers votre **MinIO interne** ou
votre **NAS**. Aucun trafic ne sort de votre réseau si la remote
est interne.

### 3.9 Reste optionnel (Pangolin, GPU…)

Le script vous demande ensuite Pangolin (tunnel reverse-proxy —
typiquement non en air-gap). Skippez tout ce qui n'a pas de sens
dans votre contexte.

## 4. Démarrer la stack

À la fin, le script propose de lancer la stack :

```
    Démarrer la stack maintenant ? [O/n] : O

[i] Pull / build des images...
[i] Démarrage des services...
[i] Attente que MariaDB soit prêt...
[✓] MariaDB prêt
[i] Migrations DB + création du compte administrateur...
On-prem (sovereign) — single-tenant org created: My Organization
[✓] Stack opérationnelle
```

À la fin :

```
═══════════════════════════════════════════════════════════
   Installation terminée 🎉
═══════════════════════════════════════════════════════════

  Mode               : sovereign
  Domaine            : https://myeline.acme.local
  Email admin        : admin@acme.local
  Configuration      : /home/<user>/myeline/.env
```

## 5. Pull le modèle Ollama

```bash
podman-compose exec ollama ollama pull llama3.1:70b
# ou bien :
podman-compose exec ollama ollama pull mistral-nemo
podman-compose exec ollama ollama pull bge-m3   # déjà pull au boot normalement
```

Vérifier que le modèle est disponible :

```bash
podman-compose exec ollama ollama list
```

## 6. Configurer le reverse-proxy interne

Pointez votre Caddy / Nginx / Traefik interne vers `localhost:5000` :

```caddy
# /etc/caddy/Caddyfile
myeline.acme.local {
    reverse_proxy localhost:5000
    # PKI interne — votre AC d'entreprise
    tls /etc/pki/myeline.crt /etc/pki/myeline.key
}
```

## 7. Première connexion

Voir [Première connexion admin](first-login.md).

## Vérifications post-install

```bash
# Healthcheck
curl https://myeline.acme.local/healthz
# → {"status": "alive"}

# Status complet
curl https://myeline.acme.local/health
# → JSON avec DB / Redis / Ollama / Mistral / web (Mistral = "skipped")

# Vérifier qu'aucun appel ne sort en réseau
sudo iptables -L OUTPUT -v
# Il ne devrait y avoir aucun paquet vers 0.0.0.0/0 sauf vos infras internes
```

## Étapes suivantes

- [Première connexion admin](first-login.md) — configurer l'org, les
  utilisateurs, optionnellement OIDC SSO
- [Sauvegarde et restauration](../operations/backup-restore.md) —
  vérifier que la cron `backup_databases` a tourné
- [Renouvellement de licence](../operations/license-renewal.md) —
  noter la date d'expiration
