# Coolify Deployment

Use the dedicated Coolify Compose file instead of the default `docker-compose.yml`.
The default file is meant for local/self-managed deployments with optional profiles,
host-bound ports, and an optional bundled Traefik proxy. Coolify already provides
the proxy and expects the Compose file to be the deployment source of truth.

## Coolify Resource Settings

1. Create a new resource from the Git repository.
2. Choose the Docker Compose build pack.
3. Set Base Directory to `/`.
4. Set Docker Compose Location to `/docker-compose.coolify.yml`.
5. Assign your public domain to the `dashboard` service on port `80`.

The dashboard serves the React app and proxies:

- `/api/*` to `openwa-api:2785`
- `/socket.io/*` to `openwa-api:2785`

That lets the dashboard and API share one public origin, for example:

- `https://openwa.example.com`
- `https://openwa.example.com/api`
- `https://openwa.example.com/api/docs`

## Required Environment Variables

Set these in Coolify before the first deploy:

```env
API_MASTER_KEY=replace-with-a-long-random-secret
DATABASE_PASSWORD=replace-with-a-long-random-secret
```

These are runtime secrets. They do not need to be available during the Docker
image build.

Recommended production value:

```env
CORS_ORIGINS=https://openwa.example.com
```

Optional values you may override:

```env
DATABASE_NAME=openwa
DATABASE_USERNAME=openwa
LOG_LEVEL=info
ENABLE_SWAGGER=true
PUPPETEER_HEADLESS=true
```

## Persistence

The Coolify Compose file defines two named volumes:

- `openwa-data`: WhatsApp sessions, browser data, local media, plugins, and the
  app's small internal SQLite config database.
- `postgres-data`: PostgreSQL data for sessions, webhooks, and messages.

Do not remove these volumes during redeploys unless you intentionally want to
reset the instance.

## Health Checks

The API health check is:

```text
/api/health
```

The dashboard health check is:

```text
/
```

Coolify should route traffic only after both the API and dashboard are healthy.

## Notes

- Do not assign a public domain to `postgres`.
- You usually do not need to assign a public domain to `openwa-api`; the
  dashboard service exposes the API at the same domain under `/api`.
- The Coolify stack does not mount `/var/run/docker.sock`. The dashboard's
  optional built-in service orchestration controls are intentionally disabled in
  this deployment shape; Postgres is already defined in Compose.
