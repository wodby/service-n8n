# n8n on Wodby

What Wodby sets up for n8n on this service. n8n runs from the official `n8nio/n8n` image (Community Edition) as user 1000. Configuration is passed as environment variables.

## Linked services

| Link | Variables |
| --- | --- |
| PostgreSQL (required) | `DB_POSTGRESDB_HOST`, `DB_POSTGRESDB_PORT`, `DB_POSTGRESDB_DATABASE`, `DB_POSTGRESDB_USER`, `DB_POSTGRESDB_PASSWORD` |
| SMTP server (optional) | `N8N_EMAIL_MODE` (`smtp`), `N8N_SMTP_HOST`, `N8N_SMTP_PORT`, `N8N_SMTP_SSL` and `N8N_SMTP_STARTTLS` (`false`), `N8N_SMTP_SENDER` (`noreply@` the service's host) |

`DB_TYPE` is `postgresdb`. The linked database holds workflows, credentials, executions, users and settings.

## What the manifest sets

- `N8N_ENCRYPTION_KEY` comes from the generated token `encryption_key`. n8n encrypts stored credentials with it. Changing it makes them unreadable, so keep it. A copy of the database restored into another environment needs the same key.
- `WEBHOOK_URL` and `N8N_EDITOR_BASE_URL` are the service's URL. The start script sets them from `N8N_PUBLIC_URL`. Without a URL, n8n builds links from the request.
- `N8N_PROXY_HOPS` is `1`: TLS ends at Wodby's router, and n8n reads the client address from its headers. HTTP requests are redirected to HTTPS by default.
- `GENERIC_TIMEZONE` and `TZ` come from the Timezone setting, `UTC` by default.
- `N8N_DIAGNOSTICS_ENABLED` and `N8N_VERSION_NOTIFICATIONS_ENABLED` are `false`. Wodby delivers new versions.
- The volume `data` is mounted at `/home/node/.n8n`. It holds community nodes, binary data and n8n's own files.

## Owner account

The Owner email and Owner password settings are required. `N8N_INSTANCE_OWNER_MANAGED_BY_ENV` is `true`, so n8n applies the owner's email and password on every start, and its setup page is never shown. n8n takes the password only as a bcrypt hash, so the container's start script hashes the setting into `N8N_INSTANCE_OWNER_PASSWORD_HASH` and removes the plain password from n8n's environment. The owner's email and password can't be changed in n8n. Change the settings and redeploy, which signs the owner out. A password shorter than 8 characters stops the container with a message in its log.

## Code node

JavaScript and Python Code nodes run in the `runner` container, n8n's `n8nio/runners` image at the same version as n8n. It connects to n8n's task broker on `127.0.0.1:5679` with `N8N_RUNNERS_AUTH_TOKEN`, from the generated token `runners_token`. `N8N_RUNNERS_MODE` is `external`, and `N8N_RUNNERS_TASK_TIMEOUT` is `300` seconds in both containers. The runner gets only these variables and `GENERIC_TIMEZONE`, not n8n's database or encryption settings. Which modules Code nodes may import is set in the runner image's `/etc/n8n-task-runners.json`: by default Python can't import standard-library modules, and JavaScript can use `crypto` and `moment`.

## Scaling

The service runs one n8n process. More replicas need n8n's queue mode with Redis and worker processes, or an Enterprise license for several main processes.

## Versions and upgrades

n8n migrates its database when a newer version starts. Back up the PostgreSQL database before an upgrade; an older version can't be started on a migrated database.

## Check the result

From inside the environment, `/healthz` on port 5678 answers `{"status":"ok"}`. `/healthz/readiness` also checks the database.
