# Contributing

Guidelines for contributing to this Docker image repository.

## Repository structure

```
<image-name>/
  Dockerfile            # Single Dockerfile, parameterized with ARG
  .dockerignore         # Build context exclusions
.github/workflows/
  <image-name>.yaml     # CI workflow per image
  cleanup.yaml          # Prune untagged manifests
```

All images are published to `ghcr.io/sergio-bershadsky/`.

## Dockerfile rules

### Required

- **`ARG` before `FROM`** for the base image version. Re-declare `ARG` after `FROM`
  if the value is needed in labels or scripts.
  ```dockerfile
  ARG ODOO_VERSION=18.0
  FROM odoo:${ODOO_VERSION}
  ARG ODOO_VERSION
  ```

- **OCI labels** linking the image to this repo. At minimum:
  ```dockerfile
  LABEL org.opencontainers.image.source="https://github.com/sergio-bershadsky/docker"
  ```

- **Run as non-root.** End the Dockerfile with `USER <non-root-user>`. Only switch to
  `root` for package installation, then switch back.

- **Single `RUN` per concern.** Combine related commands with `&&` to minimize layers.
  Always clean up caches in the same layer:
  ```dockerfile
  RUN apt-get update && apt-get install -y --no-install-recommends \
      git \
      && rm -rf /var/lib/apt/lists/*
  ```

- **`--no-install-recommends`** for apt-get. Keeps images small.

- **`--no-cache-dir`** for pip install. Prevents cache bloat.

- **`.dockerignore`** in each image directory. At minimum exclude `*.md`, `.env`,
  `__pycache__/`.

### Prohibited

- **No secrets in `ARG` or `ENV`.** Use `--mount=type=secret` for build-time secrets.
  Runtime secrets must come from environment variables or mounted files.

- **No `latest` tag as base.** Always pin to a specific version branch
  (e.g., `odoo:18.0`, not `odoo:latest`).

- **No `COPY . .`** without a `.dockerignore`. Explicitly list what gets copied.

## Workflow rules

### Build workflow (`<image-name>.yaml`)

Each image has its own workflow file. Required features:

1. **Scheduled trigger** (`cron`) to detect upstream changes automatically.
2. **`workflow_dispatch`** for manual re-builds.
3. **Multi-arch builds** (`linux/amd64,linux/arm64`) via QEMU + Buildx.
4. **`docker/metadata-action`** for OCI labels. Include at minimum:
   - `org.opencontainers.image.title`
   - `org.opencontainers.image.description`
   - `org.opencontainers.image.version`
   - `org.opencontainers.image.vendor`
   - `org.opencontainers.image.licenses`
   - `org.opencontainers.image.base.name`
5. **GHA build cache** with per-branch scoping:
   ```yaml
   cache-from: type=gha,scope=<image>-${{ matrix.branch }}
   cache-to: type=gha,scope=<image>-${{ matrix.branch }},mode=max
   ```
6. **SBOM generation** (`sbom: true` in `docker/build-push-action`).
7. **Provenance attestation** (`provenance: mode=max`).
8. **Cosign keyless signing** after push. Requires `permissions: id-token: write`.
9. **Vulnerability scanning** with Trivy. Set `exit-code: 0` to report without
   failing the build. Use `severity: CRITICAL,HIGH` to reduce noise.

### Security requirements for workflows

- **Never use untrusted input in `run:` commands.** Always pass through `env:` blocks:
  ```yaml
  # WRONG
  run: echo "${{ github.event.issue.title }}"

  # RIGHT
  env:
    TITLE: ${{ github.event.issue.title }}
  run: echo "$TITLE"
  ```
- **Use `GITHUB_TOKEN`**, not PATs, for authentication within workflows.
- **Pin actions to major versions** (e.g., `@v4`, not `@main`).

### Retention policy

**Tagged images are never deleted.** Storage on public ghcr.io is free and unlimited.
Deleting a tag that someone pins to in production would break their deployments.

The `cleanup.yaml` workflow runs weekly and only removes **untagged manifests**
(orphaned layers left over from multi-arch builds). It never touches any tagged image.

Both dated tags (`18.0-20250220`) and rolling tags (`18.0`) are safe to pin to
in production deployments.

## Tagging convention

### Image tags

| Tag | Example | Description |
|-----|---------|-------------|
| `<branch>-<YYYYMMDD>` | `18.0-20250220` | Pinned to upstream commit date |
| `<branch>` | `18.0` | Rolling, always the latest build |

### Adding a new tracked branch

Add the branch name to the `BRANCHES` env var in the image's workflow file.
The single Dockerfile handles all branches via the version `ARG`.

## Documentation rules

ghcr.io package pages display the **root `README.md`** from the linked repository --
there is no way to show a per-image README. Because of this:

- The **root `README.md`** must contain a description and registry/package links for
  every image. This is what users see on the ghcr.io package page.
- Each **`<image-name>/README.md`** contains full documentation for that image
  (usage, addons, build instructions, etc.).
- Both READMEs must include direct links to the ghcr.io package page:
  `https://github.com/sergio-bershadsky/docker/pkgs/container/<image-name>`
- When adding or updating an image, keep both READMEs in sync.
- Each **`<image-name>/TAGS.md`** is auto-generated by CI after each build. **Do not
  edit manually** — it will be overwritten on the next build.

## Adding a new image

1. Create `<image-name>/Dockerfile` with `ARG`-based version parameterization.
2. Create `<image-name>/.dockerignore`.
3. Create `<image-name>/README.md` with full image documentation and package links.
4. Create `.github/workflows/<image-name>.yaml` following the workflow rules above.
5. Update root `README.md` with the new image entry, description, and package link.
6. Add the package name to `cleanup.yaml` for untagged manifest pruning.

## Local development

### Build locally

```bash
docker build --build-arg ODOO_VERSION=18.0 -t odoo-custom:18.0 ./odoo/
```

### Test locally

```bash
docker run --rm odoo-custom:18.0 odoo --version
```

### Lint Dockerfiles

Use [hadolint](https://github.com/hadolint/hadolint):

```bash
hadolint odoo/Dockerfile
```

## Code review checklist

Before merging any PR, verify:

- [ ] Dockerfile follows the rules above (non-root, no secrets, pinned base, clean layers)
- [ ] `.dockerignore` exists and is adequate
- [ ] OCI labels are set (at minimum `image.source`)
- [ ] Workflow includes all required steps (cache, SBOM, signing, scanning)
- [ ] No untrusted input in workflow `run:` commands
- [ ] Root README and `<image>/README.md` both updated with package links
- [ ] Local build succeeds for all target versions
