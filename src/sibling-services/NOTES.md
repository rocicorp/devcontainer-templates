# Next steps

Open the repo with **"Dev Containers: Clone Repository in Container
Volume…"** — the workspace deliberately has no host bind mounts, so the
host needs only Docker + VS Code (no git checkout, no Node). The source
lives in the repo volume at `/workspaces` and is edited through VS Code.

1. In `.devcontainer/docker-compose.yml`, set the `environment:` entries to
   the connection strings your app actually reads (host = the Compose
   service name, port = the *container* port, not the host-published one).
2. If the database service has no `healthcheck`, change `condition:
   service_healthy` to `service_started`.
3. Reopen in container. Do not run the repo's `db-up`-style scripts inside
   the workspace — the services are already running as siblings. Resets and
   rebuilds happen from the host (Dev Containers: "Rebuild Container", or
   `docker compose` / `docker volume` on the host).
