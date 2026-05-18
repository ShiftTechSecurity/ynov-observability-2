# Travaux Pratiques Grafana Loki & Grafana Alloy Docker - Version Étudiant

## Énoncés des Exercices Pratiques

Ce cahier d'exercices comprend 10 ateliers pratiques conçus pour vous mener de la découverte de Grafana Loki jusqu'aux architectures multi-tenant avancées, en utilisant Grafana Alloy comme agent de collecte unique.

## Module 1 : Les Fondamentaux (4 Exercices)

### Exercice 1 : Déploiement mono-nœud et vérification du flux de logs

- Objectif : Déployer une instance fonctionnelle de Grafana, Loki, et Grafana Alloy.
- Tâche : Créer un fichier docker-compose.yml incluant Loki, Grafana et Alloy. Configurer Grafana Alloy pour collecter les fichiers de logs locaux ou de conteneurs et les acheminer vers Loki. Connecter Grafana à Loki et valider le flux via l'onglet Explore.

### Exercice 2 : Comprendre les labels et le mécanisme de Relabeling

- Objectif : Assimiler la gestion des labels par Grafana Alloy et l'indexation par Loki.
- Tâche : Modifier la configuration Alloy pour supprimer un label natif (ex. container_id ou filename). Ajouter un label statique 'environment=development' et un label dynamique 'loglevel' extrait des métadonnées du flux de logs. Vérifier le résultat dans Grafana.

### Exercice 3 : Rotation des logs et découverte dynamique

- Objectif : Garantir une collecte continue et sans doublons lors des événements de rotation de fichiers.
- Tâche : Configurer Alloy pour suivre un répertoire (/var/log/apps/*.log). Simuler l'activité d'une application, puis déclencher manuellement une rotation de fichiers (renommage et création d'un nouveau fichier vide). Confirmer qu'Alloy conserve sa position de lecture sans perte.

### Exercice 4 : Pipelines Alloy et Parsing à la source

- Objectif : Transformer, filtrer et enrichir les structures de logs directement au niveau de la couche de collecte.
- Tâche : Produire des logs fictifs au format JSON. Configurer un composant 'loki.process' dans Grafana Alloy pour parser ce JSON, extraire le champ 'user_id' en tant que label temporaire, et supprimer toutes les lignes de niveau 'debug' avant l'envoi à Loki.

## Module 2 : Maîtrise de LogQL (2 Exercices)

### Exercice 5 : Filtrage avancé et requêtes de formatage

- Objectif : Maîtriser les filtres de ligne LogQL et les fonctions d'expression de formatage de sortie.
- Tâche : Écrire une requête LogQL filtrant les lignes contenant 'HTTP/1.1' mais excluant 'status=200'. Parser ensuite ces logs (json ou logfmt) et utiliser 'line_format' pour réécrire l'affichage sous la forme : '[NIVEAU] -> MESSAGE'.

### Exercice 6 : Métriques LogQL et expressions d'alerte

- Objectif : Convertir des données textuelles de logs en métriques de séries temporelles pour le monitoring.
- Tâche : Créer une requête de métrique calculant le taux par seconde des logs d'erreurs sur une fenêtre de 5 minutes, groupé par service. Formuler ensuite une expression d'alerte Grafana qui se déclenche si ce taux dépasse 5 erreurs/sec pendant plus de 2 minutes.

## Module 3 : Architectures Avancées (2 Exercices)

### Exercice 7 : Métadonnées structurées (Structured Metadata)

- Objectif : Associer des données à forte cardinalité (ex. trace_id) sans impacter négativement l'index de Loki.
- Tâche : Configurer Grafana Alloy pour extraire un champ unique et l'injecter en tant que 'structured metadata'. Confirmer dans Grafana Explore qu'il est lié à la ligne de log sans surcharger l'index global, puis l'interroger en LogQL.

### Exercice 8 : Rétention granulaire, purge et sharding

- Objectif : Implémenter des cycles de vie de données sélectifs et appliquer des purges ciblées (conformité RGPD).
- Tâche : Configurer une rétention globale de 7 jours dans Loki, avec un override à 30 jours pour le flux '{service="audit"}'. Exécuter ensuite une requête de purge via l'API REST de Loki pour éliminer des données personnelles sur une plage horaire précise.

## Module 4 : Deep Dive & Multi-Tenancy (2 Exercices)

### Exercice 9 : Isolation multi-tenant avec injection d'en-têtes HTTP

- Objectif : Configurer un cluster Loki partagé mais hermétiquement cloisonné pour différents locataires (tenants).
- Tâche : Activer le multi-tenant dans Loki. Configurer deux pipelines distincts dans Alloy acheminant respectivement les logs avec l'en-tête HTTP 'X-Scope-OrgID: tenant-alpha' et 'X-Scope-OrgID: tenant-beta'. Connecter deux datasources séparées dans Grafana pour valider l'isolation.

### Exercice 10 : Architecture microservices et routage par passerelle (Gateway)

- Objectif : Évoluer d'un mode monolithique vers une topologie Loki distribuée et scalable.
- Tâche : Séparer le déploiement Loki en composants de Lecture (Read) et Écriture (Write). Installer un proxy inverse (Nginx/Envoy) agissant comme Gateway pour aiguiller les requêtes (/loki/api/v1/push vers les Writers, les requêtes de lecture vers les Readers). Rediriger Alloy vers cette Gateway.
