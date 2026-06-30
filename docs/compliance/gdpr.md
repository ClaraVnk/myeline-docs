# Conformité RGPD

!!! info "Édition"
    Concerne **toutes les éditions**, avec des postures différentes :
    **Cloud** = Myeline est sous-traitant (DPA) ; **On-premise** = vous
    êtes responsable de traitement ; **Community** = open source.

Cette page documente la posture RGPD de Myeline en éditions on-prem
(souverain et souverain-hybride).

## Rôles

En souverain et souverain-hybride, **vous êtes responsable de
traitement** (au sens RGPD art. 4(7)). Myeline est éditeur du
logiciel, pas sous-traitant — sauf si vous nous confiez un contrat
de support qui inclut un accès opérationnel à votre instance, auquel
cas un DPA dédié s'applique pour ces interventions.

## Registre des sous-traitants

Liste des sous-traitants potentiels selon l'édition :

### Souverain (air-gap)

**Aucun sous-traitant** — la plateforme tourne 100 % chez vous,
aucune donnée personnelle n'est transférée vers un tiers.

### Souverain-hybride (BYOK)

Selon les services activés par chaque organisation :

| Service                 | Données transmises                          | Localisation     |
|-------------------------|---------------------------------------------|------------------|
| Mistral AI              | Requêtes RAG (texte question + chunks)      | UE (France)      |
| Anthropic Claude        | Requêtes RAG                                | US               |
| OpenAI                  | Requêtes RAG                                | US               |
| Google Gemini           | Requêtes RAG                                | US / UE selon plan |
| Brevo (mailer)          | Emails transactionnels (adresse + contenu)  | UE (Allemagne)   |
| Provider OAuth (Google, MS, Dropbox…) | Identifiants OAuth + listing fichiers | US (typiquement) |

**À chaque organisation** d'inscrire ces sous-traitants dans **son
propre** registre RGPD selon les services qu'elle a activés.
Myeline ne contraint aucun choix — vous pouvez tout désactiver et
rester en mode local.

## Bases légales

| Traitement                          | Base légale                          |
|-------------------------------------|--------------------------------------|
| Création de compte                  | Exécution du contrat (CGU)           |
| Authentification                    | Exécution du contrat                 |
| Recherche RAG                       | Exécution du contrat                 |
| Connecteurs cloud                   | Consentement (OAuth explicite)       |
| Email transactionnel                | Exécution du contrat                 |
| Email marketing (newsletter)        | Consentement opt-in                  |
| Journal d'audit                     | Obligation légale (art. 30 RGPD)     |
| Métriques anonymes (Prometheus)     | Intérêt légitime (sécurité, perf)    |

## Droits des personnes

| Droit                  | Mise en œuvre                                                |
|------------------------|--------------------------------------------------------------|
| **Accès**              | Export complet via `/account/export-data` (JSON + uploads)   |
| **Rectification**      | Via le profil `/account` (nom, prénom, email vérifié)        |
| **Effacement**         | `/account/delete` (soft) ou demande à l'admin (hard purge)   |
| **Limitation**         | Suspension du compte sur demande (ne supprime pas les données) |
| **Portabilité**        | Même export que l'accès, format JSON ouvert                  |
| **Opposition**         | Désinscription newsletter + désactivation watch alerts       |
| **Décision automatisée**| Aucune décision automatisée individualisante n'est prise par Myeline |

Délai de traitement : **30 jours** maximum, conforme à l'art. 12.3.

## Durées de conservation

| Donnée                              | Durée                                    |
|-------------------------------------|------------------------------------------|
| Compte actif                        | Durée de la relation contractuelle       |
| Compte inactif                      | 3 ans après dernière connexion → purge   |
| Compte non confirmé                 | 30 jours → purge automatique             |
| Journal d'audit (actif)             | 13 mois                                  |
| Journal d'audit (archivé S3)        | 5 ans (configurable)                     |
| Logs applicatifs                    | 90 jours                                 |
| Logs mailer (souverain pur)         | 30 jours (à purger manuellement)         |
| Backups                             | 14 jours en local + selon votre politique off-host |
| Conversations RAG                   | Indéfini, suppression manuelle par l'utilisateur |

## Sécurité technique

- **Chiffrement au repos** : DB MariaDB chiffrée par votre LUKS / FS,
  champs sensibles (tokens OAuth, mots-clés alertes, conversations,
  TOTP secrets) chiffrés au niveau applicatif via Fernet.
- **Chiffrement en transit** : TLS 1.2+ obligatoire (HTTPS,
  HSTS, secure cookies).
- **Authentification** : argon2id pour les mots de passe, TOTP
  optionnel, OIDC SSO entreprise.
- **Pseudonymisation des logs** : aucun email loggé, seuls
  `user_id` numériques.
- **Audit log** : toute action sensible tracée.

## Notifications de violation

En cas de violation de données vous concernant côté infrastructure
Myeline (provider cloud, registre git, etc.) :

- Notification à l'admin sous **72 h** par email signé.
- Détails : nature de la violation, données concernées, mesures
  prises, recommandations.

Aucune violation côté votre infra ne nous est notifiable — c'est
votre périmètre.

## Contact DPO

Délégué à la protection des données Myeline :
[dpo@myeline.io](mailto:dpo@myeline.io).
