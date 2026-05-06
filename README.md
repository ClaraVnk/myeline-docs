# Myeline — Documentation publique

Source de [wiki.myeline.io](https://wiki.myeline.io).

Documentation officielle des éditions **on-prem** de la plateforme
**Myeline** : RAG self-hosted made in France, déployable en mode
souverain (air-gap on-prem) ou souverain-hybride (on-prem + APIs
externes BYOK).

## Architecture

- **Source** : ce dépôt — markdown + [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).
- **Build** : déclenché à chaque push sur `main` via GitHub Actions
  (`.github/workflows/deploy.yml`).
- **Hébergement** : GitHub Pages, alias custom `wiki.myeline.io`.
- **Recherche** : lunr.js intégré, FR + EN.

## Run

### Aperçu local

```bash
git clone git@github.com:ClaraVnk/myeline-docs.git
cd myeline-docs
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
# http://127.0.0.1:8000
```

### Build

```bash
mkdocs build --strict
# Sortie dans site/
```

## Test

```bash
mkdocs build --strict
# strict casse le build sur les liens cassés ou warnings —
# c'est ce qui tourne en CI.
```

## Deploy

Push sur `main` → GitHub Actions build et publie sur GitHub Pages.
Pas d'intervention manuelle.

Pour pointer `wiki.myeline.io` :

1. **Settings → Pages** : sélectionner *Deploy from a branch* OFF,
   utiliser GitHub Actions.
2. **Custom domain** : `wiki.myeline.io`. GitHub propose un `CNAME`
   record à ajouter chez ton DNS.
3. Cocher **Enforce HTTPS**.

## Architecture des dossiers

```
mkdocs.yml            # Config — theme, nav, plugins
requirements.txt      # Pinned MkDocs Material + plugins

docs/
├── index.md          # Landing
├── editions/         # Choix de l'édition
├── install/          # Walkthroughs d'installation
├── operations/       # Day-2 ops
├── admin/            # Panneau admin
├── usage/            # Guide utilisateur
├── compliance/       # RGPD, ISO, residency
├── troubleshooting/  # FAQ + erreurs courantes
└── faq.md
```

## Contribution

Les docs sont en français par défaut. Tout texte versionné (titres,
paragraphes, code) reste en français. Les commits suivent
[Conventional Commits](https://www.conventionalcommits.org/) :

```
docs(install): add sovereign-hybrid OAuth walkthrough
fix(faq): correct typo in license renewal section
```

PRs bienvenues — les builds CI vérifient les liens et la cohérence
de la nav.

## Licence

Le contenu est sous [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) :
réutilisable avec attribution, **sans modification**, **non
commercial**. Voir [LICENSE](LICENSE).

Le logiciel **Myeline** lui-même n'est PAS open source — il est
distribué sous licence commerciale (Ed25519-signée, 12 mois max).
Voir [editions](https://wiki.myeline.io/editions/) pour les détails.
