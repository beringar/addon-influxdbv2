ARG BUILD_FROM=ghcr.io/hassio-addons/debian-base/amd64:9.3.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base system
ARG BUILD_ARCH=amd64
ARG INFLUXDB_VERSION="2.9.1"
ARG INFLUXCLI_VERSION="2.8.0"
ARG TRAEFIK_VERSION="2.11.57"

RUN \
    apt-get -o Acquire::Check-Valid-Until=false update \
    && apt-get install -y --no-install-recommends \
        luarocks \
        procps \
    \
    && luarocks install lua-resty-http 0.15-0 \
    \
    && ARCH="${BUILD_ARCH}" \
    && if [ "${BUILD_ARCH}" = "aarch64" ]; then ARCH="arm64"; fi \
    \
    && curl -J -L -o /tmp/influxdb.deb \
        "https://dl.influxdata.com/influxdb/releases/influxdb2_${INFLUXDB_VERSION}-1_${ARCH}.deb" \
    \
    && curl -J -L -o /tmp/influxcli.tar.gz \
        "https://dl.influxdata.com/influxdb/releases/influxdb2-client-${INFLUXCLI_VERSION}-linux-${ARCH}.tar.gz" \
    \
    && curl -J -L -o /tmp/traefik.tar.gz \
        "https://github.com/traefik/traefik/releases/download/v${TRAEFIK_VERSION}/traefik_v${TRAEFIK_VERSION}_linux_${ARCH}.tar.gz" \
    \
    && dpkg --install /tmp/influxdb.deb \
    && tar xvzf /tmp/influxcli.tar.gz -C /tmp \
    && cp /tmp/influx /usr/local/bin/ \
    && tar xvzf /tmp/traefik.tar.gz -C /tmp \
    && cp /tmp/traefik /usr/local/bin/ \
    \
    && rm -fr \
        /tmp/* \
        /etc/nginx \
        /var/{cache,log}/* \
        /var/lib/apt/lists/* \
    \
    && mkdir -p /var/log/nginx \
    && touch /var/log/nginx/error.log

RUN mkdir -p /data/influxdb
RUN chmod 0750 /etc/influxdb \
    && chmod 0750 /data/influxdb

# Copy root filesystem
COPY rootfs /

# Set WORKDIR to automatically detect config.
WORKDIR /etc/influxdb

# Environment variables
ENV INFLUXD_CONFIG_PATH=/etc/influxdb
ENV INFLUXD_ENGINE_PATH=/data/influxdb
ENV INFLUXD_BOLT_PATH=/data/influxdb/influxd.bolt
ENV INFLUXD_HTTP_BIND_ADDRESS=0.0.0.0:8086
ENV INFLUXD_LOG_LEVEL=info

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon"
