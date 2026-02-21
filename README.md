# docker

Custom Docker images published to `ghcr.io/sergio-bershadsky/`.

## Images

| Image | Tracked branches | Registry | Docs |
|-------|-----------------|----------|------|
| [Odoo](odoo/) | 17.0, 18.0, 19.0 | `ghcr.io/sergio-bershadsky/odoo` | [odoo/README.md](odoo/README.md) |

## How it works

A scheduled GitHub Actions workflow runs daily and:

1. Queries the latest commit date on each upstream branch
2. Checks if the corresponding image tag already exists in ghcr.io
3. Builds and pushes only when new upstream commits are detected
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
