# neo4j service

A submodule of [rndexp-art/rndexpart](https://github.com/rndexp-art/rndexpart) — exposes `neo4j.rndexp.art` (production) / `neo4j.rndexp.localhost` (dev).

## Files
- `compose.fragment.yml` — included by the gateway's compose when this service is enabled.
- `caddy.fragment` — concatenated into the gateway's Caddyfile.
- `.env.example` — required env vars.

## Local dev
This service runs as part of the gateway. From the gateway repo root:

```sh
tools/rndexp service enable neo4j --env local
tools/rndexp up
```

Then visit `https://neo4j.rndexp.localhost`.

## Deploy
Push to the `production` branch — the workflow in `.github/workflows/deploy.yml` dispatches the gateway, which redeploys with the latest submodule SHA.

```sh
# from this submodule's directory
git push origin main:production
```

## Internal port
Listens on **7474**. The Caddy snippet reverse-proxies `127.0.0.1:7474`.
