# Snipe‑IT — Docker Compose Setup 🔧

**Short summary:** This folder contains a minimal production-ready Docker Compose setup for Snipe‑IT using a Snipe‑IT `app` container and a MariaDB `db` container defined in `docker-compose.yml`.

---

## Contents
- `docker-compose.yml` — defines the `app` (image: `snipe/snipe-it`) and `db` (image: `mariadb:11.4.7`) services and two named volumes: `db_data` and `storage`.

---

## Requirements ✅
- Docker and Docker Compose (v2+ / docker compose)
- A `.env` file placed next to `docker-compose.yml` with required variables (see below)
- Enough disk space and proper permissions for Docker volumes

---

## Example `.env`
Create a `.env` file (do NOT commit it to source control):

```
# Application
APP_VERSION=latest
APP_PORT=8000

# Database
DB_DATABASE=snipeit
DB_USERNAME=snipeuser
DB_PASSWORD=strong_password_here
MYSQL_ROOT_PASSWORD=another_strong_root_pass
```

Notes:
- `APP_VERSION` and `APP_PORT` have default fallbacks in `docker-compose.yml` if omitted.
- Keep secrets out of source control and consider a secrets manager for production.

---

## Persistent Data
- `db_data` — MariaDB data directory (persistent DB storage)
- `storage` — Snipe‑IT storage for uploads and attachments

To remove volumes (destroy data): `docker volume rm <volume_name>`

---

## Common Commands 🔁
- Start (detached):
  - `docker compose up -d`
- Stop and remove containers:
  - `docker compose down`
- View logs:
  - `docker compose logs -f app`
  - `docker compose logs -f db`
- Open a shell in the running app container:
  - `docker compose exec app /bin/sh`
- Connect to the DB container:
  - `docker compose exec db mysql -u${DB_USERNAME} -p${DB_PASSWORD} ${DB_DATABASE}`

---

## Healthcheck & Startup Notes ⚠️
- The `db` service has a healthcheck. The `app` service depends on `db` being healthy.
- If the app fails to start, check `docker compose logs db` for DB errors and confirm the `.env` credentials.

---

## Backups & Restore 💾
- Backup DB (example):
  - `docker compose exec db mysqldump -u${DB_USERNAME} -p${DB_PASSWORD} ${DB_DATABASE} > snipe_backup.sql`
- Restore: import the SQL file into the DB using the `mysql` client inside the `db` container.
- Backup `storage` by copying files from the `app` container or mounting the volume into a helper container and archiving the contents.

---

## Security & Best Practices 🔒
- Use strong passwords and rotate credentials regularly.
- Do not expose `app` directly to the public internet without TLS + reverse proxy (e.g., Traefik or Nginx).
- Add a `.env.example` (without secrets) for onboarding and add `.env` to `.gitignore`.

---

## Troubleshooting
- "App won't start": check `docker compose logs app` and ensure `db` is healthy.
- "Database connection refused": verify `.env` DB credentials and check `docker compose logs db`.
- "Port in use": change `APP_PORT` in `.env` or stop the conflicting service.
---

