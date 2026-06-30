# Conversations multi-tours

Une **conversation** est un fil de discussion persistent — chaque
question prend en compte le contexte des échanges précédents. Utile
pour raffiner un sujet, comparer plusieurs réponses, ou enchaîner
plusieurs questions liées sans tout répéter.

## Démarrer une conversation

Sur la page de recherche `/user`, cocher « Conversation multi-tours »
avant la première question. Les questions suivantes apparaissent
dans le même fil et héritent du contexte.

Vous pouvez aussi convertir une recherche existante en conversation
via le bouton « Continuer cette discussion ».

## Persistance

Toutes les conversations sont **chiffrées au repos** (Fernet,
`SECRET_KEY` dérivée) et conservées indéfiniment dans
`/user/conversations`. Vous pouvez :

- **Renommer** une conversation
- **Archiver** (masquer de la liste principale sans supprimer)
- **Supprimer** définitivement (purge immédiate)
- **Exporter** en Markdown ou JSON

## Limites contextuelles

Le contexte transmis au LLM à chaque tour comprend :

- Les **N derniers tours** (configurable, par défaut 5)
- Les **citations sources** des tours précédents
- Le **prompt système** de l'organisation (si configuré)

Au-delà de la fenêtre N, les anciens tours sont résumés
automatiquement par un LLM auxiliaire pour rester sous la limite
de contexte du modèle de synthèse.

## Bonnes pratiques

- Une conversation = un sujet. Ouvrir un nouveau fil pour un sujet
  différent évite la pollution du contexte.
- Reformuler explicitement quand le LLM dérive (« Reviens à la
  question initiale sur X »).
- Utiliser le mode strict (voir [Recherche RAG](rag-search.md#mode-strict))
  si vos décisions opérationnelles dépendent de la réponse.
