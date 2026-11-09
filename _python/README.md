# Backend Python : uv et ruff

Fichiers de référence pour le dossier `back/` d'un projet TelesCoop.

| Fichier | Rôle |
|---|---|
| [`pyproject.toml`](pyproject.toml) | Dépendances (production et groupe `dev`), configuration de ruff |
| [`.python-version`](.python-version) | Version de Python utilisée en local, en CI et sur le serveur |
| [`.pre-commit-config.yaml`](.pre-commit-config.yaml) | Hooks ruff (lint et format) et vérification que `uv.lock` est à jour |

## Principes

- **Un lockfile versionné** : `uv.lock` fige toutes les versions (dépendances transitives comprises) ; la CI et le déploiement l'utilisent avec `--locked` et échouent s'il n'est plus à jour.
- **Python géré par uv** : la version vient de `.python-version`, en local, en CI comme sur le serveur. Elle ne dépend plus de la distribution. Ce fichier est donc obligatoire.
- **Dépendances de production séparées** : les outils (ruff, coverage…) vont dans le groupe `dev`, qui n'est pas installé en production (`uv sync --no-dev`).
- **Django 6.x**, mises à jour hebdomadaires proposées par Dependabot (voir [`_github/dependabot.yml`](../_github/dependabot.yml)) et audit de vulnérabilités en CI (`pip-audit`).

## Migrer un projet depuis `requirements.txt`

```bash
cd back
cp ../path/to/deploy-template/_python/{pyproject.toml,.python-version,.pre-commit-config.yaml} .
# Adapter le nom du projet dans pyproject.toml, puis reprendre les dépendances existantes :
uv add -r requirements.txt
uv add --dev coverage ruff
uv lock
git rm requirements.txt
uv run ruff check --fix . && uv run ruff format .
```

Sur le serveur, il suffit que `uv.lock` soit présent dans `backend_path` : le déploiement utilise alors `uv sync --locked` à la place de `requirements.txt`.

## Commandes courantes

```bash
uv sync                      # installe les dépendances (dev comprises)
uv add django-filter         # ajoute une dépendance et met à jour uv.lock
uv lock --upgrade-package django  # met à jour une dépendance précise
uv run python manage.py runserver
uv run ruff check --fix . && uv run ruff format .
```
