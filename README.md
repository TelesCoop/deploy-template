
# Template de déploiement Ansible pour projet web Front et/ou Back

Template Ansible pour automatiser le déploiement et la maintenance d'applications web.

En général dans la pratique utilisé pour Django (back) et Vue 3 (front), mais s'adapte à d'autres cas.
basées sur Django (backend) et Vue 3 (ou Nuxt) en front-end.

Note : Ce template ne couvre pas la configuration initiale d'un nouveau serveur (utilisateurs, SSH, sécurité de base). Pour cela, utilisez le repository dédié : <https://github.com/TelesCoop/ansible-ssh-config>

## Prérequis

### Sur votre machine locale

- **Ansible 2.9+** installé
- **Git** avec accès aux repositories du projet
- **Clé du vault Ansible** (`vault.key`) pour accéder aux variables chiffrées (chez TelesCoop, cf Bitwarden)

### Sur les serveurs cibles

- **Ubuntu/Debian** (testé sur Ubuntu 18.04+)
- **Python 3** avec pip
- **Accès SSH** avec privilèges sudo

## Stack technique

### Composants principaux

- **Base de données** : PostgreSQL ou SQLite
- **Serveur web** : Nginx

### Services externes (optionnels)

- **Mailgun** : envoi d'emails transactionnels
- **Service S3** : stockage des sauvegardes de base de données
- **Rollbar** : monitoring et tracking des erreurs en production
- **Let's Encrypt** : certificats SSL automatiques

## Adaptation pour un nouveau projet

### 0. Copier le contenu du répertoire

Copier le contenu de ce dépôt dans un dossier "deploy" dans votre projet.

### 1. Configuration initiale

```bash
cd deploy

# Installer pre-commit (recommandé)
pip install pre-commit
pre-commit install

# Récupérer la clé du vault et la placer à la racine
cp /chemin/vers/vault.key .
```

### 2. Configuration

- éditer `group_vars/all/vars.yml`, se laisser guider par les commentaires
- adapter `frontend.yml` et `backend.yml` à votre application

### 3. Configuration des environnements

Pour ajouter un environnement,

- modifier le fichier `hosts` pour ajouter un environnement `new_env`
- copier/coller le dossier `groups_vars/prod` vers `group_vars/new_env` et faire les modifications appropriées

### 4. Génération des secrets

```bash
# Générer une nouvelle clé de vault
bash generate_vault_key_on_first_install.sh

# Éditer les variables secrètes
ansible-vault edit group_vars/all/cross_env_vault.yml
```

Pour générer la clé de sécurité Django, depuis l'environnement virtuel :

`python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"`

## Déploiement

Lancer simplement `ansible-playbook backend.yml` et/ou `ansible-playbook frontend.yml`

Les arguments supplémentaires

- `-e force_update=1` permet de rafraichir les installations de dépendances et l'appli front/back même si le code n'a pas changé (par exemple si des paramètres ont changé)
- `-e force_create_certificate=1` permet d'ajouter de nouveaux domaines (nouvellement ajoutés à la variable `https_hostnames`) au certificat https

## Commandes de maintenance

Notes : toutes les commandes ci-dessous peuvent aussi être exécutées sur le serveur.

### Surveillance et logs

```bash
# Vérifier le statut des services
ansible prod -m shell -a "supervisorctl status"

# Consulter les logs
ansible prod -m shell -a "tail -f /var/log/telescoop/{project}_prod/*.log"
```

### Quelques commandes Django utiles

```bash
# Shell Django interactif
ansible prod -m shell -a "sudo /org/projet/projet-ctl shell"

# Collecte des fichiers statiques
ansible prod -m shell -a "sudo /org/projet/projet-ctl collectstatic --noinput"

# Création d'un superutilisateur
ansible prod -m shell -a "sudo /org/projet/projet-ctl createsuperuser"
```

### Redémarrage des services

```bash
# Redémarrer tous les services
ansible prod -m shell -a "supervisorctl restart {projet}_prod-backend/frontend"

# Redémarrer nginx
ansible prod -m shell -a "systemctl restart nginx"
```

## Monitoring et logs

- **Logs centralisés** : `/var/log/votre-org/votre-projet/`
  - `backend.log` : Logs de l'application Django
  - `frontend.log` : Logs frontend (mode SSR uniquement)
  - `nginx-access.log` et `nginx-error.log` : Logs du serveur web
- **Supervision** : supervisord pour le monitoring des processus
- **Rotation automatique** : Configuration logrotate pour éviter la saturation disque
- **Rollbar** : Tracking des erreurs en production (si configuré)

## Dépannage

### Problèmes courants

#### Service ne démarre pas

```bash
# Vérifier les logs supervisord
ansible prod -m shell -a "tail -f /var/log/supervisor/supervisord.log"

# Redémarrer supervisord
ansible prod -m shell -a "systemctl restart supervisor"
```

#### Problème de permissions

```bash
# Vérifier les permissions des dossiers
ansible prod -m shell -a "ls -la /votre-org/votre-projet/"

# Corriger les permissions si nécessaire
ansible-playbook backend.yml --tags permissions
```

#### Erreur de base de données

```bash
# Vérifier la connexion PostgreSQL
ansible prod -m shell -a "sudo -u postgres psql -l"
```

## Exemples

### Installer postgis

Dans backend.yml, ajouter

```
- name: Install and setup postgis to postgresql
  block:
    - name: Install postgis
      apt:
        name:
          - postgresql-14-postgis-3
    - name: Adds postgis extension to the database {{ project_slug }}
      community.postgresql.postgresql_ext:
        name: postgis
        login_db: "{{ database_user }}"
      become: true
      become_user: postgres
  when: database_provider == "postgresql"
```
