# Guide d'installation — Apache Guacamole (Docker Compose)

Ce guide explique, étape par étape, comment déployer Apache Guacamole localement à l'aide de Docker Compose.

## 1. Pré-requis

- Docker (Docker Desktop recommandé sur Windows)
- Docker Compose v2+ (commande `docker compose`)
- Un dossier de projet (ex. `C:\Guacamole\AD-Guack`)

Exécutez les commandes depuis la racine du projet :

```pwsh
Set-Location -Path 'C:\Guacamole\AD-Guack'
```

## 2. Fichiers importants

- `.env` — variables d'environnement (mots de passe, noms de base)
- `docker-compose.yml` — services (MariaDB, guacd, guacamole)
- `initdb.sql` — script d'initialisation de la base (généré depuis l'image)

### Exemple minimal de `.env`

```
MYSQL_ROOT_PASSWORD=ChangezCeMotDePasse
MYSQL_DATABASE=guacamole
MYSQL_USER=guacamole
MYSQL_PASSWORD=MotDePasseGuac
RECORDING_SEARCH_PATH=/var/lib/guacamole/recordings
```

> Ne mettez pas de guillemets autour des valeurs dans `.env`.

### À noter dans `docker-compose.yml` (extrait)

Les services essentiels : `guacamole_db` (MariaDB), `guacd` et `guacamole`.

```yaml
version: '3.8'
services:
  guacamole_db:
    image: mariadb:10.6
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ./guacamole_db:/var/lib/mysql

  guacd:
    image: guacamole/guacd:latest

  guacamole:
    image: guacamole/guacamole:latest
    depends_on:
      - guacamole_db
      - guacd
    ports:
      - "8043:8080"
    environment:
      GUACD_HOSTNAME: guacd
      MYSQL_HOSTNAME: guacamole_db
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ./guacd_record:/var/lib/guacamole/recordings

networks: {}
```

## 3. Initialiser la base de données

1. Générer `initdb.sql` (une seule fois) :

```pwsh
docker run --rm guacamole/guacamole:latest /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```

2. Démarrer les conteneurs :

```pwsh
docker compose up -d
```

3. Après quelques secondes (attendre que MariaDB démarre), importer le schéma :

```pwsh
Get-Content .\initdb.sql | docker exec -i guacamole_db mariadb -u root -p$env:MYSQL_ROOT_PASSWORD guacamole
```

Si vous n'avez pas exporté la variable, remplacez `$env:MYSQL_ROOT_PASSWORD` par la valeur du mot de passe du `.env`.

4. Redémarrer Guacamole :

```pwsh
docker compose restart guacamole
```

## 4. Vérifier l'installation

```pwsh
docker compose ps
docker compose logs guacamole --tail 100
```

Accédez ensuite à : `http://localhost:8043/guacamole/`

Identifiants par défaut : `guacadmin` / `guacadmin` — changez-les immédiatement.

## 5. Gros fichiers & bonnes pratiques

- Évitez de versionner les fichiers volumineux (ex. données brutes de la base). Ajoutez `guacamole_db/` et `guacd_record/` dans `.gitignore` si nécessaire.
- Si vous devez stocker de gros fichiers dans le dépôt, utilisez Git LFS :

```pwsh
winget install --id Git.GitLFS -e
git lfs install
git lfs track "guacamole_db/*"
git add .gitattributes
git add guacamole_db/*
git commit -m "Track DB files with Git LFS"
git push origin main
```

Si des gros fichiers sont déjà dans l'historique, utilisez `git filter-repo` ou `bfg` pour les purger.

## 6. Dépannage rapide

- `Table ... doesn't exist` → le schéma n'a pas été importé : relancez l'étape d'import.
- `Access denied` → mot de passe incorrect dans `.env` ou lors de l'import.
- Conteneur planté → `docker compose logs <service>` puis corrigez la configuration.

## 7. Réinitialiser (tout supprimer)

```pwsh
docker compose down -v
Remove-Item -Recurse -Force .\guacamole_db
```

---

Souhaitez-vous que je :
- ajoute `guacamole_db/` et `guacd_record/` dans `.gitignore` (recommandé),
- ou configure Git LFS et purger les gros fichiers de l'historique ?
Absolument ! Voici le guide d'installation complet d'Apache Guacamole en utilisant Docker Compose, formaté en Markdown pour votre fichier README.md.

🥑 Apache Guacamole : Installation avec Docker Compose

Ce guide explique comment déployer une stack complète Apache Guacamole (serveur d'accès distant) en utilisant Docker Compose. Cette stack inclut le frontend Guacamole, le démon guacd et une base de données MariaDB pour l'authentification.

🚀 1. Prérequis

Assurez-vous que les éléments suivants sont installés sur votre système :

    Docker : Installé et en fonctionnement (incluant Docker Desktop pour Windows/macOS).

    Docker Compose : Version 2.0 ou supérieure (utilisez la commande docker compose).

    Dossier de Projet : Créez un dossier unique pour tous vos fichiers (ex: C:\Guacamole\AD-Guack).

2. ⚙️ Fichiers de Configuration

Placez les deux fichiers suivants à la racine de votre dossier de projet.

2.1. Fichier .env (Variables d'Environnement)

Ce fichier définit les mots de passe et les noms de base de données.

    ⚠️ IMPORTANT : N'utilisez JAMAIS de guillemets doubles ("") autour des valeurs pour éviter les erreurs d'authentification liées à la lecture des variables par MariaDB. Utilisez des mots de passe forts.

Extrait de code

# --- Variables pour MariaDB (guacamole_db) ---
MYSQL_ROOT_PASSWORD=VotreMotDePasseRoot
MYSQL_DATABASE=guacamole
MYSQL_USER=guacamole
MYSQL_PASSWORD=VotreMotDePasseGuac

# --- Variables pour Guacamole (frontend) ---
RECORDING_SEARCH_PATH=/var/lib/guacamole/recordings

2.2. Fichier docker-compose.yml

Ce fichier définit les trois services nécessaires : guacamole_db (MariaDB), guacd (le proxy) et guacamole (le frontend web).
YAML

version: '3.8'

services:
  guacamole_db:
    container_name: guacamole_db
    image: mariadb:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ./guacamole_db:/var/lib/mysql # Persistance des données de la DB
    networks:
      - guac_net

  guacd:
    container_name: guacd
    image: guacamole/guacd:latest
    restart: always
    networks:
      - guac_net

  guacamole:
    container_name: guacamole
    image: guacamole/guacamole:latest
    restart: always
    depends_on:
      - guacamole_db
      - guacd
    ports:
      - "8043:8080" # Accès au frontend sur le port 8043 de l'hôte
    environment:
      GUACD_HOSTNAME: guacd
      MYSQL_HOSTNAME: guacamole_db
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      RECORDING_SEARCH_PATH: ${RECORDING_SEARCH_PATH}
    volumes:
      - ./guacd_record:/var/lib/guacamole/recordings # Persistance des enregistrements
    networks:
      - guac_net

networks:
  guac_net:
    driver: bridge

3. 📝 Initialisation du Schéma SQL

La base de données doit être initialisée avec la structure de tables spécifique à Guacamole.

3.1. Génération du fichier initdb.sql

Générez le script d'initialisation en utilisant l'outil intégré à l'image Guacamole :
PowerShell

docker run --rm guacamole/guacamole:latest /opt/guacamole/bin/initdb.sh --mysql > initdb.sql

3.2. Démarrage de la Stack (Première Fois)

Démarrez les conteneurs. MariaDB est créé et utilise les mots de passe définis dans le .env.
PowerShell

docker compose up -d

3.3. Importation du Schéma (Création des Tables)

Attendez 10-15 secondes pour que MariaDB soit stable. Ensuite, injectez le schéma SQL.

    💡 NOTE : En PowerShell, nous utilisons Get-Content pour lire le fichier et le passer au conteneur via le pipe (|).

Remplacez VotreMotDePasseRoot par la valeur que vous avez définie dans le .env.
PowerShell

Get-Content .\initdb.sql | docker exec -i guacamole_db mariadb -u root -pVotreMotDePasseRoot guacamole

3.4. Redémarrage Final de Guacamole

Le conteneur Guacamole doit redémarrer pour se connecter à la base de données maintenant remplie de tables :
PowerShell

docker compose restart guacamole

4. ✅ Connexion

Votre installation Guacamole est prête.

    Vérifiez le statut :
    PowerShell

    docker compose ps

    (Les trois conteneurs doivent être Up).

    Accédez au web : Ouvrez votre navigateur à l'adresse : http://localhost:8043/guacamole/
    [acceuil de gucamole](./images/acceuil-guac.png)

    Identifiants par défaut :

        Utilisateur : guacadmin

        Mot de passe : guacadmin

    SÉCURITÉ : Changez immédiatement le mot de passe de l'utilisateur guacadmin après la première connexion.

🛑 5. Dépannage (Errors Courantes)

Problème	Commande de Diagnostic	Solution
Chargement infini ou Page d'erreur	docker compose logs guacamole	L'erreur la plus courante est Table 'guacamole.guacamole_user' doesn't exist. Cela signifie que l'étape 3.3 (Importation du Schéma) a échoué. Exécutez-la à nouveau et assurez-vous que vous utilisez le bon mot de passe root.
Access denied (dans les logs)	docker exec -it guacamole_db mariadb -u root -p[MDP_ROOT]	Le mot de passe dans votre commande ou dans le .env est incorrect. Si vous soupçonnez une incohérence, effectuez un nettoyage complet : docker compose down -v et recommencez les étapes 3.2 à 3.4 avec un mot de passe très simple dans le .env.
ParserError (< est réservé)	N/A	Vous utilisez la mauvaise syntaxe pour PowerShell. Utilisez **`Get-Content .\initdb.sql
Nettoyage complet	N/A	Pour supprimer toutes les données de MariaDB et recommencer à zéro : docker compose down -v.