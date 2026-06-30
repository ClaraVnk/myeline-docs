# Choisir son édition

Myeline existe en **trois familles**, selon que vous voulez héberger
vous-même gratuitement, utiliser la version hébergée sans rien
installer, ou déployer une installation sur-mesure sur votre propre
infrastructure.

<div class="grid cards" markdown>

-   :fontawesome-brands-github:{ .lg .middle } **Community / Self-host (OSS)**

    ---

    Édition **open source (AGPL-3.0)**, **mono-utilisateur**, à
    auto-héberger gratuitement. Multi-LLM BYOK **+ LLM 100 % local via
    Ollama**, sans licence ni organisation.

    [:octicons-arrow-right-24: Community / Self-host](../community/index.md)

-   :material-cloud:{ .lg .middle } **Cloud / Hébergé**

    ---

    Hébergé sur [myeline.io](https://myeline.io) (UE / France), rien à
    installer. **Free** gratuit (votre clé), **Pro** à 7,90 €/mois
    Mistral inclus (essai 15 jours), équipes par siège.

    [:octicons-arrow-right-24: Cloud / Hébergé](../cloud/index.md)

-   :material-shield-lock:{ .lg .middle } **On-premise / Souverain (sur devis)**

    ---

    Déploiement sur **votre infrastructure**, multi-tenant, avec
    support contractuel. Variantes **Souverain (air-gap)** et
    **Souverain-hybride (BYOK)**. Installation **sur-mesure, sur devis**.

    [:octicons-arrow-right-24: Voir les variantes](sovereign.md)

</div>

Les deux **éditions on-prem** ci-dessous (Souverain et
Souverain-hybride) sont des installations **sur-mesure, sur devis**,
chacune calibrée pour un besoin spécifique. Choisissez selon **trois
critères** : la réglementation qui s'applique à vos données, votre
besoin d'autonomie, et l'infrastructure dont vous disposez.

## Tableau de comparaison

| Critère                        | Souverain          | Souverain-hybride          |
|--------------------------------|--------------------|----------------------------|
| **Hébergement**                | Votre infra        | Votre infra                |
| **Connexion Internet**         | Aucune (air-gap)   | Sortante autorisée (BYOK)  |
| **Synthèse IA**                | Ollama local uniquement | Au choix par org : Ollama OU Mistral / Claude / OpenAI / Gemini |
| **Embedding**                  | Ollama local       | Ollama local               |
| **Connecteurs cloud**          | S3 + WebDAV uniquement (vers infra interne) | Tous (avec **vos** OAuth apps — voir BYOC) |
| **Connexion sociale (Google / Microsoft)** | ❌ | ❌ |
| **Stripe (abonnements)**       | ❌                 | ❌                          |
| **Multi-tenant**               | Mono-tenant par défaut | Mono-tenant par défaut |
| **Mises à jour**               | Manuelles (`podman pull`) | Manuelles (`podman pull`) |
| **Tarif**                      | Sur devis — nous contacter | Sur devis — nous contacter |

## Quand choisir laquelle ?

### Souverain (air-gap)

**Pour qui** : organismes publics, opérateurs d'importance vitale,
secteur défense, secteur santé HDS, banques régulées, toute
organisation soumise à un audit de localisation des données strict.

**Avantages** : aucun bit ne sort de votre réseau. Tous les composants
(synthèse IA, mailer, monitoring) tournent en local. Conforme
SecNumCloud par construction.

**À évaluer** : l'absence d'API externe oblige à utiliser des modèles
LLM locaux (Mistral-Nemo, Llama 3.1, Mixtral…) qui demandent
des serveurs avec GPU pour des performances acceptables. Voir
[pré-requis serveur](../install/prerequisites.md).

[En savoir plus →](sovereign.md){ .md-button .md-button--primary }

### Souverain-hybride (BYOK)

**Pour qui** : entreprises qui veulent l'isolation infra du souverain
mais avec la qualité des frontier LLMs (Mistral Large, Claude
Sonnet 4.6, GPT-5, Gemini 2.5).

**Avantages** : l'infrastructure et les données restent chez vous,
mais la synthèse peut être routée par organisation vers le LLM de
votre choix avec **votre propre clé API** (BYOK). Vous payez
directement le provider IA, pas un intermédiaire.

**Implications** : pour activer les connecteurs cloud, vous configurez
les credentials chez chaque provider depuis votre tenant :
- **Google Drive** : créez un service account GCP dans votre projet
- **OneDrive / Dropbox / Notion** : créez vos propres apps OAuth (BYOC)

Voir le walkthrough complet :
[Installation souverain-hybride § Connecteurs cloud](../install/sovereign-hybrid.md#byoc).

[En savoir plus →](sovereign-hybrid.md){ .md-button .md-button--primary }

## Et l'édition Cloud ?

Si vous n'avez pas de contrainte de self-hosting, l'[édition Cloud
(SaaS)](cloud.md) est le chemin le plus rapide : compte gratuit,
connecteurs cloud, recherche RAG, sans serveur à gérer (**Free** ou
**Pro à 7,90 €/mois**). Si la souveraineté ou l'air-gap sont des
exigences, l'offre **On-premise / Souverain** est une installation
**sur-mesure, sur devis** — parlons-en :
[hello@myeline.io](mailto:hello@myeline.io).

## Migration entre éditions

Passer de Souverain à Souverain-hybride (ou l'inverse) est possible
**sans perte de données** :

- **Souverain → Souverain-hybride** : changer la valeur de
  `DEPLOYMENT_MODE` dans `.env`, fournir une nouvelle clé de licence
  du tier hybrid, redémarrer. Aucune migration de données.
- **Souverain-hybride → Souverain** : symétrique.

Les changements d'édition se font à l'occasion d'un renouvellement
de licence (max 12 mois). Contactez-nous pour planifier.
