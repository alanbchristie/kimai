# Kimai

A Docker Compose deployment of [Kimai](https://www.kimai.org/), a self-hosted time-tracking application. This repository contains only the compose stack and its environment configuration — no application source code.

## Architecture

Two services are defined in `docker-compose.yml`:

- **`database`** — MariaDB 11. Started first; `kimai` waits on its healthcheck (`condition: service_healthy`) before booting. Data persists in the `kimai_db` named volume.
- **`kimai`** — the Kimai app (`kimai/kimai2`), exposed on host port `8001`. Connects to `database` via the `DATABASE_URL` connection string. App data persists in the `kimai_data` named volume.

Image versions are pinned by tag (e.g. `kimai/kimai2:2.60.0`).

## Configuration

Secrets come from environment variables resolved by Compose:

- `.env` (gitignored) supplies real values and is loaded automatically. `.env.example` is the template — copy it and fill in secrets:

  ```bash
  cp .env.example .env
  ```

- Every variable has a fallback default in `docker-compose.yml`, so `.env` is optional for local experimentation but **required for any real deployment** (the defaults are not secret).

Two settings need particular care:

- **`KIMAI_DB_PASSWORD`** is read by both the database and the app — it appears in `MYSQL_PASSWORD` and in the app's `DATABASE_URL`. Update both together; if they drift, the app cannot connect.
- **`TRUSTED_HOSTS`** is pipe-separated (e.g. `localhost|127.0.0.1|192.168.1.50`). Add the host or LAN IP the app will be reached on, or it rejects the request.

## Usage

```bash
docker compose up -d          # start the stack in the background
docker compose logs -f kimai  # follow app logs
docker compose ps             # service status / health
docker compose down           # stop (volumes are preserved)
```

Once running, Kimai is available at <http://localhost:8001>. The initial admin account is created from `ADMINMAIL` / `KIMAI_ADMIN_PASSWORD`.

## Upgrading

Edit the image tag in `docker-compose.yml` (e.g. bump `kimai/kimai2:2.60.0`), then:

```bash
docker compose pull           # fetch the newer images
docker compose up -d          # recreate the containers
```

## Hard reset

To wipe all state and start from a clean install — **this permanently deletes all time-tracking data and the database, including the admin account**:

```bash
docker compose down -v        # stop and DELETE the kimai_db + kimai_data volumes
docker compose up -d          # recreate; DB re-initialises and the app re-seeds the admin user
```

`down -v` removes the named volumes; everything else is reconstructed from the images and environment config on the next `up`. The admin account is recreated from `ADMINMAIL` / `KIMAI_ADMIN_PASSWORD`, so make sure those are set before bringing the stack back up. Plain `docker compose down` (no `-v`) preserves the volumes and is **not** a reset.
