# svc-neo4j

Neo4j Community Edition for `neo4j.rndexp.art`. A submodule of the [rndexpart gateway](https://github.com/rndexp-art/rndexpart).

- Image: `neo4j:trixie`
- HTTP browser: `127.0.0.1:7474` → `https://neo4j.rndexp.art`
- Bolt: `127.0.0.1:7687`
- Data: docker volume `neo4j_data` (persistent across deploys)

## Local dev

```sh
# from the gateway repo root
tools/rndexp render --env local
tools/rndexp up
# → https://neo4j.rndexp.localhost (login with $NEO4J_USER / $NEO4J_PASSWORD from gateway .env)
```

## Production deploy

Push to the `production` branch of this repo. The workflow at `.github/workflows/deploy.yml` `repository_dispatch`es the gateway, which redeploys with this submodule's new SHA.

## Required env vars (set in gateway's .env)

- `NEO4J_USER`
- `NEO4J_PASSWORD`
