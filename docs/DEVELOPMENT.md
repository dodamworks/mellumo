# Development setup

## Requirements

Use Docker with Compose v2 and sufficient memory for the application plus PostgreSQL, Redis and Temporal. The Dockerfile pins Node 22.23.4 and pnpm 10.6.1. Native source work should use the same versions; use pnpm only. The larger upstream `docker-compose.yaml` remains as a reference, not the Mellumo quickstart.

## Build this source locally

1. Copy `.env.mellumo.example` to `.env.mellumo`.
2. Generate three independent secrets with `openssl rand -hex 32` and set JWT_SECRET, POSTGRES_PASSWORD and TEMPORAL_POSTGRES_PASSWORD. Keep passwords hexadecimal in this example URL configuration. Never commit the private file.
3. Add your own SNS app credentials and optional AI key to `.env.mellumo`. Existing variable names are listed in `.env.example`; add only values needed for your selected provider.
4. Run:

```sh
docker compose --env-file .env.mellumo -f compose.mellumo.yaml config --quiet
docker compose --env-file .env.mellumo -f compose.mellumo.yaml up -d --build
```

The image is built from this checkout with `Dockerfile.dev`, retaining the upstream application. On first build, dependencies and assets are compiled; this may take time and memory.

Open http://localhost:4210. Temporal UI is at http://localhost:4211. Both ports bind to loopback; DB, Redis and Temporal RPC are not published by this configuration. To change the web port, update both MELLUMO_PORT and MELLUMO_APP_URL. If changing Temporal UI's port, update its CORS origin too.

Before changing provider keys/URLs, use the self-hosted [provider documentation](https://docs.postiz.com/self-host/providers/overview). Real SNS connections need platform-specific app registrations and redirects. AI keys and separately licensed editors are optional; without a key the corresponding AI functionality will not work. The baseline has a known unconfigured Copilot request path that can time out.

## Observe and stop

```sh
docker compose --env-file .env.mellumo -f compose.mellumo.yaml ps
docker compose --env-file .env.mellumo -f compose.mellumo.yaml logs --tail=100 postiz
docker compose --env-file .env.mellumo -f compose.mellumo.yaml down
```

Do not share raw logs containing tokens or personal content. `down` preserves named volumes. Back up application DB, Temporal DB/history, media/config volumes and necessary secrets before upgrades. Do not use `down -v` unless deliberately erasing the local installation.

## Source edits

Application source is at root `apps/` and `libraries/`. Rebuild the image to run source changes through this setup. The old `evaluation/` directory is a separate ignored local experiment, not the public application tree. Its running image does not automatically reflect root source edits.

Use focused checks for changes. Do not rerun all upstream behavior for metadata/documentation work. The initial foundation publication does not claim a fresh application build or a production-ready deployment.

## Hosting later

This configuration is for local development. Before hosting a modified version, configure public HTTPS URLs/provider callbacks, secrets and backups, registration/access policy, and a prominent source/license link tied to the deployed revision. Publish that revision's corresponding source and build instructions. No production service or container release is created by the foundation setup.
