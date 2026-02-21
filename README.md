# docker

Custom Docker images published to `ghcr.io/sergio-bershadsky/`.

## Images

### Odoo

| Source | Package | Full docs | All tags |
|--------|---------|-----------|----------|
| [odoo/](odoo/) | [ghcr.io/…/odoo](https://github.com/sergio-bershadsky/docker/pkgs/container/odoo) | [README](odoo/README.md) | [TAGS.md](odoo/TAGS.md) |

Stateless Odoo images for cloud-native deployments. Built on top of the official
`odoo` image with OCA addons that move all state out of the container:

- **Sessions** stored in PostgreSQL (not local files) via `session_db`
- **Attachments** stored in S3-compatible object storage (not local filestore) via `fs_attachment`

This makes containers fully disposable and horizontally scalable -- no persistent
volumes needed on the application tier. Works with Kubernetes, AWS ECS/Fargate,
Google Cloud Run, Azure Container Apps, Heroku, Fly.io, or plain Docker Compose.

Tracked branches: **17.0**, **18.0**, **19.0** — see [all available tags](odoo/TAGS.md).

```bash
docker pull ghcr.io/sergio-bershadsky/odoo:18.0
```

## How it works

A scheduled GitHub Actions workflow runs daily and:

1. Checks [`odoo/docker`](https://github.com/odoo/docker) Dockerfiles for new `ODOO_RELEASE` nightly dates
2. Checks if the corresponding image tag already exists in ghcr.io
3. Builds and pushes only when a new official nightly is published
4. Signs images with cosign (keyless, Sigstore)
5. Generates SBOM and provenance attestations
6. Scans for vulnerabilities with Trivy

Can also be triggered manually via workflow_dispatch.

**Tagged images are never deleted** -- both dated and rolling tags are safe to pin in
production. A weekly cleanup workflow only removes untagged manifests (orphaned
multi-arch layers).

All images are multi-arch (amd64 + arm64).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for Dockerfile rules, workflow requirements,
and the code review checklist.
