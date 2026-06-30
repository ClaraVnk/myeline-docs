# Bibliothèque

Myeline n'a **aucune fonction d'upload manuel de fichier**. La
bibliothèque d'un utilisateur (et celle d'une organisation) est
alimentée exclusivement par deux canaux automatiques :

- **Connecteurs cloud** — Google Drive, OneDrive, Dropbox, kDrive,
  Zotero, S3 / S3-compatible, WebDAV
- **Scrapers RSS / web**

C'est un choix de design : centraliser les sources de vérité chez
le client sur des drives qu'il maîtrise déjà, sans dupliquer les
contenus dans Myeline.

## Périmètres

`/user/ma-bibliotheque` agrège vos documents personnels.
`/org/<slug>/workspace` agrège ceux partagés au sein d'une
organisation. Chaque périmètre dispose de sa propre collection
ChromaDB :

| Périmètre        | Collection ChromaDB     | Visibilité            |
|------------------|-------------------------|-----------------------|
| Personnel        | `user_<id>_personal`    | Vous uniquement       |
| Organisation     | `org_<id>_shared`       | Membres de l'org      |
| Public plateforme| `shared`                | Tous les utilisateurs |

## Connecteurs cloud

Connectez votre drive depuis `/user/cloud` (ou `/org/<slug>/cloud`
côté organisation) :

- Google Drive, OneDrive, Dropbox, kDrive (souverain-hybride
  uniquement, voir [Connecteurs cloud](cloud-connectors.md))
- S3 / S3-compatible (toutes éditions)
- WebDAV — Nextcloud, ownCloud (toutes éditions)
- Zotero (souverain-hybride uniquement, indexe les métadonnées
  bibliographiques)

Le cron `check_cloud_sync` (toutes les 4 h, plancher tier-dépendant)
détecte les nouveaux fichiers et les ajoute à votre collection.
Vous pouvez aussi déclencher une sync manuelle (1 fois/h max).

### Formats indexés

| Format              | Extensions             | Notes                                |
|---------------------|------------------------|--------------------------------------|
| **PDF**             | `.pdf`                 | extraction texte via `pdfplumber`    |
| **Word**            | `.docx`                | via `python-docx`                    |
| **OpenDocument**    | `.odt`                 | via `odfpy`                          |
| **Texte brut**      | `.txt`, `.md`          | UTF-8 attendu                        |
| **HTML**            | `.html`, `.htm`        | nettoyage via `bleach`               |

Les fichiers d'autres formats (XLSX, PPTX, ZIP, vidéo, image…) sont
ignorés silencieusement par l'indexer. Limite de taille : 50 MB par
fichier (configurable via `MAX_UPLOAD_SIZE_MB`). L'OCR sur PDF
scannés est désactivé par défaut (activable via
`RAG_OCR_ENABLED=true` en souverain-hybride uniquement, dépendance
Tesseract).

## Scrapers RSS / web

`/user/scrapers` (Pro+) permet de surveiller des flux RSS ou des
pages web — chaque article récupéré est ajouté à votre bibliothèque
personnelle. Un membre d'organisation peut « pousser » un scraper
vers la bibliothèque commune (bouton 🏢 sur la page des scrapers).

Le cron `check_user_scrapers` tourne 3 fois par jour en heures
creuses (03:00 / 07:00 / 23:00 UTC) pour ne pas saturer le serveur
en heures ouvrées.

## Pipeline d'indexation

```
Nouveau fichier détecté (sync cloud) ou article récupéré (scraper)
  → Détection MIME (magic bytes)
  → Extraction texte
  → Découpage en chunks (~500 tokens, overlap 50)
  → Embedding (on-prem : bge-m3 Ollama local — Cloud : API Mistral UE)
  → Insertion ChromaDB dans la collection cible
  → Document interrogeable en RAG
```

L'indexation est **asynchrone** (RQ worker). Sur un PDF de 100
pages, compter 30-90 secondes après la sync.

## Suppression et ré-indexation

- **Désindexer un document** depuis la bibliothèque : retire les
  chunks ChromaDB. Le fichier reste dans le drive d'origine.
- **Ré-indexer une collection complète** : `/admin/library/reindex`
  côté admin global (opération coûteuse, utile après upgrade du
  modèle d'embedding).

## Single-document chat

Cliquer sur un document (perso ou org) ouvre un chat **scopé à ce
seul document** — utile pour fouiller un long PDF sans pollution
des autres sources. Voir
[Recherche RAG § single-document](rag-search.md#single-document).

## Pourquoi pas d'upload manuel ?

Trois raisons :

1. **Source unique de vérité** — quand un document évolue chez le
   client (nouveau PDF dans le Drive), Myeline le détecte
   automatiquement à la prochaine sync. Pas de dérive entre la
   version uploadée et la version « officielle ».
2. **Conformité** — le document reste dans le système de
   classification du client (Drive, SharePoint, Nextcloud…) qui
   gère déjà ses droits, ses backups, sa rétention.
3. **Simplicité** — un seul chemin d'ingestion à maintenir et à
   monitorer (le sync cloud), au lieu de doubler avec un upload
   form qui a ses propres bugs et limites.

Si votre cas d'usage demande quand même un upload manuel (par ex.
documents one-shot non liés à un drive), contactez-nous —
techniquement faisable, à arbitrer selon votre contexte.
