# Organisations et utilisateurs

Une **organisation** est un espace de travail partagé dans Myeline :
collection ChromaDB dédiée, bibliothèque commune, membres avec rôles,
fournisseur LLM par défaut configurable (en souverain-hybride).

## Modèle de rôles

| Rôle    | Peut...                                                                  |
|---------|---------------------------------------------------------------------------|
| Owner   | tout (incluant supprimer l'organisation, transférer le owner)             |
| Admin   | gérer membres, OIDC, quotas, ré-indexer la collection                     |
| Member  | uploader des documents, lancer des recherches, créer des conversations    |

Un utilisateur peut appartenir à **plusieurs organisations**. Le
sélecteur d'organisation en haut à droite change le contexte de
recherche RAG (collection ChromaDB ciblée).

## Créer une organisation

`/admin/orgs/new` — saisir le nom, le slug (segment URL), l'email du
owner. Si l'email correspond à un compte existant, l'utilisateur est
ajouté comme owner ; sinon, un compte est créé puis une invitation
est envoyée par email (en souverain pur, log-only — récupérer le
lien dans `logs/mailer/`).

## Inviter des membres

Depuis `/org/<slug>/members`, le owner ou un admin peut :

- Inviter par email (envoi d'un lien d'enrôlement)
- Promouvoir un member en admin
- Retirer un membre
- Transférer le rôle owner (action protégée par confirmation)

## Suppression d'un compte (RGPD)

- **Suppression douce** : `/account/delete` côté utilisateur — anonymise
  email, prénom, nom et libère les ressources personnelles. La trace
  d'audit est conservée 13 mois (registre RGPD, durée légale).
- **Suppression définitive** : `flask delete-user --email=…` côté
  CLI admin — purge complète, y compris collection ChromaDB
  personnelle.

Voir [Conformité RGPD](../compliance/gdpr.md) pour le détail.
