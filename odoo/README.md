# Odoo

Custom Odoo Docker images with pre-installed OCA addons, published to
`ghcr.io/sergio-bershadsky/odoo`.

## Tracked branches

| Branch | Base image | Registry tag |
|--------|-----------|--------------|
| 17.0 | `odoo:17.0` | `ghcr.io/sergio-bershadsky/odoo:17.0` |
| 18.0 | `odoo:18.0` | `ghcr.io/sergio-bershadsky/odoo:18.0` |
| 19.0 | `odoo:19.0` | `ghcr.io/sergio-bershadsky/odoo:19.0` |

## Pre-installed addons

| Addon | Purpose |
|-------|---------|
| `session_db` | HTTP sessions stored in the database |
| `fs_storage` | fsspec-based storage backend framework |
| `fs_attachment` | Route `ir.attachment` to external storage |
| `fs_attachment_s3` | S3-specific features (signed URLs, x-sendfile) |
| `fsspec[s3]` | S3 filesystem driver for fsspec |

## Image tags

| Tag | Example | Description |
|-----|---------|-------------|
| `<branch>-<YYYYMMDD>` | `18.0-20260217` | Pinned to official Odoo nightly release date |
| `<branch>` | `18.0` | Rolling, always the latest build |

All images are multi-arch (amd64 + arm64).
Tagged images are never deleted.

## How builds are triggered

The CI monitors the official [`odoo/docker`](https://github.com/odoo/docker) repo
daily. Each branch Dockerfile contains an `ARG ODOO_RELEASE=YYYYMMDD` that pins the
exact nightly build. When Odoo publishes a new nightly (updating `ODOO_RELEASE`), the
CI detects it and rebuilds our image. This ensures `FROM odoo:18.0` always resolves
to a Docker Hub image that actually exists.

## Usage

```bash
docker pull ghcr.io/sergio-bershadsky/odoo:18.0
```

Or in a Helm chart:

```yaml
backend:
  image:
    repository: ghcr.io/sergio-bershadsky/odoo
    tag: "18.0-20250220"
```

## Build locally

```bash
docker build --build-arg ODOO_VERSION=18.0 -t odoo-custom:18.0 .
```

## Verify image signature

```bash
cosign verify ghcr.io/sergio-bershadsky/odoo:18.0 \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  --certificate-identity-regexp='github\.com/sergio-bershadsky/docker'
```

## How the Dockerfile works

A single `Dockerfile` builds all versions via the `ODOO_VERSION` build arg:

```dockerfile
ARG ODOO_VERSION=18.0
FROM odoo:${ODOO_VERSION}
```

The CI passes `--build-arg ODOO_VERSION=<branch>` for each matrix entry.
