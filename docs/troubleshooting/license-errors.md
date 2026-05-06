# Erreurs de licence

Le système de licence Myeline utilise des signatures **Ed25519
hors-ligne** (pas d'appel réseau pour valider). Les erreurs les plus
fréquentes et leurs résolutions.

## `LicenseExpiredError`

**Symptôme** : la bannière `/license-info` indique « Expirée », les
fonctionnalités payantes (synthèse IA externe, connecteurs cloud
publics) sont bloquées.

**Cause** : votre clé de licence dépasse sa date d'expiration (12
mois max par construction).

**Résolution** :

1. Contacter [hello@myeline.io](mailto:hello@myeline.io) avec votre
   référence client pour recevoir une nouvelle clé.
2. Coller la nouvelle clé dans `/admin/license` (ou via `.env`
   `LICENSE_KEY=…` puis redémarrer).
3. Vérifier la nouvelle date d'expiration sur `/license-info`.

Voir [Renouvellement de licence](../operations/license-renewal.md)
pour la procédure complète.

## `LicenseRevokedError`

**Symptôme** : message « Licence révoquée par l'éditeur ».

**Cause** : Myeline a publié une révocation pour cette clé (typiquement
suite à une fuite, un retour produit, ou une erreur d'émission).

**Résolution** :

- En souverain pur (air-gap), la révocation est **embarquée dans la
  prochaine release de l'image** (fichier `app/data/licenses_revoked.json`).
  Mettre à jour l'image (voir [Mise à jour](../operations/upgrade.md)).
  Si vous ne pouvez pas mettre à jour, contactez-nous : on émet une
  nouvelle clé de remplacement après vérification.
- En souverain-hybride, idem — mais nous pouvons aussi pousser une
  révocation hors-bande (image rebuild dédié).

## `LicenseInvalidSignatureError`

**Symptôme** : message « Signature de licence invalide ».

**Causes possibles** :

1. **Clé corrompue** au copier-coller (espaces, retours chariot
   parasites). Recopier en mode brut depuis l'email d'origine.
2. **Mauvaise clé publique** côté serveur. La clé publique de
   vérification est embarquée dans l'image (`app/services/license.py`).
   Si vous avez modifié ce fichier, restaurez-le.
3. **Tampering** : quelqu'un a tenté d'altérer le payload. C'est
   un signal d'alerte sécurité — auditer qui a manipulé `.env`.

## `LicenseClockSkewError`

**Symptôme** : message « Horloge serveur décalée ».

**Cause** : la date système du serveur diverge de plus de 24 h de
la réalité — ce qui rend la validation de la fenêtre `not_before`
/ `expires_at` impossible.

**Résolution** :

```bash
# Vérifier
date
timedatectl status

# Corriger via NTP
sudo systemctl restart chronyd
sudo chronyc tracking

# Si pas de Internet (air-gap), pointer chronyd vers un NTP interne
```

**Important en souverain pur** : NTP doit fonctionner en interne (pas
besoin d'Internet, mais besoin d'un serveur de temps fiable joignable).

## `LicenseModeMismatchError`

**Symptôme** : « Mode de déploiement incompatible avec la licence »
au démarrage.

**Cause** : `DEPLOYMENT_MODE=sovereign-hybrid` dans `.env` mais la
clé délivrée est pour `sovereign` pur (ou inverse).

**Résolution** : aligner les deux. Soit changer `DEPLOYMENT_MODE`,
soit demander une nouvelle clé pour le bon mode.

## Diagnostic

```bash
# Décoder le payload de votre licence (sans valider la signature)
podman exec myeline-web python -c "
import os, json, base64
key = os.environ['LICENSE_KEY']
payload_b64 = key.split('.')[0]
print(json.dumps(json.loads(base64.urlsafe_b64decode(payload_b64 + '==')), indent=2))
"
```

Affiche `customer`, `mode`, `not_before`, `expires_at`, `tier`. Utile
pour diagnostiquer un mismatch sans contacter le support.
