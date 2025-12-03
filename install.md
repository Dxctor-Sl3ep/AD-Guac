```markdown
# Installation Guide — Apache Guacamole (Docker Compose)

This guide explains step-by-step how to deploy Apache Guacamole locally using Docker Compose.

## 1. Prerequisites

- Docker (Docker Desktop recommended on Windows)
- Docker Compose v2+ (use the `docker compose` command)
- A project folder (e.g. `C:\Guacamole\AD-Guack`)

Run commands from the project root:

```pwsh
Set-Location -Path 'C:\Guacamole\AD-Guack'
```

## 2. Important files

- `.env` — environment variables (passwords, database names)
- `docker-compose.yml` — service definitions (MariaDB, guacd, guacamole)
- `initdb.sql` — database initialization script (generated from the image)

### Minimal example of `.env`

```
MYSQL_ROOT_PASSWORD=ChangeThisRootPassword
MYSQL_DATABASE=guacamole
MYSQL_USER=guacamole
MYSQL_PASSWORD=GuacPassword
RECORDING_SEARCH_PATH=/var/lib/guacamole/recordings
```

> Do not put quotes around values in `.env`.

### Note from `docker-compose.yml` (excerpt)

Essential services: `guacamole_db` (MariaDB), `guacd`, and `guacamole`.

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

## 3. Initialize the database

1. Generate `initdb.sql` (only once):

```pwsh
docker run --rm guacamole/guacamole:latest /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```

2. Start the containers:

```pwsh
docker compose up -d
```

3. After a few seconds (wait for MariaDB to start), import the schema:

```pwsh
Get-Content .\initdb.sql | docker exec -i guacamole_db mariadb -u root -p$env:MYSQL_ROOT_PASSWORD guacamole
```

If you didn't export the variable, replace `$env:MYSQL_ROOT_PASSWORD` with the password value from your `.env`.

4. Restart Guacamole:

```pwsh
docker compose restart guacamole
```

## 4. Verify the installation

```pwsh
docker compose ps
docker compose logs guacamole --tail 100
```

Then open: `http://localhost:8043/guacamole/`

Default credentials: `guacadmin` / `guacadmin` — change them immediately.

![guacamole home](./images/acceuil-guac.png)

## 5. Large files & best practices

- Avoid committing large binary files (e.g. raw DB files). Consider adding `guacamole_db/` and `guacd_record/` to `.gitignore`.
- If you need large files in the repository, use Git LFS:

```pwsh
winget install --id Git.GitLFS -e
git lfs install
git lfs track "guacamole_db/*"
git add .gitattributes
git add guacamole_db/*
git commit -m "Track DB files with Git LFS"
git push origin main
```

If large files are already in history, use `git filter-repo` or `bfg` to remove them.

## 6. Quick troubleshooting

- `Table ... doesn't exist` → schema not imported: re-run import step.
- `Access denied` → incorrect password in `.env` or during import.
- Container crashed → `docker compose logs <service>` and fix configuration.

## 7. Reset (remove all data)

```pwsh
docker compose down -v
Remove-Item -Recurse -Force .\guacamole_db
```

---

Would you like me to:
- add `guacamole_db/` and `guacd_record/` to `.gitignore` (recommended),
- or set up Git LFS and purge large files from history?

For user management instructions, see [user guide](./user.md)
```
