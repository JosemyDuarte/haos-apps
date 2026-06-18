ARG BUILD_FROM=docker.io/wojooo/inker:0.4.0
FROM ${BUILD_FROM}
# NOTE: upstream slimmed this image with `dpkg --purge --force-depends` and
# removed apt lists, so `apt-get install` cannot resolve deps. We parse
# /data/options.json with `node` (already in the image) instead of jq.
COPY rootfs /
RUN chmod +x /etc/cont-init.d/00-ha-relocate /etc/cont-init.d/01-ha-options
