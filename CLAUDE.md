# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Docker Compose deployment of [Kimai](https://www.kimai.org/) (a self-hosted time-tracking app). There is no application source code here — only the compose stack and its environment configuration. Changes are almost always to `docker-compose.yml` or `.env`.

## Architecture

Two services defined in `docker-compose.yml`:

- **`database`** — MariaDB 11. Started first; `kimai` waits on its healthcheck (`condition: service_healthy`) before booting. Data persists in the `kimai_db` named volume.
- **`kimai`** — the Kimai app (`kimai/kimai2`), exposed on host port `8001`. Connects to `database` via the `DATABASE_URL` connection string. App data persists in the `kimai_data` named volume.

Both the DB and the app read `KIMAI_DB_PASSWORD` — it must be identical on each side (it appears in `MYSQL_PASSWORD` and in the app's `DATABASE_URL`), so update both together.

`TRUSTED_HOSTS` is pipe-separated (e.g. `localhost|127.0.0.1|192.168.1.50`); add the host or LAN IP it will be reached on, or the app rejects the request.

## Configuration

- Secrets come from environment variables resolved by Compose. `.env` (gitignored) overrides; `.env.example` is the template — copy it to `.env` and set real values.
- Every variable has a fallback default in `docker-compose.yml`, so `.env` is optional for local use but required for any real deployment (the defaults are not secret).

## Common commands

```bash
docker compose up -d          # start the stack in the background
docker compose logs -f kimai  # follow app logs
docker compose ps             # service status / health
docker compose down           # stop (volumes are preserved)
docker compose pull           # fetch newer images after bumping a tag
```

Image versions are pinned by tag (e.g. `kimai/kimai2:2.60.0`); upgrade by editing the tag, then `pull` and `up -d`.

## Hard reset

To wipe all state and start from a clean install — **this permanently deletes all time-tracking data and the database, including the admin account**:

```bash
docker compose down -v        # stop and DELETE the kimai_db + kimai_data volumes
docker compose up -d          # recreate; DB re-initialises and the app re-seeds the admin user
```

`down -v` removes the named volumes; everything else is reconstructed from the images and environment config on the next `up`. The admin account is recreated from `ADMINMAIL` / `KIMAI_ADMIN_PASSWORD`, so set those in `.env` before bringing the stack back up. Plain `docker compose down` (no `-v`) preserves the volumes and is not a reset.

## YAML conventions

`docker-compose.yml` follows the user's house style: list dashes (`-`) align with their parent key rather than being indented, and block lists are used instead of `[]` flow style.
