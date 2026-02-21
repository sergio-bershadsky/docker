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
# Note: adjust packages based on PyPI availability;
#       fall back to pip+git if not yet published.
RUN pip3 install --no-cache-dir \
    odoo-addon-session_db \
    odoo-addon-fs_storage \
    odoo-addon-fs_attachment \
    odoo-addon-fs_attachment_s3 \
    "fsspec[s3]"

USER odoo
