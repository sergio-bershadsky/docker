ARG ODOO_VERSION=18.0
FROM odoo:${ODOO_VERSION}

ARG ODOO_VERSION
LABEL org.opencontainers.image.source="https://github.com/sergio-bershadsky/docker"
LABEL org.opencontainers.image.base.name="docker.io/library/odoo:${ODOO_VERSION}"

USER root

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    && rm -rf /var/lib/apt/lists/*

# OCA storage stack (fsspec-based) + session management
# --break-system-packages: required for PEP 668 (Debian Bookworm+, Odoo 18+)
# Older pip (Odoo 17/Bullseye) does not recognize the flag, so detect support.
RUN PIP_FLAGS="--no-cache-dir"; \
    pip3 install --break-system-packages --help >/dev/null 2>&1 \
      && PIP_FLAGS="$PIP_FLAGS --break-system-packages"; \
    pip3 install $PIP_FLAGS \
      odoo-addon-session_db \
      odoo-addon-fs_storage \
      odoo-addon-fs_attachment \
      odoo-addon-fs_attachment_s3 \
      "fsspec[s3]"

USER odoo
