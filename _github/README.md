# Github

## Workflows

## Mise en place

Créer un dossier `.github/workflows` avec des fichiers `yml` dedans. Des exemples de workflows sont présents dans ce repository.

Au niveau de Github, il faut que le runner soit capable d'exécuter les scripts Ansible sur le serveur et donc ajouter une clé SSH adéquate au bon serveur

## Exemples

Ces fichiers sont à modifier en fonction du projet.

* [`checks.yml`](/_github/workflows/checks.yml) : pour lancer les tests `front`, `back`, `docs` et le `pre-commit`