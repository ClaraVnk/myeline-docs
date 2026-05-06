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

## OAuth refusé / `redirect_uri_mismatch`

**Symptôme** : « Erreur lors de la connexion OAuth », redirection
infinie.

**Cause** : le redirect URI déclaré dans votre app OAuth (Google
Console / Azure / Dropbox / etc.) ne correspond pas à celui que
Myeline génère.

**Résolution** :

1. Vérifier l'URL générée :

   ```bash
   podman exec myeline-web flask shell -c "
   from flask import url_for
   print(url_for('user_cloud.gdrive_callback', _external=True))
   "
   ```

2. Copier-coller exactement cette valeur dans la liste « Redirect
   URIs autorisés » de l'app OAuth chez le provider.

3. Vérifier `OAUTH_REDIRECT_BASE_URL` dans `.env` — doit pointer
   vers l'URL publique exacte (ex. `https://myeline.acme.local`),
   pas une IP privée.

## Token expiré non renouvelé

**Symptôme** : `last_error: "401 Unauthorized"`, `status: error`.

**Cause** : le refresh token a été révoqué (mot de passe changé chez
le provider, app désautorisée, période d'inactivité dépassée).

**Résolution** : l'utilisateur doit **reconnecter son drive** depuis
`/user/cloud` (bouton « Reconnecter » à côté de la connexion en
erreur). L'historique de sync est préservé.

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

## Standard tier — limites

Les comptes / licences **Standard** ont des limites strictes :

- **1 seule** connexion cloud active.
- **1 sync par 24 h**, peu importe la cadence du cron.

Le banner de progression `/user/cloud` affiche explicitement le
prochain créneau autorisé. Aucun moyen de bypasser sans upgrade
de plan / licence.
