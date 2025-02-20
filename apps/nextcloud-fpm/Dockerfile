FROM docker.io/nextcloud:26.0.1-fpm@sha256:91106db7332cd098da191c4831b29b1a2da0a5d2ab4b34e8cd281de4c1f6d433

# hadolint ignore=DL3008,DL3015,SC2086,SC2155,DL4006
RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        nano \
        jq \
        imagemagick-common \
        ffmpeg \
        libmagickcore-6.q16-6-extra \
        libde265-0 \
        heif-gdk-pixbuf \
        libheif1 \
        procps \
        smbclient \
        libfcgi-bin \
    ; \
    savedAptMark="$(apt-mark showmanual)"; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libbz2-dev \
        libc-client-dev \
        libkrb5-dev \
        libsmbclient-dev \
        libmagickcore-dev \
    ; \
    \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl; \
    docker-php-ext-install \
        bz2 \
        imap \
    ; \
    pecl install smbclient; \
    docker-php-ext-enable smbclient; \
    \
    # Reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*

# Copy occ script to /usr/bin/occ
COPY --chmod=755 ./apps/nextcloud-fpm/scripts/occ /usr/bin/occ

# Copy post-install script to a temp location so we can append it to the entrypoint.sh
COPY --chmod=755 ./apps/nextcloud-fpm/scripts/post-install.sh /tmp/post-install.sh

# Copy the healthcheck
COPY --chmod=755 ./apps/nextcloud-fpm/scripts/healthcheck.sh /healthcheck.sh

# Copy the configure-scripts that will be sourced by the post-install
COPY --chmod=755 ./apps/nextcloud-fpm/configure-scripts /configure-scripts

RUN set -ex; \
    sed -i 's/exec "$@"//g' /entrypoint.sh; \
    cat /tmp/post-install.sh >> /entrypoint.sh

LABEL "org.opencontainers.image.source"="https://github.com/truecharts/containers"

ARG CONTAINER_NAME
ARG CONTAINER_VER
LABEL org.opencontainers.image.licenses="BSD-3-Clause"
LABEL org.opencontainers.image.title="${CONTAINER_NAME}"
LABEL org.opencontainers.image.url="https://truecharts.org/docs/charts/${CONTAINER_NAME}"
LABEL org.opencontainers.image.version="${CONTAINER_VER}"
LABEL org.opencontainers.image.description="Container for ${CONTAINER_NAME} by TrueCharts"
LABEL org.opencontainers.image.authors="TrueCharts"
LABEL org.opencontainers.image.documentation="https://truecharts.org/docs/charts/${CONTAINER_NAME}"
