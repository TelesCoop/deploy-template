
# Template de déploiement Ansible - Django + Vue (ou Nuxt)

Template Ansible pour automatiser le déploiement et la maintenance d'applications web basées sur Django (backend) et Vue 3 (ou Nuxt) en front-end.


**🎯 Objectif principal** : Adapter ce template pour déployer votre propre projet en quelques étapes simples.

**⚠️ Note importante** : Ce template ne couvre pas la configuration initiale d'un nouveau serveur (utilisateurs, SSH, sécurité de base). Pour cela, utilisez le repository dédié : https://github.com/TelesCoop/ansible-ssh-config

## Table des matières

- [Prérequis](#prérequis)
- [Stack technique](#stack-technique)
- [🚀 Adaptation pour un nouveau projet](#-adaptation-pour-un-nouveau-projet)
- [Déploiement](#déploiement)
- [Commandes de maintenance](#commandes-de-maintenance)
- [Architecture des rôles](#architecture-des-rôles)
- [Configuration avancée](#configuration-avancée)
- [Dépannage](#dépannage)

## Prérequis

### Sur votre machine locale
- **Ansible 2.9+** installé
- **Git** avec accès aux repositories du projet
- **Clé du vault Ansible** (`vault.key`) pour accéder aux variables chiffrées

### Sur les serveurs cibles
- **Ubuntu/Debian** (testé sur Ubuntu 18.04+)
- **Python 3** avec pip
- **Accès SSH** avec privilèges sudo
- **Git** installé
- **Accès Internet** pour télécharger les dépendances

### Accès réseau requis
- **Port SSH** (par défaut 22, configurable)
- **Port HTTP** (80) et **HTTPS** (443) pour le web
- **Ports applicatifs** configurables pour backend et frontend

## Stack technique

### Composants principaux (open-source)
- **Frontend** : Vue 3 (ou Nuxt) + Nginx
- **Backend** : Django + gunicorn + supervisord
- **Base de données** : PostgreSQL ou SQLite
- **Serveur web** : Nginx

### Services externes (optionnels)
- **Mailgun** : envoi d'emails transactionnels
- **Service S3** : stockage des sauvegardes de base de données
- **Rollbar** : monitoring et tracking des erreurs en production
- **Let's Encrypt** : certificats SSL automatiques

## 🚀 Adaptation pour un nouveau projet

### 0. Copier le contenu du répertoire

Copier le contenu de ce dépôt dans un dossier "deploy" dans votre projet.

### 1. Configuration initiale

```bash
# Cloner le template
git clone <votre-repo>
cd deploy-template

# Installer pre-commit (recommandé)
pip install pre-commit
pre-commit install

# Récupérer la clé du vault et la placer à la racine
cp /chemin/vers/vault.key .
```

### 2. Configuration des variables principales

Éditer `group_vars/all/vars.yml` :
```yaml
organization_slug: votre-org
base_project_slug: mon-projet
main_user: mon-projet
main_user_uid: 10042  # Unique par projet sur le serveur
django_project_name: mon_projet_back

# Dans le cas d'un un mono-repo (is_mono_repo: true), le back-end et le front-end sont configurés pour utiliser le même dépôt.
backend_repo: git@github.com:votre-org/mon-projet-fullstack.git
frontend_repo: git@github.com:votre-org/mon-projet-fullstack.git

# Pour des repos séparés (is_mono_repo: false), le backend et le frontend sont configurés pour utiliser des dépôts différents.
# backend_repo: git@github.com:votre-org/mon-projet-backend.git
# frontend_repo: git@github.com:votre-org/mon-projet-frontend.git
```

### 3. Configuration des environnements

Éditer `group_vars/all/cross_env_vars.yml` pour définir :
- Ports SSH personnalisés
- Domaines publics
- Configuration réseau

Modifier le fichier `hosts` :
```ini
[prod]
votre-serveur.com:22 ansible_user=ubuntu
```

### 4. Génération des secrets

```bash
# Générer une nouvelle clé de vault
bash generate_vault_key_on_first_install.sh

# Éditer les variables secrètes
ansible-vault edit group_vars/all/cross_env_vault.yml
```

Configurer dans le vault les variables suivantes :

```yaml
# Django
django_secret_key: "votre-clé-secrète-django-très-longue"

# Base de données (si PostgreSQL)
database_password: "mot-de-passe-sécurisé"

# Mailgun (optionnel)
mailgun_api_key: "key-xxxxxxxxxxxxx"
mailgun_domain: "mg.votre-domaine.com"

# Sauvegarde S3 (optionnel)
backup_s3_access_key: "AKIAIOSFODNN7EXAMPLE"
backup_s3_secret_key: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
backup_s3_bucket: "mon-projet-backups"
backup_s3_region: "eu-west-3"

# Rollbar (optionnel)
rollbar_access_token: "xxxxxxxxxxxxxxxxxxxxxxxxx"

# Contact
contact_email: "admin@votre-domaine.com"
```

### 5. Personnalisation avancée

#### Mode frontend
Modifier `frontend_mode` dans `vars.yml` :
- `static` : génération statique (JAMstack)
- `SSR` : rendu côté serveur

#### Base de données
Changer `database_provider` :
- `sqlite` : pour les petits projets
- `postgresql` : pour la production

#### Repositories
Adapter les URLs de dépôts dans `vars.yml` selon votre structure :
- Mono-repo : `is_mono_repo: true`
- Repos séparés : `is_mono_repo: false`

#### Logique mono-repo
Lorsque `is_mono_repo` est activé, le template adapte automatiquement sa logique de déploiement :
- **Repository unique** : Le même dépôt contient le backend et le frontend
- **Branches partagées** : Utilisation de la même branche pour backend et frontend
- **Chemins relatifs** : Les chemins de build sont adaptés pour pointer vers les sous-dossiers appropriés

### 6. Ajout d'un environnement

Pour ajouter un environnement de préproduction :

```bash
# Créer le dossier de variables
mkdir group_vars/preprod
cp group_vars/prod/vars.yml group_vars/preprod/

# Ajouter dans hosts
[preprod]
preprod.mon-projet.com:22 ansible_user=ubuntu
```

## Déploiement

### Premier déploiement

Notes : les commandes ont un ordre logique qu'il est recommandé de suivre pour éviter les erreurs.

```bash
# Déploiement du backend
ansible-playbook backend.yml

# Déploiement du frontend
ansible-playbook frontend.yml

# Ou les deux en séquence
ansible-playbook backend.yml && ansible-playbook frontend.yml
```

### Redéploiement 

```bash
# Mise à jour du backend uniquement
ansible-playbook backend.yml

# Mise à jour du frontend uniquement
ansible-playbook frontend.yml

# Forcer la mise à jour des dépendances
ansible-playbook backend.yml -e force_update=1
```

## Commandes de maintenance

Notes : toutes les commandes ci-dessous peuvent aussi être exécutées sur le serveur. 

### Surveillance et logs
```bash
# Vérifier le statut des services
ansible prod -m shell -a "supervisorctl status"

# Consulter les logs
ansible prod -m shell -a "tail -f /var/log/supervisor/backend-*.log"
```

### Gestion de la base de données et Django
```bash
# Script de contrôle backend (remplacez les variables par vos valeurs)
# Sauvegarde manuelle
ansible prod -m shell -a "sudo /org/projet/projet-ctl backup"

# Migration Django
ansible prod -m shell -a "sudo /org/projet/projet-ctl migrate"

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
ansible prod -m shell -a "supervisorctl restart all"

# Redémarrer nginx
ansible prod -m shell -a "systemctl restart nginx"

# Redémarrer uniquement le backend
ansible prod -m shell -a "supervisorctl restart backend-*"

# Redémarrer uniquement le frontend (mode SSR)
ansible prod -m shell -a "supervisorctl restart frontend-*"
```

## Configuration avancée

### Personnalisation des rôles

- **Backend** : Modifier `roles/backend/templates/settings.ini.j2` pour Django
- **Frontend** : Adapter `roles/frontend/tasks/main.yml` pour d'autres frameworks
- **Nginx** : Personnaliser `roles/frontend/templates/nginx.conf.j2`

### Sécurité et SSL

- **Configuration nginx sécurisée** : Protection contre les attaques communes
- **Gestion des permissions utilisateurs** : Utilisateur dédié par projet
- **Variables sensibles chiffrées** : Ansible Vault pour tous les secrets
- **Support SSL/TLS** : Configuration prête pour Let's Encrypt

### Monitoring et logs

- **Logs centralisés** : `/var/log/votre-org/votre-projet/`
  - `backend.log` : Logs de l'application Django
  - `frontend.log` : Logs frontend (mode SSR uniquement)
  - `nginx-access.log` et `nginx-error.log` : Logs du serveur web
- **Supervision** : supervisord pour le monitoring des processus
- **Rotation automatique** : Configuration logrotate pour éviter la saturation disque
- **Rollbar** : Tracking des erreurs en production (si configuré)

#### Consulter les logs Django
```bash
# Logs en temps réel
ansible prod -m shell -a "tail -f /var/log/votre-org/votre-projet/backend.log"

# Logs nginx
ansible prod -m shell -a "tail -f /var/log/nginx/access.log"

# Statut des services
ansible prod -m shell -a "supervisorctl status"
```

## Architecture des rôles

### `backend`
- **Packages système** : Python 3, nginx, supervisord, PostgreSQL (si utilisé)
- **Utilisateur système** : Création d'un utilisateur dédié avec UID personnalisé
- **Base de données** : Configuration PostgreSQL ou SQLite selon `database_provider`
- **Application Django** :
  - Environnement virtuel Python
  - Installation des dépendances via requirements.txt
  - Configuration Django via `settings.ini`
  - Migrations automatiques
  - Collecte des fichiers statiques
- **Processus** : Configuration supervisord pour gunicorn
- **Utilitaires** : Script de contrôle `{project}-ctl` pour la gestion
- **Sauvegardes** : Tâche cron quotidienne vers S3

### `frontend`
- **Node.js** : Installation via NVM (version depuis `.nvmrc`)
- **Code source** : Clonage du repository frontend
- **Dépendances** : Installation via npm/yarn
- **Build** :
  - **Mode static** : Génération statique vers dossier nginx
  - **Mode SSR** : Build pour rendu côté serveur + supervisord
- **Configuration nginx** : Proxy, SSL, gestion des erreurs
- **Variables d'environnement** : Fichier `.env` pour la configuration

## Dépannage

### Problèmes courants

**Service ne démarre pas**
```bash
# Vérifier les logs supervisord
ansible prod -m shell -a "tail -f /var/log/supervisor/supervisord.log"

# Redémarrer supervisord
ansible prod -m shell -a "systemctl restart supervisor"
```

**Problème de permissions**
```bash
# Vérifier les permissions des dossiers
ansible prod -m shell -a "ls -la /votre-org/votre-projet/"

# Corriger les permissions si nécessaire
ansible-playbook backend.yml --tags permissions
```

**Erreur de base de données**
```bash
# Vérifier la connexion PostgreSQL
ansible prod -m shell -a "sudo -u postgres psql -l"

# Tester la migration
ansible prod -m shell -a "sudo /votre-org/votre-projet/projet-ctl migrate --dry-run"
```

---

**Support** : Pour toute question, consulter la documentation Ansible ou contacter l'équipe de TelesCoop.
