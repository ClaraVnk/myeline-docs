# Bienvenue sur la documentation Myeline

**Myeline** est une plateforme d'intelligence sur vos données,
déployable directement sur **votre infrastructure**. Elle indexe RSS,
scrapers web, documents personnels et stockages cloud puis les rend
interrogeables en langage naturel via RAG. L'embedding est **toujours
local** (Ollama + bge-m3) ; la synthèse utilise un LLM local (Ollama)
ou une API externe (Mistral / Anthropic / OpenAI / Gemini) selon
votre édition.

Cette documentation couvre les **deux éditions on-prem** que nous
livrons clé en main aux organisations : **Souverain** (air-gap) et
**Souverain-hybride** (BYOK).

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
  organisations, OIDC SSO, quotas, audit log.
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
