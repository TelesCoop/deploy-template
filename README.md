# Outils de déploiement pour les projets TelesCoop

Ce projet contient les éléments pour mettre en place des déploiements de projets _à la TelesCoop_. En piochant à droite à gauche, il est possible de :
* déployer des projets avec la stack TelesCoop (VueJS, Django, Nuxt, Mkdocs, Hugo, …) ;
* ajouter de la CI Github pour tester et déployer ;
* déployer des environnements de dev spécifiques à une branche (et donc supprimés à la fermeture de la MR)

## Scripts Ansible

Voir [`_deploy/README.md`](/_deploy/README.md)

## Workflows Github

Voir [`_github/README.md](/_github/README.md)

## Comment déployer un projet TelesCoop

Check-list des choses à faire :
- Sélectionner un serveur TelesCoop adapté à recevoir le projet ;
- Configurer les scripts Ansible de déploiement (à partir de [`_deploy`](/_deploy/README.md)) ;
- Ajouter la clé SSH de déploiement généré par le script Ansible à Github (pour pouvoir faire un `git pull` depuis le serveur) ;
- Ajouter les entrées de monitoring côté Uptime, à savoir : monitoring du site, de la backup et du `/health-check`, pour les environnements de prod & de pré-prod.
- Mettre en place la backup automatisée ;

