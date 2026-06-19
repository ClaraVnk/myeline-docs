# Obtenir une clé API gratuite en 2 minutes

Sur l'édition **Free** de Myeline, vous apportez votre propre clé API
IA (BYOK). Bonne nouvelle : plusieurs fournisseurs offrent un **niveau
gratuit**, sans carte bancaire. Ce guide vous montre la procédure la
plus rapide.

!!! tip "Vous êtes sur l'édition Pro ?"
    Pas besoin de clé : **Mistral est inclus**. Ce guide ne concerne
    que l'édition [Free](../editions/cloud.md), où vous apportez votre
    propre clé.

## Le plus simple : Google Gemini (gratuit, sans carte)

1. Ouvrez [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Connectez-vous avec un compte Google.
3. Cliquez sur **Create API key**.
4. Copiez la clé affichée (elle commence par `AIza…`).

Aucune carte bancaire requise, le niveau gratuit suffit largement pour
un usage individuel.

## Alternative gratuite : Mistral

1. Ouvrez [console.mistral.ai/api-keys](https://console.mistral.ai/api-keys).
2. Créez un compte (email + mot de passe).
3. Cliquez sur **Create new key**.
4. Copiez la clé.

Mistral propose un niveau gratuit et c'est un LLM **européen** — utile
si la localisation des données compte pour vous.

## Options payantes : OpenAI et Anthropic

Ces deux fournisseurs nécessitent un **compte de facturation actif**
(carte bancaire) — il n'y a pas de niveau réellement gratuit.

- **OpenAI** : [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
  → *Create new secret key*.
- **Anthropic Claude** : [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)
  → *Create Key*.

!!! warning "Gardez votre clé secrète"
    Une clé API donne accès à votre compte (et à votre facturation).
    Ne la partagez jamais et ne la publiez pas. Si elle fuite,
    révoquez-la depuis la console du fournisseur et générez-en une
    nouvelle.

## Dernière étape

Collez votre clé dans **Paramètres → Fournisseur IA** dans Myeline.
C'est tout — la recherche RAG produit désormais des synthèses sur
votre clé.
