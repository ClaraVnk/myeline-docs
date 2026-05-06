# Choisir son édition

Deux éditions on-prem de Myeline, chacune calibrée pour un besoin
spécifique. Choisissez selon **trois critères** : la réglementation
qui s'applique à vos données, votre besoin d'autonomie, et
l'infrastructure dont vous disposez.

## Tableau de comparaison

| Critère                        | Souverain          | Souverain-hybride          |
|--------------------------------|--------------------|----------------------------|
| **Hébergement**                | Votre infra        | Votre infra                |
| **Connexion Internet**         | Aucune (air-gap)   | Sortante autorisée (BYOK)  |
| **Synthèse IA**                | Ollama local uniquement | Au choix par org : Ollama OU Mistral / Claude / OpenAI / Gemini |
| **Embedding**                  | Ollama local       | Ollama local               |
| **Connecteurs cloud**          | S3 + WebDAV uniquement (vers infra interne) | Tous (avec **vos** OAuth apps — voir BYOC) |
| **Connexion sociale (Google/MS/Apple login)** | ❌ | ❌ (utiliser OIDC entreprise à la place) |
| **OIDC SSO entreprise**        | Inclus             | Inclus                     |
| **Stripe (abonnements)**       | ❌                 | ❌                          |
| **Multi-tenant**               | Mono-tenant par défaut | Mono-tenant par défaut |
| **Mises à jour**               | Manuelles (`podman pull`) | Manuelles (`podman pull`) |
| **Tarif**                      | Sur devis (licence annuelle) | Sur devis (licence annuelle) |

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

**Implications** : pour activer les connecteurs cloud (GDrive,
OneDrive, Dropbox), vous devez créer **vos propres apps OAuth**
chez chaque provider — voir le walkthrough [BYOC](sovereign-hybrid.md#byoc).

[En savoir plus →](sovereign-hybrid.md){ .md-button .md-button--primary }

## Migration entre éditions

Passer de Souverain à Souverain-hybride (ou l'inverse) est possible
**sans perte de données** :

- **Souverain → Souverain-hybride** : changer la valeur de
  `DEPLOYMENT_MODE` dans `.env`, fournir une nouvelle clé de licence
  du tier hybrid, redémarrer. Aucune migration de données.
- **Souverain-hybride → Souverain** : symétrique.

Les changements d'édition se font à l'occasion d'un renouvellement
de licence (max 12 mois). Contactez-nous pour planifier.
