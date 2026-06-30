# FAQ Cloud

Questions fréquentes sur l'offre **hébergée** (Free / Pro) de
[myeline.io](https://myeline.io). Pour les questions on-premise, voir la
[FAQ on-premise](../faq.md).

## Tarifs

### Combien coûte l'offre hébergée ?

- **Free** : **gratuit**, sans limite de durée — vous branchez votre
  **propre clé IA** (BYOK : Mistral / OpenAI / Claude / Gemini).
- **Pro** : **7,90 €/mois**, **facturation mensuelle**, avec **15 jours
  d'essai gratuit**. Mistral inclus (aucune clé à gérer) et toutes les
  limites levées.
- **Équipes / organisations** : facturées **par siège**, soit
  **7,90 €/membre**.

Comparatif complet : [Édition Cloud (SaaS)](../editions/cloud.md).

### Qu'apporte Pro par rapport à Free ?

Pro inclut **Mistral** (indexation + synthèse, aucune clé à gérer),
ajoute la **recherche augmentée** (HyDE, multi-requêtes, reranking,
contextual retrieval), et lève toutes les limites : documents et
sources **illimités**, **chat multi-tours**, **alertes de veille**,
**digest toutes fréquences**, **sync jusqu'à toutes les heures**.

### Y a-t-il une offre annuelle ?

Non. **Pro est mensuel uniquement** (7,90 €/mois). Pour un besoin
on-premise, le tarif est **sur devis** :
[hello@myeline.io](mailto:hello@myeline.io).

## Essai

### L'essai Pro est-il vraiment gratuit ?

Oui : **15 jours**, **aucun débit avant J+15**, **annulable à tout
moment** (carte requise pour démarrer l'essai). À la fin de l'essai,
vous passez en Pro mensuel à 7,90 €/mois, ou vous revenez en Free.

### Que se passe-t-il si je ne passe pas Pro ?

Vous restez en **Free**, gratuitement et sans limite de durée, avec
votre propre clé API. Free reste pleinement utilisable au quotidien.

## Équipes

### Comment fonctionne la facturation par siège ?

Vous créez une **organisation** et invitez des membres. Chaque membre
actif est facturé **7,90 €/mois**. Tous les membres disposent des
fonctionnalités **Pro complètes** et partagent une **bibliothèque
d'organisation**.

### Les bibliothèques sont-elles isolées entre organisations ?

Oui. Chaque organisation a sa propre collection vectorielle, ses
utilisateurs et ses sources. Les organisations sont strictement
isolées les unes des autres.

## Données et résidence

### Où sont hébergées mes données ?

La plateforme Cloud est **hébergée en Europe (France)**. L'embedding et
la synthèse passent par l'**API Mistral AI** (hébergée en UE / France,
**pas d'entraînement sur les données API**). En **Free**, la synthèse
et l'indexation passent par **votre** clé : les données transitent
alors vers **votre** provider IA, selon ses conditions.

### Est-ce conforme RGPD ?

Oui, par construction : hébergement UE, chiffrement, durée de rétention
configurable, exercice des droits (accès, effacement). Détail :
[Conformité RGPD](../compliance/gdpr.md) et
[Localisation des données](../compliance/data-residency.md).

### Et si je ne peux pas envoyer mes données vers un SaaS ?

Si la réglementation impose un air-gap ou une souveraineté stricte
(HDS, SecNumCloud), l'offre **On-premise / Souverain** déploie Myeline
sur **votre propre infrastructure**. C'est une installation
**sur-mesure, sur devis** : [Choisir son édition](../editions/index.md)
ou [hello@myeline.io](mailto:hello@myeline.io).

## Démarrage

### Comment je commence ?

Voir les [Premiers pas (Cloud)](getting-started.md) : créer un compte,
brancher une clé (ou passer Pro), connecter une source, lancer une
première recherche RAG.
