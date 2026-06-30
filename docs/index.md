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

Myeline existe en deux familles : l'offre **Cloud** hébergée sur
[myeline.io](https://myeline.io) (Free / Pro, rien à installer) et
l'offre **On-premise / Souverain** livrée clé en main aux
organisations — une installation **sur-mesure, sur devis** (air-gap ou
BYOK).

!!! tip "Vous utilisez la version hébergée (myeline.io) ?"
    Cette doc est surtout orientée on-prem. Pour le SaaS, commencez
    par l'[édition Cloud (Free / Pro)](editions/cloud.md)
    et le guide [Obtenir une clé API gratuite en 2 minutes](usage/get-api-key.md).

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
- :material-magnify: **[Recherche RAG côté utilisateur](usage/rag-search.md)** :
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
