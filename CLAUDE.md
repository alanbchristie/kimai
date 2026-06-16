# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Docker Compose deployment of [Kimai](https://www.kimai.org/) — there is no application source code, only `docker-compose.yml` and its environment configuration. Changes are almost always to `docker-compose.yml` or `.env`.

**See [README.md](README.md) for the architecture, configuration, usage commands, upgrade steps, and hard-reset procedure.** Read it before making changes — the editing gotchas below assume that context.

## Editing gotchas

- **`KIMAI_DB_PASSWORD` appears in two places** (`MYSQL_PASSWORD` and the app's `DATABASE_URL`) and must stay identical — update both together or the app can't reach the DB.
- **`TRUSTED_HOSTS` is pipe-separated** (`localhost|127.0.0.1|...`); a host not listed there is rejected by the app.
- **`.env` is gitignored** and must never be committed; `.env.example` is the tracked template — keep the two in sync when adding or renaming variables.

## YAML conventions

`docker-compose.yml` follows the user's house style: list dashes (`-`) align with their parent key rather than being indented, and block lists are used instead of `[]` flow style.
