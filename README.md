# Rocicorp Dev Container Templates

Shared [dev container templates](https://containers.dev/implementors/templates/)
for Rocicorp repos. Companion to
[rocicorp/devcontainer-features](https://github.com/rocicorp/devcontainer-features).

## The rule

The dev container is the sandbox for development and AI-agent work, so it
must not be able to create or control containers:

- **No Docker-in-Docker** (requires privileged mode and weakens the
  container boundary).
- **No host Docker socket** mounted into the workspace.
- Local services (Postgres etc.) run as **sibling Compose containers**,
  orchestrated by host-side Dev Containers tooling. The workspace reaches
  them over the Compose network by service name; it cannot manage them.

## Templates

### `node` — workspace only

For repos with no local services. Node + pnpm + agent tooling.

```bash
npx @devcontainers/cli templates apply --workspace-folder . \
  --template-id ghcr.io/rocicorp/devcontainer-templates/node:1
```

### `sibling-services` — workspace + the repo's Compose services

For repos that already have a `docker-compose.yml` for local services. The
template adds a `dev` workspace service beside them; the repo's Compose file
is used unmodified.

```bash
npx @devcontainers/cli templates apply --workspace-folder . \
  --template-id ghcr.io/rocicorp/devcontainer-templates/sibling-services:1 \
  --template-args '{
    "appName": "hello-zero",
    "servicesComposeFile": "docker/docker-compose.yml",
    "dbService": "zstart_postgres"
  }'
```

Then edit the `environment:` block in the generated
`.devcontainer/docker-compose.yml` to the env vars your app reads (host =
Compose service name, port = container port), and open the repo in the
container. `db-up`-style scripts are not needed (and don't work) inside the
workspace — the services are already running.

Both templates can also be applied from the VS Code / Dev Containers UI:
"Add Dev Container Configuration Files…" and search for the template id.

## Conventions for service Compose files

- Give the database service a `healthcheck` (`pg_isready …`) so the
  workspace can `depends_on: condition: service_healthy`.
- Published `ports:` are for the host; in-workspace connections use
  `service_name:container_port`.
- Persistent data goes in named volumes; reset by removing the volume from
  the host, never by giving the workspace Docker access.

## Reference implementations

- [rocicorp/mono `.devcontainer/`](https://github.com/rocicorp/mono/tree/main/.devcontainer)
  — multi-profile setup (default + zbugs) with static Postgres siblings for
  the zero-cache pg test matrix (`TEST_PG_<major>` env vars).

## Releasing

Pushing to `main` publishes the templates to
`ghcr.io/rocicorp/devcontainer-templates/<id>` via
`.github/workflows/release.yml`. Bump `version` in the template's
`devcontainer-template.json` when changing it.
