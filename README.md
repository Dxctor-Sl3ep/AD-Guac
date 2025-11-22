**Guacamole — Installation avec Docker Compose**

Ce dépôt contient une configuration Docker Compose pour déployer Apache Guacamole (frontend), `guacd` (le démon proxy) et une base MariaDB.

**Prérequis**
- **Docker**: installé et en fonctionnement.
- **Docker Compose** (intégré à Docker Desktop ou `docker compose` v2).
- **PowerShell** (Windows) si vous suivez les commandes ci‑dessous.

**Fichiers importants**
- `docker-compose.yml`: configuration des services (`guacamole`, `guacd`, `guacamole_db`).
- `.env`: variables d'environnement (DB et chemin des enregistrements).
- `initdb.sql`: script d'initialisation du schéma Guacamole (présent dans le dépôt).
- `guacamole_db/`, `guacd_record/`: volumes de données (persistants).

**Exemple de `.env`** (ajustez si besoin) :
```
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=guacamole
MYSQL_USER=root
MYSQL_PASSWORD=root
RECORDING_SEARCH_PATH=/var/lib/guacamole/recordings
```

**Démarrage rapide**
1. Placer le fichier `.env` à la racine (même dossier que `docker-compose.yml`).
2. Démarrer la stack en arrière‑plan :
```powershell
docker compose up -d
```
3. Vérifier que les conteneurs tournent :
```powershell
docker compose ps
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

**Initialiser la base de données (si nécessaire)**
Le dépôt contient `initdb.sql`. Si la base n'est pas encore initialisée, importez ce script dans le conteneur MariaDB.

Commande PowerShell (depuis le dossier du projet) :
```powershell
# adapte le mot de passe si différent
Get-Content .\initdb.sql | docker exec -i guacamole_db mysql -u root -proot guacamole
```
Remplacez `root` par les valeurs définies dans votre `MYSQL_ROOT_PASSWORD` / `MYSQL_USER` / `MYSQL_PASSWORD` si elles sont différentes.

Après l'import, redémarrez Guacamole :
```powershell
docker compose restart guacamole
```

**Accès web**
- Ouvrez `http://localhost:8080/guacamole/` depuis un navigateur sur la machine hôte. Ou
- `http://localhost:8080`
- Si vous exposez sur un autre hôte, remplacez `localhost` par l'adresse IP ou le nom d'hôte.

Note: certains schémas `initdb.sql` incluent un utilisateur admin par défaut (`guacadmin/guacadmin`). Si ce n'est pas le cas, créez un utilisateur administrateur via le script SQL fourni ou via la base.

**Commandes utiles & dépannage**
- Voir les logs en temps réel :
```powershell
docker compose logs -f guacamole
docker compose logs -f guacd
docker compose logs -f guacamole_db
```
- Tester l'URL localement :
```powershell
try { Invoke-WebRequest -Uri http://localhost:8080/guacamole/ -UseBasicParsing -TimeoutSec 10 | Select-Object StatusCode, StatusDescription } catch { $_.Exception.Message }
# ou
curl.exe http://localhost:8080/guacamole/
```
- Vérifier l'écoute du port 8080 :
```powershell
Get-NetTCPConnection -LocalPort 8080 | Format-Table -AutoSize
```
- Créer une règle pare‑feu (exécuter PowerShell en Administrateur) :
```powershell
New-NetFirewallRule -DisplayName "Allow Guacamole 8080" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8080
```

**Cas d'erreurs fréquentes et solutions**
- Erreur de connexion à la base (`Access denied`, `Unknown database`): vérifiez les variables dans ` .env`, importez `initdb.sql` et redémarrez la stack.
- `guacamole` ne se connecte pas à `guacd`: vérifiez que le service `guacd` est `UP` et que la variable `GUACD_HOSTNAME` vaut `guacd` (nom de service docker-compose).
- Problèmes de volumes sur Windows (OneDrive): évitez d'utiliser des dossiers synchronisés (OneDrive) pour les volumes de base de données — privilégiez un dossier local (ex. `C:\data\guacamole`) pour éviter des problèmes de verrous/permissions.

**Arrêt et clean (si besoin)**
- Arrêter la stack :
```powershell
docker compose down
```
- Supprimer aussi les volumes (perd les données persistantes) :
```powershell
docker compose down -v
```

**Si vous avez besoin d'aide**
- Copiez/collez ici la sortie de :
  - `docker compose ps`
  - `docker compose logs guacamole --tail 200`
  - sortie de `Invoke-WebRequest` ou le message d'erreur du navigateur
- Je vous aiderai à interpréter les logs et proposer une correction précise.

---
Fait par l'équipe projet — guide minimal d'installation et dépannage pour Guacamole via Docker Compose.
1. Dans ton dossier C:\guacamole, crée un fichier docker-compose.yml 
Faire attention à ne pas être dans un dossier : ..\OneDrive\
2. Ouvrir se fichier avec vscode et mettre :

```version: "3.8"

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
      - ./guacamole_db:/var/lib/mysql
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
      - "8080:8080"
    environment:
      GUACD_HOSTNAME: guacd
      MYSQL_HOSTNAME: guacamole_db
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      RECORDING_SEARCH_PATH: ${RECORDING_SEARCH_PATH}
    volumes:
      - ./guacd_record:/var/lib/guacamole/recordings
    networks:
      - guac_net

networks:
  guac_net:
```

3. Créer un fichier .env

```
MYSQL_ROOT_PASSWORD=ton_root_password
MYSQL_DATABASE=guacamole
MYSQL_USER=guacamole
MYSQL_PASSWORD=ton_password_guac
RECORDING_SEARCH_PATH=/var/lib/guacamole/recordings
<!-- éviter root root. cela cassera miaradb -->
```

4. Inistialiser la base de donée Guacamole
Dans C:\Guacamole
4.1 
```
docker run --rm guacamole/guacamole:latest /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```

4.2
```
docker cp initdb.sql guacamole_db:/initdb.sql
```

4.3
```
ls
```
Tu devrais avoir :
    docker-compose.yml
    .env
    initdb.sql

4.4
```
docker compose up -d
```

4.5
```
docker ps
```
Tu dois voir au moins :
    guacamole_db
    guacd
    guacamole

4.6 Copier le fichier initdb.sql dans MariaDB
```
docker cp initdb.sql guacamole_db:/initdb.sql
```
Tu dois obtenir :
    Successfully copied ...

4.7 Entrer dans le conteneur MariaDB
```
docker exec -it guacamole_db bash
```
Tu arrives dans le shell linux

4.8 Importer la base Guacamole
```
cat /initdb.sql | mariadb -u root -p guacamole
```
Entre ton password de MYSQL_ROOT_PASSWORD def dans .env

5. 
```
exit
```

6.
```
docker restart guacamole
```

7. Accede à Guacamole
http://localhost:8080/guacamole
id par défaut :
    guacadmin
    guacadmin