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
[Bibliothèque et upload](usage/upload-documents.md).

L'OCR sur PDF scannés est désactivé par défaut — activable via
`RAG_OCR_ENABLED=true` (souverain-hybride uniquement, dépendance
Tesseract).

### Y a-t-il un upload manuel pour les utilisateurs perso ?

**Non**. La bibliothèque personnelle se remplit automatiquement via
les connecteurs cloud (Drive, OneDrive, Dropbox, S3, WebDAV…) ou
les scrapers RSS/web.

L'upload direct de fichier est réservé aux **organisations**
(`/org/<slug>/workspace`).

### Combien d'utilisateurs simultanés ?

Sans goulot d'étranglement matériel : **plusieurs centaines** sur
une infra correctement dimensionnée. Le bottleneck typique est
Ollama (synthèse IA) sur un seul GPU — au-delà de ~20-30 utilisateurs
actifs simultanés, prévoir un cluster GPU ou bascule BYOK
souverain-hybride.

### Le multi-tenant fonctionne-t-il en on-prem ?

Oui. Une instance Myeline peut héberger **plusieurs organisations**,
chacune avec sa propre collection ChromaDB, ses utilisateurs, son
OIDC, son provider IA (en souverain-hybride). Les organisations sont
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

### Puis-je intégrer Myeline à mon SSO entreprise ?

Oui — OIDC (Azure AD, Okta, Keycloak, Authentik, tout IdP standard).
Voir [SSO entreprise](admin/oidc-sso.md).

### Y a-t-il une API publique ?

Pas d'API REST exposée formellement aux utilisateurs finaux dans
les éditions actuelles. L'admin a accès aux endpoints internes
(`/admin/*`, `/metrics`, `/healthz`), et les blueprints Flask sont
introspectables si vous voulez scripter en interne.

Une API publique (REST + OpenAPI) est sur la roadmap pour Q4 2026.

## Support

### Quel est le SLA ?

24 h ouvrées en réponse standard, 72 h pour les correctifs critiques
(souverain et souverain-hybride). Détail dans le contrat de support.

### Où poser une question ?

- Email : [hello@myeline.io](mailto:hello@myeline.io)
- Pour les bugs reproductibles : joindre une trace `/health` + logs
  `web` des 50 dernières lignes.

### Puis-je accéder au code source ?

Oui en souverain-hybride sous escrow et sur demande spécifique
(clauses commerciales). Le code de Myeline n'est pas open-source
public mais l'éditeur peut fournir un dépôt en lecture pour audit
sécurité côté client.
