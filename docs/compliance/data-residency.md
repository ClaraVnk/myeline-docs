# Localisation des données

Sujet central pour les organismes soumis à des contraintes de
souveraineté (HDS, OIV, secteur public, défense).

## Vue d'ensemble par édition

| Composant                 | Souverain (air-gap)            | Souverain-hybride (BYOK)                    |
|---------------------------|--------------------------------|---------------------------------------------|
| Application Web           | Votre infra                    | Votre infra                                 |
| Base de données           | Votre infra                    | Votre infra                                 |
| ChromaDB (vecteurs)       | Votre infra                    | Votre infra                                 |
| Uploads / fichiers        | Votre infra                    | Votre infra                                 |
| Embedding (bge-m3)        | Votre infra (Ollama local)     | Votre infra                                 |
| Synthèse LLM              | Votre infra (Ollama local)     | Au choix : Ollama local OU API externe BYOK |
| Mailer                    | Log-only (votre infra)         | Brevo (UE) ou MTA interne                   |
| Audit log archive         | MinIO interne (votre infra)    | MinIO interne ou S3 cloud (votre choix)     |
| Backups                   | Votre infra + off-host de votre choix | Idem                                  |
| OAuth credentials         | n/a (connecteurs publics désactivés) | Vos providers (US/UE selon)           |

## Souverain (air-gap) — verrouillage technique

En édition souverain :

- **Aucun appel API externe n'est possible** par construction.
  Les routes `/payment/*`, `/auth/social/*` renvoient 404. Les
  connecteurs cloud publics (GDrive, OneDrive…) sont masqués et
  leurs routes désactivées.
- **Le mailer Brevo est forcé en log-only** — même si une clé API
  est présente dans `.env`.
- **Les assets frontend sont bundlés localement** (pas de CDN
  jsdelivr ni Google Fonts) — voir `app/static/vendor/`.
- **Validation de licence offline** par signature Ed25519 (pas
  d'appel à un serveur de licence).
- **Aucune télémétrie** (Google Analytics et compagnie hard-désactivés).

Vérification : `curl -i http://localhost:5000/auth/social/google`
doit renvoyer 404 ; `python -m app.utils.deployment_mode` doit
afficher `mode=sovereign`.

## Souverain-hybride — points sortants identifiés

En édition souverain-hybride, **chaque appel sortant est explicite
et documenté**. La liste exhaustive selon la configuration :

| Destination                 | Quand                                | Volume typique         |
|-----------------------------|--------------------------------------|------------------------|
| Mistral / Anthropic / OpenAI / Gemini API | À chaque requête RAG si BYOK activé | 1-5 KB par question  |
| Brevo SMTP                  | Email transactionnel (signup, reset, invitation) | 1 KB par email         |
| Google / Microsoft / Dropbox APIs | Sync cloud connectors            | Variable (listing + download) |
| OIDC IdP                    | Login SSO (SSO sur demande, plus en self-service) | 2-3 KB par auth        |

**Aucun appel sortant** :

- Vers un serveur de licence Myeline (validation offline)
- Vers une CDN frontend
- Vers un service de télémétrie

## Recommandations infra

### Souverain pur

- VLAN dédié, sans route Internet (ou DROP par défaut sur le
  firewall de sortie).
- DNS interne uniquement.
- NTP synchro sur un serveur interne (pour la validation des
  signatures de licence — l'horloge doit rester proche de la
  réalité ±1 jour, sinon la licence sera rejetée comme expirée
  ou pas-encore-valide).

### Souverain-hybride

- Egress filtering : whitelist explicite des domaines autorisés
  (api.mistral.ai, api.anthropic.com, api.openai.com,
  generativelanguage.googleapis.com, api.brevo.com, accounts.google.com,
  login.microsoftonline.com, api.dropbox.com…). Tout le reste DROP.
- Logging des connexions sortantes (pour audit).
- Reverse proxy (Pangolin / Traefik / Nginx) en front, TLS ≥ 1.2,
  HSTS, CSP stricte.

## Données personnelles vs métadonnées techniques

Myeline distingue trois niveaux de criticité :

1. **Données utilisateur** (documents, requêtes, conversations) —
   restent toujours sur votre infra.
2. **Identifiants techniques** (user_id, org_id, action_code dans
   l'audit) — restent sur votre infra.
3. **Métriques anonymes** (uptime, latence, taux d'erreur) —
   exposées via `/metrics`, scrapables par votre Prometheus
   uniquement (pas de push externe).

Aucun **email**, aucun **contenu de requête**, aucun **document
indexé** ne quitte votre infra — sauf à activer explicitement un
provider IA externe en souverain-hybride (et dans ce cas, seul le
texte de la question + les chunks pertinents transitent vers le
provider).
