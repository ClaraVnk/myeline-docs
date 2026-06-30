# FAQ

Les questions les plus fréquentes sur les éditions on-prem de
Myeline.

## Général

### Quelle différence entre souverain et souverain-hybride ?

- **Souverain** : 100 % air-gap, aucun appel sortant possible. La
  synthèse IA tourne en local via Ollama.
- **Souverain-hybride** : air-gap par défaut, mais chaque
  organisation peut activer un provider IA externe (Mistral / Claude
  / OpenAI / Gemini) avec **sa propre clé** (BYOK).

Détail : [Choisir son édition](editions/index.md).

### Puis-je migrer entre les deux ?

Oui, sans perte de données. Changer `DEPLOYMENT_MODE` dans `.env`,
fournir la nouvelle clé de licence du tier voulu, redémarrer.

### Combien coûte la licence ?

Sur devis, en fonction du nombre d'utilisateurs et du périmètre.
Contactez [hello@myeline.io](mailto:hello@myeline.io). Les licences
sont **annuelles**, max 12 mois (impossible d'émettre une licence
perpétuelle, par construction).

### Y a-t-il un mode « essai » ?

Oui — licence de 30 jours pour évaluation, à demander sur le même
canal. Couvre toutes les fonctionnalités du tier hybride.

## Tarifs (offre hébergée)

### Combien coûte l'offre hébergée sur myeline.io ?

- **Free** : **gratuit**, sans limite de durée — vous branchez votre
  **propre clé IA** (BYOK, Mistral / OpenAI / Claude / Gemini).
- **Pro** : **7,90 €/mois**, **facturation mensuelle**, avec **15 jours
  d'essai gratuit**. Mistral inclus (aucune clé à gérer) et toutes les
  limites levées.
- **Équipes / organisations** : facturées **par siège**, soit
  **7,90 €/membre**.
- **On-premise / Souverain** : installation **sur-mesure, sur devis** —
  [nous contacter](mailto:hello@myeline.io).

Détail : [Édition Cloud](editions/cloud.md).

### L'essai Pro est-il vraiment gratuit ?

Oui : **15 jours**, aucun débit avant J+15, annulable à tout moment
(carte requise pour démarrer l'essai). À la fin de l'essai, vous passez
en Pro mensuel à 7,90 €/mois, ou vous revenez en Free.

### Y a-t-il une offre annuelle ?

Non. L'offre **Pro est mensuelle uniquement** (7,90 €/mois). Pour un
besoin on-premise, le tarif est **sur devis** — parlons-en :
[hello@myeline.io](mailto:hello@myeline.io).

## Données

### Mes données quittent-elles mon réseau ?

- **Souverain** : non, jamais. Aucun appel sortant n'est techniquement
  possible.
- **Souverain-hybride** : seulement si vous activez explicitement un
  provider IA externe. Dans ce cas, le texte de la question + les
  chunks pertinents transitent vers le provider à chaque requête.
  Documents et embeddings restent toujours sur votre infra.

Détail : [Localisation des données](compliance/data-residency.md).

### Quelle taille de base puis-je gérer ?

Ordre de grandeur sur des sizings standards :

| Volume                | Setup recommandé              |
|-----------------------|-------------------------------|
| ≤ 100 K documents     | 32 GB RAM, NVMe, GPU optionnel |
| 100 K - 1 M documents | 64 GB RAM, GPU 24 GB, NVMe stripé |
| > 1 M documents       | Sharding ChromaDB + cluster Ollama dédié |

Voir [Pré-requis serveur](install/prerequisites.md).

### Est-ce conforme RGPD / HDS ?

- **RGPD** : oui, par construction (chiffrement, pseudonymisation,
  audit, durée de rétention configurable, droit à l'effacement).
- **HDS** : Myeline n'est pas en soi certifié HDS — la certification
  porte sur l'hébergeur. En souverain, l'hébergeur c'est vous : si
  votre infra est HDS-certifiée, votre déploiement Myeline l'est.

Détail : [Conformité RGPD](compliance/gdpr.md).

## Fonctionnalités

### Puis-je utiliser GPT-4 ou Claude ?

- **Souverain** : non.
- **Souverain-hybride** : oui, via BYOK. Chaque organisation choisit
  son provider et fournit sa propre clé API. Aucun coût d'IA via
  Myeline — vous payez le provider en direct.

### Quels formats de documents sont supportés ?

PDF, DOCX, ODT, TXT, MD, HTML, CSV. Voir
[Bibliothèque et upload](concepts/library.md).

L'OCR sur PDF scannés est désactivé par défaut — activable via
`RAG_OCR_ENABLED=true` (souverain-hybride uniquement, dépendance
Tesseract).

### Y a-t-il un upload manuel de fichier ?

**Non, pas du tout** — ni côté personnel, ni côté organisation. La
bibliothèque (perso ou partagée) se remplit exclusivement via :

- **Connecteurs cloud** : Google Drive, OneDrive, Dropbox, kDrive,
  Zotero, S3 / S3-compatible, WebDAV
- **Scrapers RSS / web**

C'est un choix de design (source unique de vérité chez le client,
conformité, simplicité d'ingestion) — voir
[Bibliothèque § Pourquoi pas d'upload manuel ?](concepts/library.md#pourquoi-pas-dupload-manuel).

### Combien d'utilisateurs simultanés ?

Sans goulot d'étranglement matériel : **plusieurs centaines** sur
une infra correctement dimensionnée. Le bottleneck typique est
Ollama (synthèse IA) sur un seul GPU — au-delà de ~20-30 utilisateurs
actifs simultanés, prévoir un cluster GPU ou bascule BYOK
souverain-hybride.

### Le multi-tenant fonctionne-t-il en on-prem ?

Oui. Une instance Myeline peut héberger **plusieurs organisations**,
chacune avec sa propre collection ChromaDB, ses utilisateurs et son
provider IA (en souverain-hybride). Les organisations sont
strictement isolées.

## Opérations

### À quelle fréquence dois-je mettre à jour ?

- **Patches sécurité** : sous 7 jours après publication.
- **Minor releases** : selon votre fenêtre de maintenance.
- **Major releases** (rares) : 30 jours de préavis, possibilité de
  suivre des release candidates en environnement de pré-production.

Voir [Mise à jour](operations/upgrade.md).

### Comment sauvegarder ?

Cron quotidien automatique à 02:30 (DB + ChromaDB + uploads). Off-host
recommandé via rclone / MinIO / borg. Voir
[Sauvegarde et restauration](operations/backup-restore.md).

### Y a-t-il une API publique ?

Pas d'API REST exposée formellement aux utilisateurs finaux dans
les éditions actuelles. L'admin a accès aux endpoints internes
(`/admin/*`, `/metrics`, `/healthz`), et les blueprints Flask sont
introspectables si vous voulez scripter en interne.

Une API publique (REST + OpenAPI) est sur la roadmap pour Q4 2026.

## Support

### Quel est le SLA ?

Le SLA et les conditions détaillées de support sont précisés dans
le contrat de licence signé avec votre organisation.

### Où poser une question ?

- Email : [hello@myeline.io](mailto:hello@myeline.io)
- Pour les bugs reproductibles : joindre une trace `/health` + logs
  `web` des 50 dernières lignes.

### Puis-je accéder au code source ?

Oui en souverain-hybride sous escrow et sur demande spécifique
(clauses commerciales). Le code de Myeline n'est pas open-source
public mais l'éditeur peut fournir un dépôt en lecture pour audit
sécurité côté client.
