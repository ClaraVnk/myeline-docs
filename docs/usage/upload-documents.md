# Téléverser des documents

Myeline distingue deux périmètres d'indexation : la **bibliothèque
personnelle** d'un utilisateur et la **bibliothèque partagée** d'une
organisation. Les modes d'alimentation sont différents.

## Bibliothèque personnelle

`/user/ma-bibliotheque` regroupe vos documents personnels. **Il n'y
a pas d'upload manuel** : la bibliothèque se remplit automatiquement
via deux canaux.

### Via les connecteurs cloud

Connectez votre drive depuis `/user/cloud` :

- Google Drive, OneDrive, Dropbox, kDrive (souverain-hybride
  uniquement, voir [Connecteurs cloud](../admin/cloud-connectors.md))
- S3 / S3-compatible (toutes éditions)
- WebDAV (Nextcloud, ownCloud — toutes éditions)
- Zotero (souverain-hybride uniquement, indexe les métadonnées
  bibliographiques)

Le cron `check_cloud_sync` (toutes les 4 h, plancher tier-dépendant)
détecte les nouveaux fichiers et les ajoute à votre collection
ChromaDB. Vous pouvez aussi déclencher une sync manuelle (1 fois/h).

### Via les scrapers RSS / web

`/user/scrapers` (Pro+) permet de surveiller des flux RSS ou des
pages web — chaque article récupéré est ajouté à votre bibliothèque.

## Bibliothèque d'organisation

Dans le contexte d'une **organisation**, l'admin et les membres
peuvent **uploader directement des fichiers** depuis `/org/<slug>/workspace` :

| Format              | Extensions             | Notes                                |
|---------------------|------------------------|--------------------------------------|
| **PDF**             | `.pdf`                 | extraction texte via `pdfplumber`    |
| **Word**            | `.docx`                | via `python-docx`                    |
| **OpenDocument**    | `.odt`                 | via `odfpy`                          |
| **Texte brut**      | `.txt`, `.md`          | UTF-8 attendu                        |
| **HTML**            | `.html`, `.htm`        | nettoyage via `bleach`               |

### Limites

- **Taille maximale par fichier** : 50 MB (configurable via
  `MAX_UPLOAD_SIZE_MB`)
- **Quota par organisation** : selon le plan (voir
  [Quotas](../admin/quotas.md))
- **Rate limit** : 20 uploads / heure / utilisateur

### Pipeline d'indexation

```
Upload (POST /org/<slug>/docs/upload)
  → MIME check (magic bytes)
  → Extraction texte
  → Découpage en chunks (~500 tokens, overlap 50)
  → Embedding bge-m3 (Ollama local)
  → Insertion ChromaDB (collection org_<id>_shared)
  → Document visible dans la recherche
```

L'indexation est **asynchrone** (RQ worker). Sur un PDF de 100 pages,
compter 30-90 secondes.

### Suppression et ré-indexation

- **Supprimer** un document : retire les chunks ChromaDB + le fichier
  source. Action admin uniquement.
- **Ré-indexer** une collection complète : `/admin/library/reindex`
  côté admin global, opération coûteuse (utile après upgrade du
  modèle d'embedding).

## Single-document chat

Cliquer sur un document (perso ou org) ouvre un chat **scopé à ce
seul document** — utile pour fouiller un long PDF sans pollution
des autres sources. Voir
[Recherche RAG § single-document](rag-search.md#single-document).
