# Patch Notes — Huly Coolify SMEnode Named Volumes

Domain: `huly.smenode.com`

This package is based on the official Huly self-host files uploaded by the user. The original files are preserved under `ORIGINAL-FILES/`.

## Version

```env
HULY_VERSION=v0.7.423
DESKTOP_CHANNEL=0.7.423
```

## Operational choice in this v3 package

This version intentionally uses Docker named volumes instead of host bind mounts.

Result:

- No SSH `mkdir` step is required.
- No SSH `chown` step is required.
- Docker/Coolify creates and manages volumes automatically.

## Changes from the uploaded upstream compose

### 1. nginx host port removed

Original:

```yaml
ports:
  - "${HTTP_BIND}:${HTTP_PORT}:80"
```

Patched:

```yaml
expose:
  - "80"
```

Reason: Coolify should publish only the `nginx` service through its reverse proxy.

### 2. kvs host port removed

Original:

```yaml
ports:
  - 8094:8094
```

Patched:

```yaml
expose:
  - "8094"
```

Reason: `kvs` should remain private inside the Compose stack.

### 3. active `huly_net` network references removed

The active service-level `networks: - huly_net` entries and the active top-level `networks:` block were removed.

Reason: Coolify creates its own Compose network and service discovery still works by service name.

Commented optional examples in the original compose may still contain commented `networks:` lines. They remain comments and are not active.

### 4. transactor FRONT_URL corrected for domain

Original:

```yaml
- FRONT_URL=http://localhost:8087
```

Patched:

```yaml
- FRONT_URL=http${SECURE:+s}://${HOST_ADDRESS}
```

Reason: public links/callbacks should use `huly.smenode.com`, not localhost.

### 5. huly.nginx renamed to .huly.nginx

The uploaded file was named `huly.nginx`, while the compose file mounts:

```yaml
./.huly.nginx:/etc/nginx/conf.d/default.conf
```

So the deployment file is named `.huly.nginx`.

### 6. .env.example changed to named volumes

The previous package included:

```env
VOLUME_ELASTIC_PATH=/data/huly/elastic
VOLUME_FILES_PATH=/data/huly/files
VOLUME_CR_DATA_PATH=/data/huly/cockroach
VOLUME_CR_CERTS_PATH=/data/huly/cockroach-certs
VOLUME_REDPANDA_PATH=/data/huly/redpanda
```

This v3 package removes those active variables.

Reason: leaving `VOLUME_*_PATH` unset activates the defaults in compose:

```yaml
${VOLUME_ELASTIC_PATH:-elastic}
${VOLUME_FILES_PATH:-files}
${VOLUME_CR_DATA_PATH:-cr_data}
${VOLUME_CR_CERTS_PATH:-cr_certs}
${VOLUME_REDPANDA_PATH:-redpanda}
```

So Docker/Coolify uses named volumes automatically.

## What was not changed

- No Huly image names were changed.
- No optional services were enabled.
- No mail service was added.
- No AI service was added.
- No print/export/GitHub/calendar/Telegram/Love/HulyPulse service was enabled.
- No database type was changed.
- No nginx routes were intentionally altered.

## Files not needed for Coolify

`setup.sh` and `nginx.sh` are kept in `ORIGINAL-FILES/` only. Do not run them in Coolify.
