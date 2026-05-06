# Conformité

Myeline est conçu pour faciliter votre conformité aux principaux
référentiels qui s'appliquent aux solutions RAG en environnement
professionnel.

- [RGPD](gdpr.md) — registre des sous-traitants, DPA, exercice des
  droits, durées de conservation.
- [Localisation des données](data-residency.md) — où vivent
  physiquement vos données selon l'édition choisie.
- [Cartographie ISO 27001](iso-27001-mapping.md) — correspondance
  entre les contrôles de l'annexe A et les fonctionnalités natives
  de la plateforme (utile en audit).

## Référentiels couverts par construction

| Référentiel             | Édition                 | Couverture                                      |
|-------------------------|-------------------------|-------------------------------------------------|
| **RGPD**                | Toutes                  | Native (chiffrement, audit, droits, retention)  |
| **HDS (Santé)**         | Souverain               | Compatible — l'hébergeur étant vous, votre infra doit être HDS-certifiée |
| **SecNumCloud**         | Souverain               | Compatible par construction (air-gap)           |
| **NIS2 / LPM (OIV)**    | Souverain               | Compatible — l'isolation infra est compatible avec les exigences |
| **ISO 27001 / 27701**   | Toutes                  | Aide via la cartographie + journal d'audit      |
| **AI Act (UE)**         | Toutes                  | Système d'IA à risque limité ; transparence + log d'usage |

Aucune édition ne « certifie » seule votre conformité — la
certification est attribuée à votre organisation et son
infrastructure, pas à Myeline. Mais Myeline ne crée pas d'obstacle
technique aux audits.

## Documents de référence

Disponibles sur demande à [hello@myeline.io](mailto:hello@myeline.io) :

- DPA (Data Processing Agreement) à signer avec Myeline (en
  souverain pur, l'objet du DPA est purement le support technique
  — Myeline n'a pas accès aux données)
- Politique de sécurité (PSSI Myeline)
- Pen test report (annuel, par un tiers indépendant)
- Modèle de registre des traitements (Article 30 RGPD) pré-rempli
  pour la fonctionnalité Myeline
