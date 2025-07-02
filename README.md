
# Template de déploiement Ansible - Django + Nuxt

Template Ansible pour automatiser le déploiement et la maintenance d'applications web basées sur Django (backend) et Vue 3 (ou Nuxt) en front-end. 

## Table des matières

- [Prérequis](#prérequis)
- [Stack technique](#stack-technique)
- [Installation et utilisation](#installation-et-utilisation)
- [Architecture des rôles](#architecture-des-rôles)
- [Commandes de maintenance](#commandes-de-maintenance)
- [Adaptation pour un nouveau projet](#adaptation-pour-un-nouveau-projet)
- [Configuration avancée](#configuration-avancée)

## Prérequis

- **Ansible 2.9+** installé sur votre machine locale
- **Python 3** sur les serveurs cibles
- **Accès SSH** configuré vers vos serveurs
- **Clé du vault Ansible** (`vault.key`) pour accéder aux variables chiffrées

## Stack technique

### Composants principaux (open-source)
- **Frontend** : Vue 3 (ou Nuxt) + Nginx
- **Backend** : Django + gunicorn + supervisord
- **Base de données** : PostgreSQL ou SQLite
- **Serveur web** : Nginx

### Services externes (optionnels)
- **Mailgun** : envoi d'emails
- **Service S3** : sauvegarde de base de données
- **Rollbar** : monitoring des erreurs

## Installation et utilisation

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

### 2. Configuration des serveurs

Modifier le fichier `hosts` :
```ini
[prod]
votre-serveur.com:22 ansible_user=ubuntu
```

### 3. Déploiement complet

```bash
# Premier déploiement (nouveau serveur)
ansible-playbook base.yml
ansible-playbook backend.yml
ansible-playbook frontend.yml

# Ou déploiement complet en une commande
ansible-playbook bootstrap.yml
```

### 4. Mises à jour

```bash
# Mise à jour du backend uniquement
ansible-playbook backend.yml

# Mise à jour du frontend uniquement
ansible-playbook frontend.yml

# Forcer la mise à jour des dépendances
ansible-playbook backend.yml -e force_update=1
```

## Architecture des rôles

### `base`
- Installation des dépendances système
- Configuration des mises à jour automatiques
- Paramétrage des logs nginx
- Configuration firewall et sécurité

**Usage** : Une seule fois par serveur, pas nécessaire pour chaque nouveau projet.

### `backend`
- Clonage du repository backend
- Installation des dépendances Python
- Configuration Django (settings, migration)
- Paramétrage supervisord pour gunicorn
- Configuration de la base de données

### `frontend`
- Clonage du repository frontend
- Installation des dépendances Node.js
- Build de l'application (static ou SSR)
- Configuration nginx
- Paramétrage supervisord (mode SSR uniquement)

## Commandes de maintenance

### Surveillance et logs
```bash
# Vérifier le statut des services
ansible prod -m shell -a "supervisorctl status"

# Consulter les logs
ansible prod -m shell -a "tail -f /var/log/supervisor/backend-*.log"
```

### Redémarrage des services
```bash
# Redémarrer tous les services
ansible prod -m shell -a "supervisorctl restart all"

# Redémarrer nginx
ansible prod -m shell -a "systemctl restart nginx"
```

## Adaptation pour un nouveau projet

### 1. Configuration des variables principales

Éditer `group_vars/all/vars.yml` :
```yaml
organization_slug: votre-org
base_project_slug: mon-projet
main_user: mon-projet
main_user_uid: 10042  # Unique par projet sur le serveur
django_project_name: mon_projet_back
backend_repo: git@github.com:votre-org/mon-projet-backend.git
frontend_repo: git@github.com:votre-org/mon-projet-frontend.git
```

### 2. Configuration des environnements

Éditer `group_vars/all/cross_env_vars.yml` pour définir :
- Ports SSH personnalisés
- Domaines publics
- Configuration réseau

### 3. Génération des secrets

```bash
# Générer une nouvelle clé de vault
bash generate_vault_key_on_first_install.sh

# Éditer les variables secrètes
ansible-vault edit group_vars/all/cross_env_vault.yml
```

Configurer dans le vault :
- Clés API (Mailgun, S3, Rollbar)
- Mots de passe de base de données
- Clés secrètes Django

### 4. Personnalisation avancée

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

### 5. Ajout d'un environnement

Pour ajouter un environnement de préproduction :

```bash
# Créer le dossier de variables
mkdir group_vars/preprod
cp group_vars/prod/vars.yml group_vars/preprod/

# Ajouter dans hosts
[preprod]
preprod.mon-projet.com:22 ansible_user=ubuntu
```

## Configuration avancée

### Personnalisation des rôles

- **Backend** : Modifier `roles/backend/templates/settings.ini.j2` pour Django
- **Frontend** : Adapter `roles/frontend/tasks/main.yml` pour d'autres frameworks
- **Nginx** : Personnaliser `roles/frontend/templates/nginx.conf.j2`

### Sécurité

- Les règles iptables sont configurées dans le rôle `base`
- Configuration RAID disponible (à adapter selon vos besoins)
- Certificats SSL automatiques via Let's Encrypt (si configuré)

### Monitoring

- Logs centralisés dans `/var/log/${votre-org}/${votre-projet}/`
- Supervision via supervisord
- Rotation automatique des logs

---

**Support** : Pour toute question, consulter la documentation Ansible ou contacter l'équipe de développement.
