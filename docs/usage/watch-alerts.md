# Alertes et veille

Une **alerte de veille** surveille en continu vos sources (RSS,
scrapers, drives cloud) et vous envoie un email dès qu'un nouveau
contenu mentionne un mot-clé que vous avez défini.

## Créer une alerte

`/user/watch-alerts` :

1. **Mot-clé ou expression** : la requête textuelle à surveiller
   (ex. `directive NIS2`, `cybersecurity workforce act`).
2. **Périmètre** : personnel / organisation / public, ou combinaison.
3. **Fréquence d'envoi** : immédiat / quotidien / hebdomadaire.
4. **Sensibilité** (`high` / `medium` / `low`) : seuil minimum de
   pertinence d'un match pour déclencher un email.

Les mots-clés sont **chiffrés au repos** (`watch_alert.keyword_enc`).

## Mécanisme

Le cron `check_watch_alerts` (08:00 + 20:00 UTC, fuseau utilisateur)
parcourt les nouveaux contenus depuis le dernier run, calcule un
score de similarité contre les alertes actives, et déclenche un
email digest si le seuil est atteint.

```
Nouveaux articles / docs depuis dernier run
  ↓
Embedding bge-m3 du contenu vs embedding du mot-clé
  ↓
Score de pertinence (cosine similarity)
  ↓
Si score ≥ seuil : ajouter au digest utilisateur
  ↓
Envoi groupé en fin de cron (1 email par utilisateur, max 50 hits)
```

## Gestion

- **Pause / reprise** : désactiver temporairement sans perdre la
  config.
- **Modifier** le mot-clé ou le seuil — recalcul du backlog
  optionnel (1 fois max par 24 h).
- **Supprimer** : purge immédiate de l'alerte et de son historique
  de hits.

## Limites

- Maximum **20 alertes actives** par utilisateur (configurable via
  `WATCH_ALERTS_PER_USER`).
- Délais de détection : 8 h max (cron deux fois par jour). Pas de
  notification temps réel — c'est de la veille, pas du monitoring.
- En souverain pur : les emails sont en **log-only** par défaut
  (mailer désactivé). Configurer un MTA interne pour les recevoir
  réellement (voir [`docs/EMAIL.md`](https://github.com/myeline/myeline)
  côté repo applicatif).

## Pour qui ?

- Veille concurrentielle : surveiller le nom d'un concurrent dans
  votre flux d'articles.
- Veille réglementaire : être alerté dès qu'une nouvelle directive
  ou un texte mentionne un sujet sensible.
- Suivi de marque : détecter les mentions de votre organisation.
- Recherche académique : nouveaux papiers sur un thème précis (en
  combinaison avec le connecteur Zotero).
