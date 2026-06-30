# Édition Cloud (SaaS)

L'édition **Cloud** est la version hébergée de Myeline, accessible
directement sur [myeline.io](https://myeline.io) — **rien à
installer**. Vous créez un compte, vous connectez vos drives et vos
sources, et vous interrogez l'ensemble en langage naturel via RAG.

La plateforme est **hébergée en Europe (France)**, conforme RGPD.
L'embedding passe par l'**API d'embedding Mistral AI** (hébergée en
UE / France, pas d'entraînement sur les données API) : avec la **clé
plateforme incluse** en Pro, avec **votre propre clé** en Free. Le LLM
EU inclus pour la synthèse est **Mistral**. *(L'embedding local
Ollama + bge-m3 ne concerne que l'offre on-premise souveraine.)*

!!! info "Hébergé ou on-premise ?"
    Cette page couvre l'offre **hébergée** (Free / Pro). Si vous avez
    besoin de tout faire tourner sur **votre propre infrastructure**
    (air-gap, SecNumCloud, HDS), l'offre **On-premise / Souverain**
    est une installation **sur-mesure, sur devis** — voir
    [On-premise / Souverain](sovereign.md) ou écrivez-nous à
    [hello@myeline.io](mailto:hello@myeline.io).

## Free ou Pro ?

Deux formules hébergées. **Free** est réellement utilisable au
quotidien avec **votre propre clé API gratuite** ; **Pro** apporte
surtout le confort de **Mistral inclus — aucune clé à gérer**, la
levée de toutes les limites et le travail en équipe.

| Critère                          | Free (gratuit)                         | Pro — 7,90 €/mois                        |
|----------------------------------|----------------------------------------|------------------------------------------|
| **Hébergement**                  | Cloud Myeline (UE / France)            | Cloud Myeline (UE / France)              |
| **Synthèse IA**                  | **BYOK requis** (votre clé Mistral / OpenAI / Claude / Gemini) | **Mistral inclus** (BYOK possible pour surcharger) |
| **Embedding (indexation)**       | **Votre clé** (Mistral ou OpenAI — Claude/Gemini ne peuvent pas indexer) | **Mistral inclus** |
| **Recherche augmentée** (HyDE, multi-requêtes, reranking) | ❌            | ✅                                       |
| **Connecteurs cloud**            | Tous (Google Drive, OneDrive, Dropbox, kDrive…) | Tous                            |
| **Bibliothèque personnelle**     | Jusqu'à **500 documents**              | **Illimité**                             |
| **Sources RSS / web custom**     | **50 sources**, **100 articles/source** | **Illimité**                            |
| **Digest**                       | Mensuel uniquement                     | **Toutes fréquences**                    |
| **Chat multi-tours**             | ❌ (recherche RAG simple)              | ✅                                       |
| **Alertes de veille**            | ❌                                     | ✅                                       |
| **Sync cloud**                   | Toutes les **24 h** (fixe)            | Configurable, **jusqu'à toutes les heures** |
| **Équipe / organisation**        | ❌                                     | **Par siège — 7,90 €/membre**            |
| **Tarif**                        | 0 € (vous payez votre provider IA)     | **7,90 €/mois** (essai 15 jours)         |

!!! tip "🎁 Essai Pro gratuit — 15 jours"
    L'offre **Pro** inclut **15 jours d'essai gratuit** (carte requise,
    aucun débit avant J+15, annulable à tout moment). La facturation est
    **mensuelle**. L'édition **Free** reste gratuite indéfiniment, avec
    votre propre clé API.

## Free — gratuit, avec votre clé (BYOK)

L'édition Free est **gratuite et sans limite de durée**. La seule
contrainte : vous apportez **votre propre clé API IA** (BYOK), qui sert
**à la fois à l'indexation (embeddings) ET à la synthèse des
réponses**. Free tourne **entièrement sur votre clé** — Myeline ne
dépense **0 € d'API** pour vous.

Tout passe par votre clé, donc le choix du provider compte :

- une seule clé **Mistral** ou **OpenAI** couvre **les deux**
  (indexation + réponses) — ces providers exposent une API d'embedding ;
- une clé **Claude** ou **Gemini** peut **répondre** mais **ne peut pas
  indexer** (pas d'API d'embedding) — à éviter en clé unique sur Free.

**Sans clé, pas d'accès** : un utilisateur Free doit configurer une clé
valide avant d'atteindre le tableau de bord.

!!! info "Recherche augmentée : un plus de Pro"
    Les passes de **recherche augmentée** (HyDE, multi-requêtes,
    reranking, contextual retrieval) appellent le LLM plateforme et sont
    donc **désactivées en Free** — elles sont **incluses en Pro** comme
    valeur ajoutée.

Plusieurs providers proposent un **niveau gratuit** (Mistral, Gemini),
ce qui rend Free réellement utilisable sans aucune dépense — privilégiez
Mistral pour couvrir indexation **et** réponses avec une seule clé. Voir
le guide [Obtenir une clé API gratuite en 2 minutes](../usage/get-api-key.md).

Inclus dans Free :

- **Tous les connecteurs cloud** (Google Drive, OneDrive, Dropbox,
  kDrive…)
- **Bibliothèque personnelle jusqu'à 500 documents**
- **Sources RSS / web custom** : jusqu'à **50 sources**, plafonnées à
  **100 articles par source**
- **Digest mensuel**
- **Sync cloud toutes les 24 h** (fréquence fixe)
- **Recherche RAG** : indexation **et** synthèse exécutées sur votre clé

## Pro — 7,90 €/mois, Mistral inclus

L'édition Pro (**7,90 €/mois**, facturation **mensuelle**, **15 jours
d'essai gratuit**) inclut **Mistral AI** (France, hébergé UE, pas
d'entraînement sur les données API) pour **l'indexation (embeddings) ET
la synthèse** : aucune clé à gérer, tout fonctionne dès l'inscription.
Vous pouvez toujours **apporter votre propre clé** (BYOK) pour surcharger
le provider de synthèse si vous préférez Claude, GPT ou Gemini.

Pro reprend tout Free **sans aucune limite**, et ajoute la **recherche
augmentée** (HyDE, multi-requêtes, reranking, contextual retrieval),
désactivée en Free :

- **Documents, sources et articles illimités**
- **Toutes les fréquences de digest** (quotidien, hebdo, mensuel…)
- **Chat multi-tours** et **alertes de veille**
- **Sync cloud configurable, jusqu'à toutes les heures**
- **Organisation / équipe** facturée **par siège** : **7,90 €/membre**.
  Chaque membre dispose des fonctionnalités Pro complètes.

## On-premise / Souverain — sur devis

Vous ne pouvez pas envoyer vos données vers un SaaS — réglementation
HDS stricte, air-gap, SecNumCloud, exigence de souveraineté ? L'offre
**On-premise / Souverain** déploie Myeline **sur votre propre
infrastructure**.

C'est une **installation sur-mesure, sur devis** : périmètre, modèles
IA (Mistral, Ollama local, BYOK), connecteurs et intégration sont
calibrés avec vous. Il n'y a **pas de souscription en self-service** —
on en parle ensemble.

[:octicons-arrow-right-24: En savoir plus sur l'On-premise / Souverain](sovereign.md){ .md-button }
[:octicons-mail-24: Nous contacter](mailto:hello@myeline.io){ .md-button .md-button--primary }

## Comment s'inscrire

1. **Créez un compte gratuit** sur [myeline.io](https://myeline.io).
2. **Obtenez une clé API** auprès d'un provider IA (gratuit chez Gemini
   ou Mistral) — voir
   [Obtenir une clé API gratuite en 2 minutes](../usage/get-api-key.md).
3. **Collez votre clé** dans *Paramètres → Fournisseur IA*.
4. **Connectez vos drives** et ajoutez vos sources RSS / web.
5. Pour passer à **Pro**, faites-le depuis votre **tableau de bord**
   (bouton *Passer à Pro*, essai 15 jours) — Mistral est alors inclus et
   toutes les limites sont levées.
