# Édition Community / Self-Host (open source)

L'édition **Community** est la version **open source** de Myeline,
publiée sous licence **AGPL-3.0** sur GitHub. Elle tourne **entièrement
sur votre propre infrastructure**, elle est **mono-utilisateur**, et
elle ne dépend d'**aucun service Myeline** : pas de licence à acheter,
pas de compte à créer, pas d'appel vers nos serveurs.

C'est le cœur open-core de Myeline. Si vous voulez juste héberger votre
propre base de connaissances RAG sur votre serveur, sans organisation,
sans facturation et sans dépendance externe, c'est cette édition.

[:fontawesome-brands-github: Le code sur GitHub](https://github.com/ClaraVnk/myeline){ .md-button .md-button--primary }
[:material-book-open-variant: Guide self-host](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md){ .md-button }

!!! info "Community vs éditions hébergées / enterprise"
    Cette page couvre l'édition **open source, mono-utilisateur**. Les
    fonctionnalités multi-tenant (organisations et équipes, SSO OIDC,
    facturation, déploiement enterprise) ne font **pas** partie de ce
    projet — elles relèvent de l'[édition Cloud (SaaS)](cloud.md) et des
    [éditions on-prem Souverain / Souverain-hybride](sovereign-hybrid.md).

## Pour qui ?

- **Développeurs et bidouilleurs** qui veulent un RAG personnel auto-hébergé,
  inspectable et modifiable (le code est ouvert, AGPL).
- **Self-hosters** qui font déjà tourner leur propre stack (Nextcloud,
  Paperless, etc.) et veulent y ajouter une couche d'intelligence sur
  leurs documents.
- **Souveraineté des données personnelles** : tout reste sur votre
  serveur — documents, index vectoriel, historique de conversation. En
  mode LLM local (Ollama), **aucune donnée ne quitte la machine**.

## Fonctionnalités

- **Connecteurs de stockage cloud** — synchronisation et indexation
  automatiques depuis Google Drive, OneDrive, Dropbox, kDrive
  (Infomaniak), S3 / compatible S3 (Scaleway, OVH, R2, MinIO…), WebDAV,
  Notion et Zotero. Chaque connecteur utilise **vos propres
  identifiants** (voir [BYOC](#byoc-vos-propres-apps-oauth)).
- **Sources RSS / web** — pointez Myeline vers un flux ou un site ;
  détection auto RSS vs HTML, scraping du contenu (`trafilatura` +
  parcours de sitemap), extraction des dates de publication.
- **Bibliothèque personnelle** — import de documents (PDF / DOCX / ODF),
  parsés et indexés avec le reste.
- **Recherche RAG** — récupération top-K avec diversification MMR,
  citations cliquables `[N]`, surlignage des mots-clés, bascule
  strict/souple, filtrage temporel, indicateur de fraîcheur des sources,
  conversations multi-tours et chat sur un document unique.
- **Outillage de requête** — historique automatique, requêtes
  sauvegardées, retour 👍/👎 sur les réponses.
- **Digests et alertes de veille** — veilles par mots-clés + périmètre,
  exécutées sur planning, avec digest email des nouvelles sources.
- **Authentification à deux facteurs** — 2FA TOTP sur le compte.

## Multi-LLM (BYOK) et LLM 100 % local

L'édition Community est **bring-your-own-key**. Vous choisissez le
fournisseur de synthèse et collez **votre propre clé API** dans
*Paramètres → Fournisseur IA* ; la clé est stockée **chiffrée au repos**.

| Fournisseur        | Synthèse | Embedding (indexation) |
|--------------------|----------|------------------------|
| Mistral            | ✅       | ✅                     |
| OpenAI             | ✅       | ✅                     |
| Anthropic Claude   | ✅       | —                      |
| Google Gemini      | ✅       | —                      |
| Voyage / Cohere    | —        | ✅                     |

L'indexation nécessite une clé capable de faire de l'embedding (Mistral
ou OpenAI couvrent synthèse **et** embedding ; Claude/Gemini répondent
mais n'indexent pas).

!!! tip "LLM self-host via Ollama — zéro tiers"
    Pour une installation **totalement autonome**, faites tourner le
    modèle vous-même. Le service **[Ollama](https://ollama.com)** est
    fourni dans le `docker-compose` : pointez `OLLAMA_URL` dessus et
    Myeline l'utilise à la fois pour l'embedding (`bge-m3`) et la
    synthèse. **Aucune clé API, aucune donnée ne quitte l'hôte.** C'est
    la configuration recommandée quand la résidence des données ou
    l'air-gap comptent.

    N'importe quel endpoint **compatible OpenAI** (vLLM, LM Studio,
    serveur llama.cpp) fonctionne de la même manière en pointant le
    provider OpenAI vers votre URL locale.

## BYOC — vos propres apps OAuth { #byoc-vos-propres-apps-oauth }

Chaque connecteur cloud s'authentifie avec **vos identifiants**, pas
ceux de Myeline. Pour les connecteurs OAuth (Google Drive, OneDrive,
Dropbox, kDrive), vous enregistrez **une app par fournisseur**, une
seule fois ; le redirect URI est
`https://<votre-domaine>/user/cloud/<provider>/callback`. Les
connecteurs à jeton (Notion, S3, WebDAV, Zotero) ne demandent qu'un
token ou une clé d'accès saisie dans l'UI.

Détails complets, scopes et liens consoles dans le
[guide self-host (§ Bring-your-own credentials)](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md#5-bring-your-own-credentials-connectors).

## Ce qui la distingue des autres éditions

| Critère                         | Community (Self-Host)        | Cloud (SaaS)                 | Souverain / Souverain-hybride |
|---------------------------------|------------------------------|------------------------------|-------------------------------|
| **Licence**                     | Open source **AGPL-3.0**     | Propriétaire (SaaS)          | Propriétaire (licence annuelle) |
| **Coût**                        | Gratuit                      | Free / 19,90 €/mois / devis  | Sur devis                     |
| **Hébergement**                 | Votre infra                  | Cloud Myeline (UE)           | Votre infra                   |
| **Utilisateurs**                | **Mono-utilisateur**         | Multi (par siège / org)      | Multi-tenant enterprise       |
| **Synthèse IA**                 | BYOK **+ Ollama local**      | Mistral inclus / BYOK        | Ollama local et/ou BYOK       |
| **Connecteurs cloud**           | Tous (vos apps OAuth)        | Tous                         | Tous (BYOC) / S3+WebDAV (air-gap) |
| **Organisations & équipes**     | ❌                           | ✅ (Pro / Enterprise)        | ✅                            |
| **SSO OIDC entreprise**         | ❌                           | ✅ (Enterprise)              | ✅                            |
| **Facturation / Stripe**        | ❌                           | ✅                           | ❌                            |
| **Support**                     | Communautaire (best-effort)  | Selon formule                | Inclus au contrat             |

En résumé : l'édition Community est le **socle open source mono-utilisateur**.
Les organisations, le SSO, la facturation et le multi-tenant sont
réservés aux éditions [Cloud](cloud.md) et
[Souverain / Souverain-hybride](sovereign-hybrid.md).

## Démarrer

Pré-requis : Podman 4.6+ et `podman-compose` (ou Docker + Compose).

```bash
git clone https://github.com/ClaraVnk/myeline.git ~/Projects/myeline
cd ~/Projects/myeline
cp .env.example .env        # puis éditer — voir docs/SELF_HOSTING.md

# Build et démarrage de la stack (web, worker, cron, MariaDB, Redis, Ollama)
podman-compose -f docker-compose.yml up -d --build

# Création du compte (premier démarrage uniquement)
podman-compose -f docker-compose.yml exec web flask create-admin
```

Puis ouvrez <http://localhost:5000>.

Le walkthrough complet (pré-requis, génération des clés, choix du LLM,
enregistrement des apps OAuth par connecteur) vit dans le
**[guide self-host](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md)**
du dépôt.

## Licence

L'édition self-host est sous **GNU Affero General Public License v3.0**.
La clause d'usage réseau de l'AGPL impose que, si vous proposez un
Myeline **modifié** comme service réseau, vous mettiez la source
modifiée à disposition de ses utilisateurs. Une **licence commerciale**
(usage sans les obligations de l'AGPL) est disponible —
contact : [hello@myeline.io](mailto:hello@myeline.io).
