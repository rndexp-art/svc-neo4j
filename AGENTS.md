# AGENTS.md — svc-neo4j

This is a service submodule of the [rndexpart gateway](https://github.com/rndexp-art/rndexpart). Read the gateway's [AGENTS.md](https://github.com/rndexp-art/rndexpart/blob/main/AGENTS.md) for the overall architecture and tooling.

## What this service is
- Neo4j Community Edition (`neo4j:trixie`) at `neo4j.rndexp.art` (production), `neo4j.rndexp.localhost` (dev).
- HTTP browser on container port **7474**, Bolt on **7687** — both bound to `127.0.0.1` only on the host. Caddy reverse-proxies HTTP; the `:7473` site block lets Neo4j Browser reach the Neo4j HTTPS API for bolt-over-websocket negotiation.
- Data lives in the docker volume `neo4j_data`. **Do not** convert to a bind mount without a migration plan — `--remove-orphans` would then nuke it on a path mismatch.

## What lives here
- `compose.fragment.yml` — neo4j service definition. Not standalone.
- `caddy.fragment` — two site blocks (`:443` for the browser, `:7473` for the secure HTTP API used by neo4j-browser).
- `.env.example` — `NEO4J_USER` / `NEO4J_PASSWORD` (set in the gateway's `.env`).

## Environment variables
The gateway's `.env` (or `/opt/rndexpart/.env` in prod, written by GH Actions from secrets) must define:
- `NEO4J_USER`
- `NEO4J_PASSWORD`

Compose interpolates `${NEO4J_USER}` / `${NEO4J_PASSWORD}` from there. The `$$` in the healthcheck is a literal `$` after compose interpolation — do not "fix" it; the healthcheck splits `NEO4J_AUTH` at runtime via shell parameter expansion (`${NEO4J_AUTH%%/*}` / `${NEO4J_AUTH#*/}`). We can't pass `NEO4J_USER`/`NEO4J_PASSWORD` directly into the container because the neo4j image interprets every `NEO4J_*` env var as a config setting and refuses to start under strict validation.

## How deploys work
1. Push to `main` for development; merge to `production` to deploy.
2. `.github/workflows/deploy.yml` fires on push to `production`. It calls `repository_dispatch` against the gateway repo with `event_type=service-updated`, `client_payload.service=neo4j`.
3. The gateway's `deploy-gateway.yml` workflow handles the rest: bumps the submodule pin, SSHes to the VPS, re-renders, and `docker compose up -d`.

## Conventions / gotchas
- All hostnames in `caddy.fragment` use the production form (`neo4j.rndexp.art`). The gateway's renderer rewrites them for local dev.
- Caddy binds port `:7473` because of the second site block. Don't remove it — it's how Neo4j Browser reaches the HTTPS API for bolt negotiation.
- Don't add a `version:` key to `compose.fragment.yml`; Compose v2 doesn't need it and `include:` handling is fussy about it.

## Adding a tool / agent task
Don't add a separate Python tooling stack here. Use the gateway's `tools/rndexp` from the parent repo for cross-cutting actions (deploy, restart, secrets).
