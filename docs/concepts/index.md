# Concepts (toutes éditions)

!!! info "Usage commun"
    Ces concepts sont **communs à toutes les éditions** (Community,
    Cloud, on-premise) — légères variations selon l'édition. Ils
    décrivent l'usage **côté utilisateur final**, via l'interface web.

- [Obtenir une clé API](get-api-key.md) — créer une clé gratuite
  (Mistral / Gemini) pour l'indexation et la synthèse.
- [Bibliothèque](library.md) — alimentation perso et org
  via les connecteurs cloud et les scrapers (pas d'upload manuel),
  formats indexés, pipeline.
- [Connecteurs cloud](cloud-connectors.md) — Google Drive, OneDrive,
  Dropbox, kDrive, S3, WebDAV… activation et credentials.
- [Recherche RAG](rag-search.md) — poser une question, raffiner, citer
  les sources, mode strict.
- [Conversations multi-tours](conversations.md) — fil de discussion
  persistent, historique, export.
- [Alertes et veille](watch-alerts.md) — surveiller un mot-clé sur
  vos sources et recevoir un digest.

## Périmètre de recherche

Trois **scopes** de recherche, sélectionnables via le menu
contextuel :

| Scope          | Contenu                                                  |
|----------------|-----------------------------------------------------------|
| **Personnel**  | vos documents uploadés + vos drives cloud connectés       |
| **Organisation** | bibliothèque partagée par votre org (membres + admin)   |
| **Public**     | sources publiques de la plateforme (RSS, scrapers admin) |

L'utilisateur peut combiner les scopes (cocher plusieurs cases) — la
recherche RRF (Reciprocal Rank Fusion) fusionne les résultats des
collections ChromaDB sélectionnées.

## Quotas

Voir [Quotas et plans](../admin/quotas.md). En éditions on-prem,
toutes les fonctionnalités Pro+ sont activées par défaut (multi-turn,
conversations, alertes, single-document chat).
