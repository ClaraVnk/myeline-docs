# Community / Self-host (open source)

L'édition **Community** est le cœur **open source** de Myeline, publié
sous licence **AGPL-3.0** sur GitHub. Vous l'hébergez **sur votre
propre serveur**, gratuitement : pas de licence à acheter, pas de
compte à créer, aucun appel vers nos serveurs. C'est l'édition
**mono-utilisateur** pour qui veut sa propre base de connaissances RAG,
souveraine et inspectable.

[:fontawesome-brands-github: Le code sur GitHub](https://github.com/ClaraVnk/myeline){ .md-button .md-button--primary }
[:material-book-open-variant: Guide self-host](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md){ .md-button }

!!! info "Cette page, le wiki, et le produit"
    Le produit Community est **anglophone** (UI et documentation du
    dépôt en anglais). Ce wiki reste **bilingue FR / EN**. Pour
    l'**usage détaillé** des fonctionnalités (recherche RAG,
    bibliothèque, connecteurs, conversations, veille), reportez-vous
    aux [Concepts (toutes éditions)](../concepts/index.md) : ils sont
    communs à toutes les éditions.

## Pour qui ?

- **Développeurs et self-hosters** qui veulent un RAG personnel
  auto-hébergé, modifiable, sans dépendance à un SaaS.
- **Souveraineté des données personnelles** : documents, index
  vectoriel et historique restent sur votre machine. En mode LLM local
  (Ollama), **aucune donnée ne quitte l'hôte**.
- **Évaluation technique** avant un déploiement on-premise plus large.

## Ce qu'elle apporte

- **RAG complet** : recherche top-K avec diversification MMR, citations
  cliquables, mode strict/souple, filtrage temporel, conversations
  multi-tours, chat sur un document unique.
- **Connecteurs de stockage cloud** (Google Drive, OneDrive, Dropbox,
  kDrive, S3, WebDAV, Notion, Zotero) avec **vos propres identifiants**
  (BYOC).
- **Sources RSS / web** scrapées et indexées.
- **Multi-LLM BYOK + LLM 100 % local via Ollama** : Mistral, OpenAI,
  Claude, Gemini avec votre clé, ou Ollama (`bge-m3` + synthèse)
  totalement hors-ligne.
- **Digests et alertes de veille**, **2FA TOTP**.

Le détail fonctionnel complet vit dans la page d'édition :
[Édition Community / Self-Host](../editions/community.md).

## Démarrer en 3 commandes

Pré-requis : Podman 4.6+ et `podman-compose` (ou Docker + Compose).

```bash
git clone https://github.com/ClaraVnk/myeline.git ~/Projects/myeline
cd ~/Projects/myeline
cp .env.example .env        # puis éditer — voir docs/SELF_HOSTING.md

podman-compose -f docker-compose.yml up -d --build
podman-compose -f docker-compose.yml exec web flask create-admin
```

Puis ouvrez <http://localhost:5000>. Le walkthrough complet
(pré-requis, génération des clés, choix du LLM, apps OAuth par
connecteur) est dans le
**[guide self-host](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md)**.

## Et après ?

- **Comprendre l'usage** : [Concepts (toutes éditions)](../concepts/index.md).
- **Comparer les éditions** : [Choisir son édition](../editions/index.md).
- **Besoin de multi-tenant, d'équipes ou de support contractuel ?**
  Voir le [Cloud (SaaS)](../cloud/index.md) ou l'offre
  [On-premise / Souverain](../editions/sovereign.md) (sur devis).

## Licence

Édition self-host sous **GNU Affero General Public License v3.0**. La
clause réseau de l'AGPL impose, si vous exposez un Myeline **modifié**
comme service réseau, de mettre la source modifiée à disposition de ses
utilisateurs. Une **licence commerciale** (usage sans les obligations
AGPL) est disponible : [hello@myeline.io](mailto:hello@myeline.io).
