# Utilisation

Cette section couvre l'usage **côté utilisateur final** — les
fonctionnalités exposées via l'interface web.

- [Bibliothèque](upload-documents.md) — alimentation perso et org
  via les connecteurs cloud et les scrapers (pas d'upload manuel),
  formats indexés, pipeline.
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
