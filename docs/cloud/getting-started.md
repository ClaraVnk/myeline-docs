# Premiers pas (Cloud)

Ce guide vous emmène de la création de compte à votre première réponse
RAG, sur l'offre hébergée [myeline.io](https://myeline.io). Comptez
**5 à 10 minutes**.

!!! info "Pour le Cloud uniquement"
    Cette page concerne l'édition **Cloud (SaaS)**. Pour l'usage
    détaillé des fonctionnalités (commun à toutes les éditions), voir
    les [Concepts (toutes éditions)](../concepts/index.md).

## 1. Créer un compte

Rendez-vous sur [myeline.io](https://myeline.io) et créez un compte
gratuit (email + mot de passe). Activez la **2FA TOTP** depuis
*Paramètres → Sécurité* — recommandé dès le premier jour.

Vous démarrez en **Free** : gratuit, sans limite de durée, avec votre
propre clé API.

## 2. Brancher une clé API (Free) ou passer Pro

=== "Free — votre clé (BYOK)"

    En Free, **votre clé API** sert à l'indexation **et** à la synthèse.
    Privilégiez **Mistral** : une seule clé couvre les deux. Suivez
    [Obtenir une clé API gratuite en 2 minutes](../concepts/get-api-key.md),
    puis collez la clé dans *Paramètres → Fournisseur IA*.

    !!! warning "Sans clé, pas d'accès"
        Un compte Free doit configurer une clé valide avant
        d'atteindre le tableau de bord. Une clé **Claude** ou **Gemini**
        seule **ne peut pas indexer** (pas d'API d'embedding) — utilisez
        Mistral ou OpenAI comme clé unique.

=== "Pro — Mistral inclus"

    En **Pro** (7,90 €/mois, essai 15 jours), **Mistral est inclus** :
    aucune clé à gérer, indexation et synthèse fonctionnent dès
    l'inscription. Activez l'essai depuis le tableau de bord
    (*Passer à Pro*). Vous pouvez tout de même brancher votre propre clé
    (BYOK) pour surcharger le provider de synthèse.

## 3. Connecter une source

Dans *Sources* (ou *Connecteurs*), ajoutez au moins une source à
indexer :

- **Connecteur cloud** : Google Drive, OneDrive, Dropbox, kDrive…
  Autorisez l'accès via OAuth ; Myeline synchronise et indexe les
  documents. Détails et périmètres : [Connecteurs cloud](../concepts/cloud-connectors.md).
- **Source RSS / web** : collez l'URL d'un flux ou d'un site ; Myeline
  détecte RSS vs HTML et scrape le contenu.

La première synchronisation peut prendre quelques minutes selon le
volume. Voir [Bibliothèque](../concepts/library.md) pour le pipeline
d'indexation et les formats supportés.

## 4. Première recherche RAG

Une fois l'indexation terminée, ouvrez la **recherche** et posez une
question en langage naturel. Myeline récupère les passages pertinents,
rédige une réponse et cite ses sources avec des liens cliquables `[N]`.

- Sélectionnez le **périmètre** (personnel / organisation / public).
- Basculez en **mode strict** pour n'autoriser que les réponses
  sourcées.
- Enchaînez en **conversation multi-tours** (Pro) pour affiner.

Tout est détaillé dans [Recherche RAG](../concepts/rag-search.md) et
[Conversations multi-tours](../concepts/conversations.md).

## 5. Passer à Pro (optionnel)

Pour lever toutes les limites (documents et sources illimités, recherche
augmentée, chat multi-tours, alertes de veille, sync horaire), passez en
**Pro** depuis votre **tableau de bord** (bouton *Passer à Pro*, essai
15 jours). Mistral est alors inclus.

Pour travailler à plusieurs, créez une **organisation** : facturation
**par siège (7,90 €/membre)**, bibliothèque partagée.

## Et ensuite ?

- [Alertes et veille](../concepts/watch-alerts.md) — surveiller un
  mot-clé et recevoir un digest.
- [FAQ Cloud](faq.md) — tarifs, essai, équipes, résidence des données.
- [Choisir son édition](../editions/index.md) — si vos besoins
  évoluent vers l'on-premise.
