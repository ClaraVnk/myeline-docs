# Bienvenue sur la documentation Myeline

**Myeline** est une plateforme d'intelligence sur vos données,
déployable directement sur **votre infrastructure**. Elle indexe RSS,
scrapers web, documents personnels et stockages cloud puis les rend
interrogeables en langage naturel via RAG. Selon l'édition,
l'embedding et la synthèse tournent **en local** (on-prem : Ollama +
bge-m3) ou via une **API externe** : sur le Cloud hébergé, l'embedding
passe par l'**API Mistral** (UE / France) — clé incluse en Pro, votre
propre clé en Free — et la synthèse utilise Mistral inclus (Pro) ou
votre clé BYOK (Free). En on-prem souverain, tout reste local.

Myeline se décline en deux grandes familles d'édition, plus une offre
sur-mesure :

- **[Community / Self-host (OSS)](community/index.md)** — l'édition
  **open source (AGPL-3.0)**, mono-utilisateur, à auto-héberger
  gratuitement (BYOK + Ollama local).
- **[Cloud / Hébergé](cloud/index.md)** — la version SaaS sur
  [myeline.io](https://myeline.io) : **Free** (votre clé) ou **Pro**
  à 7,90 €/mois (Mistral inclus, essai 15 jours), rien à installer.
- **[On-premise / Souverain](editions/index.md)** — déploiement sur
  **votre infrastructure** (air-gap ou BYOK), multi-tenant : une
  installation **sur-mesure, sur devis**.

L'usage des fonctionnalités (recherche RAG, bibliothèque, connecteurs,
conversations, veille) est **commun à toutes les éditions** et décrit
dans les [Concepts (toutes éditions)](concepts/index.md).

!!! tip "Vous utilisez la version hébergée (myeline.io) ?"
    Commencez par le [Cloud / Hébergé](cloud/index.md), les
    [Premiers pas](cloud/getting-started.md) et le guide
    [Obtenir une clé API gratuite en 2 minutes](concepts/get-api-key.md).

---

## Quelle édition est faite pour vous ?

<div class="grid cards" markdown>

-   :material-shield-lock:{ .lg .middle } **Souverain (air-gap)**

    ---

    100 % on-prem, **aucun appel API externe**. Synthèse
    locale via Ollama, mailer en log-only, connecteurs limités à
    S3 / WebDAV internes. Parfait pour les secteurs régulés
    (santé, défense, finance, secteur public).

    [:octicons-arrow-right-24: En savoir plus](editions/sovereign.md)

-   :material-server:{ .lg .middle } **Souverain-hybride (BYOK)**

    ---

    On-prem + APIs externes BYOK. Vous tournez sur votre infra
    avec vos propres clés Mistral / Anthropic / OpenAI / Gemini
    et activez les connecteurs cloud avec vos propres apps OAuth.

    [:octicons-arrow-right-24: En savoir plus](editions/sovereign-hybrid.md)

-   :material-cloud:{ .lg .middle } **Cloud (SaaS)**

    ---

    Hébergé sur myeline.io (UE / France), rien à installer. **Free**
    gratuit avec votre clé API, **Pro** à 7,90 €/mois avec Mistral
    inclus (essai 15 jours). Besoin d'on-premise ? Sur devis.

    [:octicons-arrow-right-24: En savoir plus](editions/cloud.md)

-   :fontawesome-brands-github:{ .lg .middle } **Community / Self-Host**

    ---

    Édition **open source (AGPL-3.0)**, **mono-utilisateur**, à
    auto-héberger gratuitement. Multi-LLM BYOK + **LLM 100 % local via
    Ollama**, sans licence ni organisation. Code sur GitHub.

    [:octicons-arrow-right-24: En savoir plus](editions/community.md)

</div>

---

## Premiers pas

1. **[Choisir son édition](editions/index.md)** — quel mode correspond
   à votre besoin (réglementation, budget, autonomie).
2. **[Pré-requis serveur](install/prerequisites.md)** — sizing CPU /
   RAM / disque selon le mode et le nombre d'utilisateurs.
3. **[Installation](install/index.md)** — walkthrough pas-à-pas
   guidé par notre script `install.sh`.
4. **[Première connexion admin](install/first-login.md)** —
   configurer le compte administrateur, l'organisation, vérifier
   l'état général.

---

## Ressources rapides

- :material-cog: **[Opérations day-2](operations/index.md)** : sauvegarde,
  upgrade, supervision, renouvellement de licence.
- :material-account-cog: **[Administration](admin/index.md)** : gérer
  organisations, quotas, audit log.
- :material-magnify: **[Recherche RAG côté utilisateur](concepts/rag-search.md)** :
  comment poser des questions, raffiner les résultats, conversations.
- :material-shield-check: **[Conformité RGPD](compliance/gdpr.md)** :
  registre des sous-traitants, DPA, exercice des droits.
- :material-help-circle: **[FAQ](faq.md)** : les 20 questions les plus
  fréquentes.

---

## Support

Les deux éditions on-prem incluent du support email. Les conditions
détaillées (SLA, périmètre, escalade) sont précisées dans le contrat
de licence signé avec votre organisation.

Contact : [hello@myeline.io](mailto:hello@myeline.io)
