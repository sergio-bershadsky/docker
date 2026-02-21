# docker

Custom Docker images published to `ghcr.io/sergio-bershadsky/`.

## Images

| Image | Tracked branches | Registry |
|-------|-----------------|----------|
| Odoo  | 17.0, 18.0, 19.0 | `ghcr.io/sergio-bershadsky/odoo` |

## How it works

A scheduled GitHub Actions workflow runs daily and:

1. Queries the latest commit date on each `odoo/odoo` branch
2. Checks if `ghcr.io/sergio-bershadsky/odoo:<branch>-<YYYYMMDD>` already exists
3. Builds and pushes only for branches with new commits
4. Signs images with cosign (keyless, Sigstore)
5. Generates SBOM and provenance attestations
6. Scans for vulnerabilities with Trivy

Can also be triggered manually via workflow_dispatch.

**Tagged images are never deleted** — both dated and rolling tags are safe to pin in
production. A weekly cleanup workflow only removes untagged manifests (orphaned
multi-arch layers).

### Image tags

| Tag | Example | Description |
|-----|---------|-------------|
| `<branch>-<YYYYMMDD>` | `18.0-20250220` | Pinned to upstream commit date |
| `<branch>` | `18.0` | Rolling, always the latest build |

All images are multi-arch (amd64 + arm64).

## Odoo

Single `odoo/Dockerfile` parameterized with `ODOO_VERSION` build arg.

Pre-installed OCA addons:

- **session_db** -- HTTP sessions stored in the database
- **fs_storage** -- fsspec-based storage backend framework
- **fs_attachment** -- route `ir.attachment` to external storage
- **fs_attachment_s3** -- S3-specific features (signed URLs, x-sendfile)

### Build locally

```bash
docker build --build-arg ODOO_VERSION=18.0 -t odoo-custom:18.0 ./odoo/
```

### Verify image signature

```bash
cosign verify ghcr.io/sergio-bershadsky/odoo:18.0 \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  --certificate-identity-regexp='github\.com/sergio-bershadsky/docker'
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for Dockerfile rules, workflow requirements,
and the code review checklist.
