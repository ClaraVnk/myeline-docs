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

## Fournisseurs IA par organisation (Enterprise / Sovereign-hybride)

Depuis `/org/<slug>/admin`, owner et admin d'une org Enterprise peuvent
configurer deux fournisseurs indépendants :

- **LLM** (synthèse des réponses) : carte « Fournisseur IA » —
  Mistral / OpenAI / Claude / Gemini avec votre clé, ou rester sur le
  défaut de la plateforme.
- **Embedding** (vectorisation des documents et des requêtes pour la
  collection `org_{id}_shared`) : carte « Fournisseur d'embedding
  (BYOK) ».

### Bloc « Fournisseur d'embedding (BYOK) »

| Fournisseur | Modèle par défaut             | Dim   | Commentaire                          |
|-------------|-------------------------------|-------|--------------------------------------|
| Mistral     | `mistral-embed`               | 1024  | 🇫🇷 hébergé UE, RGPD                |
| OpenAI      | `text-embedding-3-small`      | 1536  | `-3-large` 3072 dim aussi supporté   |
| Voyage      | `voyage-3`                    | 1024  | Qualité top                          |
| Cohere      | `embed-multilingual-v3.0`     | 1024  | Multilingue (100+ langues)           |

Procédure pour activer ou changer de fournisseur :

1. Cliquer sur la tuile du fournisseur cible, coller la clé API
   (`sk-...`, `voyage-...`, etc.), laisser le champ « modèle » vide
   pour utiliser le défaut ou spécifier une variante.
2. Si la collection `org_{id}_shared` existait déjà sous un autre
   provider/modèle, le formulaire **refuse la bascule** avec un
   message qui pointe la commande à lancer côté serveur.
3. Wipe + réindexation :
   ```bash
   flask wipe-org-embed-collection --org <id> --confirm
   # Ré-activer le provider via /org/<slug>/admin
   flask reindex-embeddings
   ```
4. Pendant la réindexation, les recherches restent fonctionnelles mais
   peuvent être dégradées (les premiers résultats reviendront sur le
   nouvel espace vectoriel à mesure que les chunks sont re-embedés).

**Sécurité.** Les clés sont chiffrées (`org.embed_api_key_enc`, Fernet)
et jamais réémises en clair par l'UI — seul un masque `••••••••`
confirme qu'une clé est enregistrée. La clé d'embedding est distincte
de la clé LLM, vous pouvez les changer indépendamment.

**Périmètre.** Le routing embed BYOK s'applique uniquement à la
collection partagée de l'organisation. La bibliothèque personnelle de
chaque membre (`user_{id}_personal`) et la collection publique
(`shared`) restent sur le fournisseur par défaut du déploiement, parce
qu'un user peut avoir un abonnement individuel hors org.

**Désactivation.** Le bouton « Désactiver » de la carte vide les trois
colonnes (`embed_provider`, `embed_api_key_enc`, `embed_model`) et
ramène l'org au défaut. Les vecteurs précédents restent en place — il
est conseillé de réindexer ensuite pour aligner l'espace vectoriel sur
le nouveau provider par défaut.
