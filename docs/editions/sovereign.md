# Édition Souveraine (air-gap)

L'édition **Souveraine** déploie Myeline **100 % sur votre
infrastructure**, sans aucun appel API externe. Conçue pour les
secteurs régulés où la résidence et l'isolation des données sont
contractuelles ou réglementaires.

!!! info "Offre On-premise / Souverain — sur devis"
    Le déploiement souverain fait partie de l'offre **On-premise /
    Souverain**, une **installation sur-mesure, sur devis**. Il n'y a
    **pas de souscription en self-service** : périmètre, modèles et
    intégration sont calibrés avec vous. Pour démarrer,
    [nous contacter](mailto:hello@myeline.io).

## Promesse air-gap

Aucun bit de donnée utilisateur ne quitte votre périmètre réseau.
Concrètement, dans cette édition :

| Composant                | Comportement air-gap                                |
|--------------------------|------------------------------------------------------|
| **Embedding**            | Ollama local (`bge-m3`) — chez vous                  |
| **Synthèse IA**          | Ollama local (modèle au choix : Mistral-Nemo, Llama 3.1, Mixtral, etc.) |
| **Mistral / Anthropic / OpenAI / Gemini** | **Bloqués** — même si une clé API traîne dans `.env` |
| **Mailer (Brevo)**       | Forcé en log-only — emails écrits dans `logs/mailer/` |
| **Stripe**               | Routes `/payment/*` retournent 404, webhook 404      |
| **Connecteurs cloud**    | **Seuls S3 (MinIO interne) et WebDAV (Nextcloud interne) sont disponibles**. Public-cloud (GDrive, OneDrive, Dropbox, Notion, Zotero, kDrive) bloqués. |
| **Connexion sociale**    | `/auth/social/*` bloqués (Google / Microsoft). Connexion par email + mot de passe interne. |
| **Validation de licence**| Ed25519 hors-ligne, signée — aucune connexion réseau requise |
| **CDN frontend**         | Assets bundlés localement (`/static/vendor/`) — pas de jsdelivr / Google Fonts |
| **Google Analytics**     | Hard-désactivé                                        |

## Pour qui ?

- Secteur santé soumis à **HDS** strict (au-delà de "héberger chez un HDS")
- Secteur **défense / DGA** (zones secret défense, IGI 1300)
- **OIV / OIS** (NIS2, LPM)
- **Banques** sous régulation BCE / ACPR avec exigences de
  cloisonnement
- Recherche sensible (CEA, ONERA…)
- Cabinets juridiques avec contrats de confidentialité absolus
- Organismes publics ayant une obligation **SecNumCloud**

## Pré-requis

Voir [Pré-requis serveur](../install/prerequisites.md). En résumé :

- **CPU only** : 16 vCores, 32 GB RAM, 200 GB NVMe (≤ 20 utilisateurs internes).
  Les requêtes LLM prennent 15-40 s en synthèse locale CPU.
- **Avec GPU** : RTX 4090 24 GB ou L40S — recommandé dès 1 utilisateur
  pour une expérience interactive.
- **Bare-metal + 2× GPU** pour Llama 3.1 70B ou Mixtral 8×7B.

## Modèle économique

- **Licence annuelle** sur devis (renouvelée chaque année)
- Pas de Stripe, pas d'abonnement self-serve
- Le périmètre fonctionnel inclut **toutes les fonctions Pro+
  automatiquement** (multi-turn, conversations, alertes, query
  history…) — vu que la licence couvre l'opérateur, il n'y a pas de
  notion de "plan utilisateur" interne
- Support inclus — conditions détaillées dans le contrat de licence

## Limitations connues à connaître

- **Pas d'API externe** = pas de Mistral cloud / Claude / OpenAI /
  Gemini → vous êtes limité à la qualité du modèle local que vous
  hébergez. Mistral-Nemo (Q4) est correct pour 90 % des cas, mais
  pour des tâches complexes vous voudrez un Llama 3.1 70B (GPU
  obligatoire).
- **Pas de Brevo** = pas d'emails transactionnels par défaut. Si vous
  voulez de vrais emails, configurez un MTA / SMTP interne et
  pointez Myeline vers lui (chantier custom à discuter).
- **Pas de connecteurs cloud publics** = vos utilisateurs ne peuvent
  pas indexer leur Google Drive perso. Seul un MinIO interne ou un
  Nextcloud interne fait sens en air-gap.
- **Mises à jour manuelles** : vous décidez quand pull la nouvelle
  image. Aucune mise à jour automatique.
- **Révocation de licence** : impossible instantanément en air-gap.
  La défense pratique = expirations courtes (≤ 12 mois). Voir
  [renouvellement de licence](../operations/license-renewal.md).

## Démarrer

```bash
# 1. Recevoir votre clé de licence par email après signature du devis.
# 2. Sur votre serveur :

git clone -b synapse git@github.com:ClaraVnk/myeline.git
cd myeline
./scripts/install.sh
# → Choisir option 2 (Installation souveraine)
# → Coller la clé de licence reçue
# → Répondre aux questions guidées (Ollama URL, modèle, etc.)
```

Pour le walkthrough complet, voir
[Installation souveraine](../install/sovereign.md).
