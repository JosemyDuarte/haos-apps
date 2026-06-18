ARG BUILD_FROM=ghcr.io/josemyduarte/inker:0.4.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}
# NOTE: the fork image removes the apt lists (`rm -rf /var/lib/apt/lists/*`), so
# `apt-get install` cannot resolve deps here. We don't need it: the `node` binary
# is present in the image (copied from node:22-slim), so 01-ha-options parses
# /data/options.json with `node` instead of jq. Keep this a pure COPY overlay.
COPY rootfs /
RUN chmod +x /etc/cont-init.d/00-ha-relocate /etc/cont-init.d/01-ha-options
