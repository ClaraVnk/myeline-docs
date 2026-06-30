# Édition Souverain-hybride (BYOK)

L'édition **Souverain-hybride** est conçue pour les organisations
qui veulent l'isolation infrastructure du souverain pur, mais sans
sacrifier la qualité des frontier LLMs (Mistral Large, Claude
Sonnet 4.6, GPT-5, Gemini 2.5).

C'est le compromis : **vos données restent sur votre infra**, mais
**les appels de synthèse IA peuvent sortir** vers des APIs externes
**que vous configurez vous-même** (BYOK — Bring Your Own Key).

!!! info "Offre On-premise / Souverain — sur devis"
    Le déploiement souverain-hybride fait partie de l'offre
    **On-premise / Souverain**, une **installation sur-mesure, sur
    devis** — pas de souscription en self-service. Pour démarrer,
    [nous contacter](mailto:hello@myeline.io).

## Architecture

```mermaid
graph TB
    U[Utilisateurs internes] -->|HTTPS| Web[Myeline Web<br/>chez le client]
    Web --> DB[(MariaDB)]
    Web --> Redis[(Redis)]
    Web --> Chroma[(ChromaDB)]
    Web --> Ollama[Ollama local<br/>bge-m3 embedding]
    Web -.->|"BYOK Mistral<br/>Claude / OpenAI / Gemini<br/>(par org, au choix)"| LLM[Provider IA<br/>de votre choix]
    Web -.->|"Service account GCP<br/>(votre projet)"| GDrive[Google Drive]
    Web -.->|"BYOC OAuth<br/>(vos propres apps)"| OneDrive[OneDrive]
```

## Différences avec souverain pur

| Fonctionnalité           | Souverain          | Souverain-hybride          |
|--------------------------|--------------------|----------------------------|
| Ollama local             | ✅                 | ✅                         |
| API externe (Mistral, etc.) | ❌              | ✅ (BYOK par org)          |
| Connecteur Google Drive  | ❌                 | ✅ (service account GCP)   |
| Connecteurs OneDrive / Dropbox | ❌           | ✅ (BYOC — vos OAuth apps) |
| Connecteur Notion / Zotero | ❌               | ✅                         |
| Stripe                   | ❌                 | ❌                         |
| Audit log local          | ✅                 | ✅                         |
| Backup off-host (S3 / rclone) | Vers infra interne | Vers MinIO ou S3 cloud |

## BYOK — Bring Your Own Key

En souverain-hybride, **chaque organisation** au sein de
votre déploiement peut choisir indépendamment son provider LLM via
`/admin/orgs/<slug>` :

- **Local Ollama** (par défaut, gratuit)
- **Mistral AI** (votre clé)
- **Anthropic Claude** (votre clé)
- **OpenAI** (votre clé)
- **Google Gemini** (votre clé)

Les clés sont stockées **chiffrées** dans la DB
(`org.ai_api_key_enc` via Fernet, dérivée de `CLOUD_TOKEN_KEY`).
Aucune clé "platform" partagée — chaque org paie son provider en
direct.

### BYOK embedding (synapse 0025)

Depuis la migration 0025, chaque organisation peut aussi
choisir son **fournisseur d'embedding** indépendamment du LLM. Utile
quand le déploiement utilise Ollama bge-m3 par défaut mais qu'une org
veut un embedding de meilleure qualité (Voyage, Cohere multilingue) ou
une trace de facturation séparée (Mistral, OpenAI).

| Fournisseur | Modèle par défaut             | Dim     | Notes                              |
|-------------|-------------------------------|---------|------------------------------------|
| Mistral     | `mistral-embed`               | 1024    | 🇫🇷 hébergé UE, RGPD              |
| OpenAI      | `text-embedding-3-small`      | 1536    | `-3-large` 3072 dim aussi supporté |
| Voyage      | `voyage-3`                    | 1024    | Qualité top, anglais + multilingue |
| Cohere      | `embed-multilingual-v3.0`     | 1024    | Multilingue fort, 100+ langues     |

Configuration via `/org/<slug>/admin` → carte « Fournisseur d'embedding
(BYOK) ». Le routing s'applique uniquement à la collection
`org_{id}_shared` ; la collection partagée publique et les bibliothèques
personnelles des membres restent sur le fournisseur par défaut du
déploiement.

**Verrou dimensionnel.** Un changement de fournisseur sur une collection
déjà indexée est refusé tant que les vecteurs existants ne sont pas
effacés — même quand la dimension est identique (un vecteur Mistral
1024 dim et un vecteur Voyage 1024 dim vivent dans des espaces cosinus
incompatibles). Procédure :

```bash
flask wipe-org-embed-collection --org <id> --confirm
# Puis activer le nouveau fournisseur via /org/<slug>/admin
flask reindex-embeddings
```

**Sécurité.** Les clés d'embedding suivent le même schéma de stockage
que les clés LLM (`org.embed_api_key_enc` Fernet-chiffré, jamais réémis
en clair par l'UI — un masque `••••••••` confirme uniquement qu'une
clé est enregistrée).

## BYOC — Bring Your Own Credentials { #byoc }

Pour activer les connecteurs cloud (Google Drive, OneDrive, Dropbox,
kDrive) en souverain-hybride, vous devez **enregistrer vos propres
apps OAuth** chez chaque provider. Les credentials de myeline.io ne
fonctionnent que pour le SaaS central.

**Pourquoi ?** Vos utilisateurs internes accèdent à Myeline via
`https://myeline.acme.local/...`, donc le redirect URI OAuth est
`https://myeline.acme.local/user/cloud/gdrive/callback`. Google /
Microsoft / Dropbox vérifient le redirect URI contre la liste blanche
de l'app OAuth qui a initié le flow — Myeline central n'a pas
votre URL dans sa whitelist (et nous ne pouvons pas la rajouter à
l'échelle pour des raisons de sécurité et de scalabilité).

Voir le walkthrough détaillé :
[Installation souverain-hybride § BYOC](../install/sovereign-hybrid.md#byoc).

## Pour qui ?

- Entreprises qui veulent un **trade-off pragmatique** entre
  souveraineté et qualité de la synthèse IA
- Organisations qui ont déjà des contrats Anthropic / OpenAI / Mistral
  Enterprise et veulent les utiliser via Myeline
- Cabinets de conseil ou d'avocats qui hébergent eux-mêmes Myeline
  pour leurs clients et facturent une marge sur l'IA
- Universités / labos qui veulent indexer leur Drive Workspace mais
  garder la base sur leur cluster de calcul interne

## Modèle économique

- **Licence annuelle** sur devis (12 mois max)
- Vous payez **directement les providers IA** que vous activez
  (Mistral La Plateforme, Anthropic Console, OpenAI, Google AI Studio)
- Pas de marge cachée chez nous — vous voyez vos coûts IA en direct
- Support inclus — conditions détaillées dans le contrat de licence

## Démarrer

```bash
git clone -b synapse git@github.com:ClaraVnk/myeline.git
cd myeline
./scripts/install.sh
# → Choisir option 3 (Installation souverain-hybride)
# → Coller la clé de licence
# → Le wizard demande explicitement les credentials BYOC OAuth
```

Walkthrough complet : [Installation souverain-hybride](../install/sovereign-hybrid.md).
