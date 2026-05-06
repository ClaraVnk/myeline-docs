# Recherche RAG

La recherche RAG est le cœur de Myeline : poser une question en
langage naturel, recevoir une réponse synthétisée **avec citations
des sources**.

## Pipeline

```
Question utilisateur
  ↓
Embedding bge-m3 (Ollama local)
  ↓
ChromaDB hybrid search (RRF + MMR sur les scopes sélectionnés)
  ↓
HyDE / multi-query / contextual retrieval (LLM auxiliaire)
  ↓
Cross-encoder reranking (LLM auxiliaire)
  ↓
Synthèse (Ollama local OU API externe BYOK selon édition)
  ↓
Réponse + citations [1], [2], [3]…
```

L'**embedding est toujours local**, peu importe l'édition. La
synthèse :

- En souverain pur : Ollama local (Mistral-Nemo, Llama 3.1, Mixtral
  selon ce que vous hostez)
- En souverain-hybride : Ollama local par défaut, ou bascule par
  organisation vers Mistral / Claude / OpenAI / Gemini avec votre
  clé (BYOK)

## Citations

Chaque réponse cite ses sources sous forme de `[1]`, `[2]`,
cliquables. La source affiche :

- **Titre du document** ou de l'article
- **Auteur / source** quand disponible
- **Date** quand disponible
- **Extrait** (chunk de ~500 tokens) qui a contribué à la réponse
- **Score de pertinence**

Si une réponse n'a pas de citation, c'est que le LLM n'a pas trouvé
de support dans les documents — soit la base ne contient pas
l'information, soit votre question est trop vague.

## Mode strict

Toggle dans l'interface : si activé, **le LLM est forcé à ne
répondre que sur la base des sources retrouvées** (prompt système
explicite + post-validation). Si aucune source ne couvre la
question, la réponse est : « Je n'ai pas trouvé l'information dans
les sources disponibles. »

Recommandé pour :

- Recherche réglementaire / juridique
- Recherche médicale
- Tout contexte où une hallucination serait un risque

## Single-document

Depuis la bibliothèque, cliquer sur un document ouvre un chat
**scopé à ce seul document**. Utile pour :

- Interroger un long rapport (« quelles sont les recommandations ? »)
- Comparer des sections (« les conclusions de la partie 3 contredisent-elles la partie 1 ? »)
- Extraire des données chiffrées (« quel est le CA 2024 mentionné ? »)

## Conseils d'écriture

- **Questions précises** > questions vagues. « Quelles sont les
  exceptions au consentement explicite RGPD selon l'art. 6 ? »
  marche mieux que « parle-moi du RGPD ».
- **Contexte** : préciser le domaine si vos sources couvrent plusieurs
  sujets (« en droit du travail français, … »).
- **Itérer** : raffiner la question en multi-tours (voir
  [Conversations](conversations.md)) plutôt que tout dire en une fois.

## Limites connues

- Documents > 200 pages : le chunking peut diluer les liens entre
  sections distantes. Préférer single-document chat avec contextual
  retrieval activé.
- Tableaux complexes : extraction PDF imparfaite (limites des
  parseurs `pdfplumber`/`docx`). Préférer un export CSV quand
  possible.
- Langues mixtes dans une même base : `bge-m3` est multilingue (100+
  langues), mais la qualité de retrieval dégrade quand FR et EN se
  mélangent dans une même question.
