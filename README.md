🚀 Apache Guacamole sur Docker : Pourquoi ce Projet ?

Ce projet déploie Apache Guacamole, un client de passerelle de bureau à distance sans client (clientless remote desktop gateway), en utilisant Docker Compose.

L'objectif principal est de fournir une solution d'accès distant sécurisée et facile à gérer, tout en tirant parti des avantages de la conteneurisation.

🥑 1. Intérêt d'Apache Guacamole

Guacamole est souvent appelé un "VPN pour l'accès distant" car il centralise toutes vos connexions (RDP, VNC, SSH) et les rend accessibles via un simple navigateur web, sans nécessiter l'installation de logiciel client spécifique.
Caractéristique	Bénéfice Principal
Accès Web (Clientless)	Accédez à n'importe quel serveur (Windows, Linux, etc.) depuis n'importe quel appareil (PC, tablette) avec seulement un navigateur web. Aucun logiciel tiers n'est requis.
Centralisation	Tous les protocoles (RDP, VNC, SSH) sont gérés par Guacamole. Un seul port à ouvrir sur le pare-feu externe (souvent 80 ou 443) au lieu d'ouvrir les ports pour chaque service distant (3389, 22, 5900).
Sécurité Accrue	L'authentification passe par Guacamole, agissant comme un point de contrôle unique et sécurisé.
Journalisation/Audit	Guacamole enregistre les sessions (vidéo) et les événements, offrant une traçabilité complète de qui a accédé à quoi et quand.

🐳 2. Intérêt de la Conteneurisation (Docker Compose)

Déployer Guacamole via Docker Compose résout les problèmes de dépendances, d'environnement et de gestion des services complexes.

2.1. Isolation et Environnement

    Zéro conflit de dépendances : Guacamole nécessite Java et divers outils de compilation (guacd). Docker inclut toutes ces dépendances dans les conteneurs.

    Environnement garanti : Le service fonctionnera de manière identique, que vous le lanciez sur Windows, macOS ou Linux.

2.2. Facilité de Déploiement et d'Évolutivité

    Déploiement en une seule commande : Le fichier docker-compose.yml définit l'ensemble de l'architecture (guacamole, guacd, et mariadb). Le lancement se fait via un simple docker compose up -d.

    Architecture modulaire : Chaque service est isolé dans son propre conteneur :

        guacamole_db : Stocke les utilisateurs et les connexions.

        guacd : Le démon qui gère les protocoles (RDP/VNC/SSH).

        guacamole : Le frontend web.

    Persistance des données : Les volumes Docker sont utilisés pour garantir que les données de la base de données et les enregistrements de sessions persistent même si les conteneurs sont supprimés ou mis à jour.

2.3. Gestion et Maintenance

    Configuration centralisée : Toutes les variables critiques (mots de passe, noms de bases de données) sont gérées via le fichier .env.

    Mise à jour simplifiée : Pour mettre à jour Guacamole vers la dernière version, il suffit de modifier l'image dans le docker-compose.yml et de relancer la stack (docker compose pull puis docker compose up -d).

📖 3. Démarrage du Projet

Toutes les étapes pour lancer cette stack sont détaillées dans le fichier install.md.