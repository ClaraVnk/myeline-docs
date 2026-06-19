# Échecs de synchronisation

Les connecteurs cloud peuvent échouer pour des raisons très
différentes selon le provider. Cette page recense les symptômes
les plus fréquents.

## Diagnostic général

```bash
# Statut des connexions
podman exec myeline-web flask shell << 'EOF'
from app.models.cloud_connection import CloudConnection
for c in CloudConnection.query.all():
    print(f"{c.id} {c.provider} user={c.user_id} status={c.status} last_sync={c.last_sync_at} last_error={c.last_error}")
EOF

# Logs cron
podman exec myeline-cron cat /var/log/cron/check_cloud_sync.log | tail -50

# Force une sync manuelle
podman exec myeline-web flask run-cron check_cloud_sync
```

## Google Drive — dossier inaccessible au service account

**Symptôme** : à la connexion d'un dossier Drive (`/user/cloud/gdrive/connect`),
message *« Impossible d'accéder au dossier — vérifiez le partage »*, ou
`last_error: "403 Forbidden"` lors de la sync.

**Cause** : le dossier n'a pas été partagé avec l'email du service account,
ou seulement avec un rôle insuffisant (ex. partagé uniquement avec un
groupe Workspace qui n'inclut pas le SA).

**Résolution** :

1. Récupérer l'email du service account affiché dans
   `/user/cloud/gdrive/connect` (forme
   `myeline-drive-reader@<project>.iam.gserviceaccount.com`).
2. Côté Google Drive UI : clic droit sur le dossier → **Partager** →
   ajouter cet email exact en rôle **Lecteur** → **Envoyer**.
3. Re-tenter la connexion sur Myeline.

> Si vous indexez un dossier dans un **Drive partagé** (Workspace shared
> drive), le service account doit être membre du shared drive lui-même —
> le partage de fichier individuel ne suffit pas. Ajoutez le SA en
> *Lecteur* dans Drive Web UI → Drives partagés → *votre drive* → Membres.

## Token API expiré (OneDrive, Dropbox, Notion)

**Symptôme** : `last_error: "401 Unauthorized"`, `status: error`.

**Cause** : le refresh token OAuth a été révoqué (mot de passe changé
chez le provider, app désautorisée, période d'inactivité dépassée).

**Résolution** : l'utilisateur doit **reconnecter son drive** depuis
`/user/cloud` (bouton « Reconnecter » à côté de la connexion en
erreur). L'historique de sync est préservé.

> Ne s'applique **pas à Google Drive** : depuis Myeline v1.0.2, Drive
> utilise un compte de service (server-to-server) qui ne dépend pas
> de tokens OAuth utilisateur. Voir [Connecteurs cloud § Google Drive](../admin/cloud-connectors.md#google-drive--compte-de-service).

## OAuth refusé / `redirect_uri_mismatch` (OneDrive, Dropbox, Notion)

**Symptôme** : « Erreur lors de la connexion OAuth », redirection
infinie.

**Cause** : le redirect URI déclaré dans votre app OAuth (Azure /
Dropbox / Notion) ne correspond pas à celui que Myeline génère.

**Résolution** :

1. Vérifier l'URL générée :

   ```bash
   podman exec myeline-web flask shell -c "
   from flask import url_for
   print(url_for('user_cloud.cloud_callback', provider='onedrive', _external=True))
   "
   ```

2. Copier-coller exactement cette valeur dans la liste « Redirect
   URIs autorisés » de l'app OAuth chez le provider.

3. Vérifier `OAUTH_REDIRECT_BASE_URL` dans `.env` — doit pointer
   vers l'URL publique exacte (ex. `https://myeline.acme.local`),
   pas une IP privée.

## 429 / Rate limit

**Symptôme** : `last_error: "429 Too Many Requests"`.

**Cause** : trop de requêtes vers l'API du provider — typiquement
sur un drive de plusieurs centaines de Go avec sync forcée.

**Résolution** :

- Patienter — la sync reprendra au prochain cron (4 h).
- Réduire la fréquence côté connexion (paramètre « Intervalle de
  sync » sur `/user/cloud`).
- Vérifier qu'aucune autre app n'épuise le quota du même compte
  Google / Microsoft.

## Fichiers ignorés (silencieux)

**Symptôme** : un fichier est dans le drive mais n'apparaît pas
dans la bibliothèque.

**Causes possibles** :

- **Format non supporté** (XLSX, PPTX, ZIP, vidéo, image…). Voir la
  liste des formats indexés dans
  [Bibliothèque et upload](../usage/upload-documents.md).
- **Taille > 50 MB** (rejet silencieux côté indexer).
- **Document chiffré côté provider** (Drive « Confidentialité
  protégée », SharePoint « Information rights management »). L'API
  refuse le téléchargement.
- **Trash / corbeille** : les fichiers à la corbeille ne sont jamais
  indexés.
- **Filtre par dossier** : si l'utilisateur a sélectionné un sous-dossier
  spécifique lors de la connexion, les fichiers hors de ce dossier
  sont ignorés.

Vérification :

```bash
# Logs détaillés du worker pendant une sync
podman exec myeline-worker rq info
podman logs --tail 200 myeline-worker | grep -i "skip\|ignore\|reject"
```

## Échec de l'embedding

**Symptôme** : la sync se termine en succès mais le fichier reste
en « En cours d'indexation » indéfiniment.

**Cause** : Ollama injoignable au moment où le worker traite le
fichier. Le worker fait 3 retries puis abandonne — le fichier est
dans la queue mais pas embeddé.

**Résolution** :

```bash
# Relancer la queue
podman exec myeline-worker rq requeue --queue myeline-pro --all
podman exec myeline-worker rq requeue --queue myeline --all
```

Vérifier ensuite l'état Ollama (voir [Problèmes Ollama](ollama-issues.md)).

## Free tier — limites

Les comptes **Free** ont des limites strictes :

- **1 sync par 24 h**, peu importe la cadence du cron.
- Bibliothèque plafonnée à **500 documents**.
- Sources RSS/web personnalisées plafonnées à **50 sources**, **100
  articles par source**.

Le banner de progression `/user/cloud` affiche explicitement le
prochain créneau autorisé. Aucun moyen de bypasser sans passer au
plan **Pro**.
