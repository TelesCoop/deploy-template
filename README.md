
# Mise en production et maintenance avec Ansible d'un site avec Stack Django Vue

## Stack technique

Principaux outils open-sources

- frontend:
  - Serveur nginx
  - framework Vue
- backend: framework Django, gunicorn pour le wsgi

Outils externes potentiellement payants

- Mailgun pour les mails
- AWS S3 pour la sauvegarde de la BDD
- Rollbar pour le suivi des bogues en production

Ce considère donc :

- que vous utilisez `Vue` ou un autre projet qui se compile avec
  `npm run generate` pour le frontend. Adapter
  `roles/frontend/tasks/main.yml:Build frontend code` si nécessaire
- que vous utilisez `Django` pour le backend
- que vous utilisez les outils externes sus-mentionnés (s'il y en a
  un ou plusieurs que vous n'utilisez pas, il suffit de les supprimer de
  `roles/backend/templates/settings.ini.j2`)
- que vous utilisez `gunicorn` comme outil `wsgi` pour le backend. À adapter
  dans `roles/backend/teamplte/supervisor.conf.j2`
- que vous utilisez
  [telescoop-backup](https://gitlab.com/telescoop-public/django-apps/telescoop-backup)
  pour sauvegarder votre base de donnée. À adapter dans `roles/backend/tasks/cron.yml`.

## Les différents rôles/playbook

Pour un nouveau serveur ou projet, ils sont à lancer dans cet ordre.

- `bootstrap` : gère les accès en ssh. À lancer en premier sur un nouveau serveur.
  Lancer avec `ansible-playbook bootstrap.yml`.
  N'est pas nécessaire en cas d'ajout d'un projet sur le même serveur.

- `base` : installe les dépendances devops du projet. Configure les mises à jour
  automatiques et les roulements de log nginx
  N'est pas nécessaire en cas d'ajout d'un projet sur le même serveur.

- `backend` : télécharge le code back, installe les dépendances du backend et paramètre
  `supervisord`.

- `frontend` : télécharge le code front, installe les dépendances du backend et paramètre
  `nginx`.

## Procédure de maintenance

La maintenance est toujours faite via Ansible. Il est donc
nécessaire d'installer Ansible sur son PC, en version 2.9.

Pour pouvoir utiliser Ansible, il faut avoir la clé du
"vault" qui contient des informations sensibles.
Cette clé peut être récupérée auprès du développeur de l'application ou
de vos collègues.
Elle doit être copiée en local, à la racine de ce dépôt, dans un fichier
nommé `vault.key`.

Installation du serveur ou mise à jour de tout :

- `ansible-playbook bootstrap.yml`
- `ansible-playbook base.yml`

### MAJ ou installation du frontend

`ansible-playbook frontend.yml`

### MAJ ou installation du backend

`ansible-playbook backend.yml`

## Adapter ce template pour un nouveau projet

Commencer par installer, si nécessaire, [`pre-commit`](https://pre-commit.com/)
et l'activer `pre-commit install`. Cela permet d'avoir des vérifications avant
chaque commit.

- modifier le fichier `hosts` pour indiquer sur quel nom de domain ou IP se trouve
  le serveur à manager.
- modifier les variables dans `group_vars/vars.yml`, notamment
  organization_slug, project_slug, main_user, public_hostnames, django_project_name
- modifier les variables dans `group_vars/cross_env_vars.yml`, notamment :
  - le port ssh
- vérifier les variables dans `roles/backend/vars/main.yml` et `roles/frontend/vars/main.yml`.
- générer des identifiants Mailgun, AWS S3 et Rollbar pour le projet.
- modifier la clé du coffre-fort Ansible avec une clé générée aléatoirement en
  lançant `bash generate_vault_key_on_first_install.sh`. Cela crée un fichier
  `vault.key` qui contient la clé du coffre-fort Ansible. Sauvegarder cette clé
  en endroit sûr et la partager de manière sûre avec les collègues (via Lastpass).
- modifier les valeurs du vault: `ansible-vault edit group_vars/all/cross_env_vault.yml`

## TODO

- Comprendre et adapter dans `base` les règles `iptables` et `RAID`.
