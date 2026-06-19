# Édition Cloud (SaaS)

L'édition **Cloud** est la version hébergée de Myeline, accessible
directement sur [myeline.io](https://myeline.io) — **rien à
installer**. Vous créez un compte, vous connectez vos drives et vos
sources, et vous interrogez l'ensemble en langage naturel via RAG.

La plateforme est **hébergée en Europe (France)**, conforme RGPD.
L'embedding tourne **localement sur la plateforme** (jamais délégué à
un tiers) et le LLM EU inclus est **Mistral**.

!!! info "Cloud vs on-prem"
    Cette page couvre l'offre **hébergée** (Free / Pro / Enterprise).
    Si vous avez besoin de tout faire tourner sur **votre propre
    infrastructure** (air-gap ou BYOK on-prem), voir
    [les éditions on-prem](index.md).

## Free, Pro ou Enterprise ?

Trois formules. **Free** est réellement utilisable au quotidien avec
**votre propre clé API gratuite** ; **Pro** apporte surtout le confort
de **Mistral inclus — aucune clé à gérer** et la levée de toutes les
limites ; **Enterprise** est la version isolée pour les organisations.

| Critère                          | Free (gratuit)                         | Pro — 19,90 €/mois                      | Enterprise                         |
|----------------------------------|----------------------------------------|-----------------------------------------|------------------------------------|
| **Hébergement**                  | Cloud Myeline (UE / France)            | Cloud Myeline (UE / France)             | Stack dédiée et isolée             |
| **Synthèse IA**                  | **BYOK requis** (votre clé Anthropic / OpenAI / Gemini / Mistral) | **Mistral inclus** (BYOK possible pour surcharger) | Mistral inclus + BYOK              |
| **Embedding**                    | Local plateforme                       | Local plateforme                        | Local plateforme                   |
| **Connecteurs cloud**            | Tous (Google Drive, OneDrive, Dropbox, kDrive…) | Tous                          | Tous                               |
| **Bibliothèque personnelle**     | Jusqu'à **500 documents**              | **Illimité**                            | Illimité                           |
| **Sources RSS / web custom**     | **50 sources**, **100 articles/source** | **Illimité**                           | Illimité                           |
| **Digest**                       | Mensuel uniquement                     | **Toutes fréquences**                   | Toutes fréquences                  |
| **Chat multi-tours**             | ❌ (recherche RAG simple)              | ✅                                      | ✅                                 |
| **Alertes de veille**            | ❌                                     | ✅                                      | ✅                                 |
| **Sync cloud**                   | Toutes les **24 h** (fixe)            | Configurable, **jusqu'à toutes les heures** | Sur mesure (jusqu'à 15 min)    |
| **Équipe / organisation**        | ❌                                     | **Par siège — 19,90 €/siège, jusqu'à 10 membres** | Membres illimités        |
| **OIDC SSO**                     | ❌                                     | ❌                                      | ✅                                 |
| **Journal d'audit**              | ❌                                     | ❌                                      | ✅                                 |
| **SLA**                          | Best-effort                            | Best-effort                             | SLA sur mesure                     |
| **Tarif**                        | 0 € (vous payez votre provider IA)     | 19,90 €/mois (annuel −20 %)             | Sur devis                          |

!!! note "Pas d'essai 15 jours"
    L'ancienne période d'essai de 15 jours a été supprimée. **Free est
    l'essai** : utilisez la plateforme gratuitement, indéfiniment, avec
    votre propre clé API.

## Free — gratuit, avec votre clé (BYOK)

L'édition Free est **gratuite et sans limite de durée**. La seule
contrainte : vous apportez **votre propre clé API IA** (BYOK) parmi
Anthropic Claude, OpenAI, Google Gemini ou Mistral. Sans clé, la
recherche RAG ne peut pas produire de synthèse IA — l'indexation et la
recherche brute fonctionnent, mais la réponse synthétisée nécessite
une clé valide.

Plusieurs providers proposent un **niveau gratuit** (Gemini, Mistral),
ce qui rend Free réellement utilisable sans aucune dépense. Voir le
guide [Obtenir une clé API gratuite en 2 minutes](../usage/get-api-key.md).

Inclus dans Free :

- **Tous les connecteurs cloud** (Google Drive, OneDrive, Dropbox,
  kDrive…)
- **Bibliothèque personnelle jusqu'à 500 documents**
- **Sources RSS / web custom** : jusqu'à **50 sources**, plafonnées à
  **100 articles par source**
- **Digest mensuel**
- **Sync cloud toutes les 24 h** (fréquence fixe)
- **Recherche RAG** exécutée sur votre clé

## Pro — 19,90 €/mois, Mistral inclus

L'édition Pro (**19,90 €/mois**, ou **−20 % en facturation annuelle**)
inclut **Mistral AI** : aucune clé à gérer, la synthèse IA fonctionne
dès l'inscription. Vous pouvez toujours **apporter votre propre clé**
(BYOK) pour surcharger le provider si vous préférez Claude, GPT ou
Gemini.

Pro reprend tout Free **sans aucune limite** :

- **Documents, sources et articles illimités**
- **Toutes les fréquences de digest** (quotidien, hebdo, mensuel…)
- **Chat multi-tours** et **alertes de veille**
- **Sync cloud configurable, jusqu'à toutes les heures**
- **Organisation / équipe** facturée **par siège** : **19,90 €/siège,
  jusqu'à 10 membres**. Au-delà de 10 membres, passez à Enterprise.

## Enterprise — sur devis

L'édition Enterprise est destinée aux organisations qui veulent une
**stack dédiée et isolée** sur le cloud Myeline, avec :

- **OIDC SSO** (Azure AD, Okta, Keycloak…)
- **Membres illimités**
- **SLA sur mesure**
- **Journal d'audit** complet

!!! tip "Enterprise et on-prem se recoupent"
    Les besoins Enterprise (isolation, SSO, audit, membres illimités)
    recoupent les [éditions on-prem Souverain / Souverain-hybride](index.md).
    Si la souveraineté ou l'air-gap sont des exigences, parlons-en :
    [hello@myeline.io](mailto:hello@myeline.io).

## Comment s'inscrire

1. **Créez un compte gratuit** sur [myeline.io](https://myeline.io).
2. **Obtenez une clé API** auprès d'un provider IA (gratuit chez Gemini
   ou Mistral) — voir
   [Obtenir une clé API gratuite en 2 minutes](../usage/get-api-key.md).
3. **Collez votre clé** dans *Paramètres → Fournisseur IA*.
4. **Connectez vos drives** et ajoutez vos sources RSS / web.
5. Pour passer à **Pro**, faites-le depuis votre **tableau de bord**
   (bouton *Passer à Pro*) — Mistral est alors inclus et toutes les
   limites sont levées.

## Besoin de self-hosting ?

Si vous ne pouvez pas envoyer vos données vers un SaaS — réglementation
HDS stricte, air-gap, SecNumCloud — les
[éditions on-prem](index.md) déploient Myeline **sur votre propre
infrastructure**, en mode Souverain (air-gap) ou Souverain-hybride
(BYOK).
