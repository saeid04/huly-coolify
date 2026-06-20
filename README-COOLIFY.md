# Huly on Coolify — huly.smenode.com

This package is prepared for deploying Huly on Coolify using Docker Compose and Docker named volumes. It is based on the upstream files provided by Saeid and patched only where needed for Coolify.

## What this version does

- Uses `huly.smenode.com` as the public host.
- Uses Docker named volumes, so no manual SSH `mkdir` or `chown` step is required.
- Exposes only the internal `nginx` service on port `80` to Coolify.
- Keeps Huly backend services private inside the Compose stack.
- Does not enable optional services such as mail, AI, print, exports, GitHub, calendar, Love, Redis, MongoDB, or HulyPulse.

## Files to upload to your GitHub repo

Upload these files/folders to a new GitHub repo:

- `compose.yml`
- `.huly.nginx`
- `.env.example`
- `README-COOLIFY.md`
- `PATCH-NOTES.md`
- `SHA256SUMS.txt`
- `ORIGINAL-FILES/` optional but recommended for audit

## DNS

Create this DNS record:

```text
huly.smenode.com A <your Coolify server IP>
```

If you use Cloudflare, start with **DNS only** until deployment is confirmed working.

## Coolify application settings

Create a new application in Coolify from your GitHub repo.

Use:

```text
Build Pack: Docker Compose
Branch: main
Base Directory: /
Docker Compose Location: /compose.yml
```

## Environment variables

In Coolify, open:

```text
Application > Environment Variables > Developer View
```

Copy the contents of `.env.example` and paste them there.

Change these values before deploying:

```env
CR_USER_PASSWORD=CHANGE_ME_STRONG_COCKROACH_PASSWORD
CR_DB_URL=postgresql://selfhost:CHANGE_ME_STRONG_COCKROACH_PASSWORD@cockroach:26257/defaultdb?sslmode=disable
REDPANDA_ADMIN_PWD=CHANGE_ME_STRONG_REDPANDA_PASSWORD
SECRET=CHANGE_ME_LONG_RANDOM_SECRET
```

Generate secrets with:

```bash
openssl rand -hex 32
```

Important: the password inside `CR_DB_URL` must exactly match `CR_USER_PASSWORD`.

Do not add these variables in this named-volume version:

```env
VOLUME_ELASTIC_PATH=
VOLUME_FILES_PATH=
VOLUME_CR_DATA_PATH=
VOLUME_CR_CERTS_PATH=
VOLUME_REDPANDA_PATH=
```

Leaving them unset lets Docker/Coolify create named volumes automatically.

## Public service

In Coolify, publish only this service:

```text
Service: nginx
Domain: https://huly.smenode.com
Target port: 80
```

Do not publish these services:

```text
cockroach
redpanda
minio
elastic
account
workspace
front
transactor
collaborator
fulltext
stats
rekoni
kvs
```

## Deploy

Click **Deploy** in Coolify.

After containers start, open:

```text
https://huly.smenode.com
```

Create the first account before disabling sign-up.

## Disable sign-up after first account

After your first account is created, add this environment variable to both `account` and `front` service environments in `compose.yml`:

```yaml
- DISABLE_SIGNUP=true
```

Commit the change and redeploy.

## Backups

This version uses Docker named volumes. Make sure your Coolify/server backup strategy includes Docker volumes for this app.

Named volumes used by this stack:

```text
elastic
files
cr_data
cr_certs
redpanda
mongodb
```

The most important volumes are:

```text
cr_data
files
redpanda
elastic
```

## Do not run setup.sh or nginx.sh in Coolify

The upstream `setup.sh` and `nginx.sh` are preserved in `ORIGINAL-FILES/` for audit only. They are for classic server installation with system nginx, not for Coolify.
