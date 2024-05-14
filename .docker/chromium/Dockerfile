# Stage 1: Setup our base
ARG BASE_IMAGE=newsoftwarehq/neko:base
FROM --platform=linux/amd64 $BASE_IMAGE as base

#
# install neko chromium
RUN set -eux; \
    echo "deb http://ftp.de.debian.org/debian bookworm main" >> /etc/apt/sources.list; \
    apt-get update; \
    apt-get install -y --no-install-recommends unzip chromium chromium-common chromium-sandbox openbox; \
    #
    # install widevine module
    CHROMIUM_DIR="/usr/lib/chromium"; \
    WIDEVINE_VERSION=$(wget --quiet -O - https://dl.google.com/widevine-cdm/versions.txt | sort --version-sort | tail -n 1); \
    wget -O /tmp/widevine.zip "https://dl.google.com/widevine-cdm/${WIDEVINE_VERSION}-linux-x64.zip"; \
    mkdir -p "${CHROMIUM_DIR}/WidevineCdm/_platform_specific/linux_x64"; \
    unzip -p /tmp/widevine.zip LICENSE.txt > "${CHROMIUM_DIR}/WidevineCdm/LICENSE"; \
    unzip -p /tmp/widevine.zip manifest.json > "${CHROMIUM_DIR}/WidevineCdm/manifest.json"; \
    unzip -p /tmp/widevine.zip libwidevinecdm.so > "${CHROMIUM_DIR}/WidevineCdm/_platform_specific/linux_x64/libwidevinecdm.so"; \
    find "${CHROMIUM_DIR}/WidevineCdm" -type d -exec chmod 0755 '{}' \;; \
    find "${CHROMIUM_DIR}/WidevineCdm" -type f -exec chmod 0644 '{}' \;; \
    rm /tmp/widevine.zip; \
    #
    # clean up
    apt-get --purge autoremove -y unzip; \
    apt-get clean -y; \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# Stage 2: Final image
FROM --platform=linux/amd64 base AS final

COPY supervisord.conf /etc/neko/supervisord/chromium.conf
COPY --chown=neko preferences.json /home/neko/.config/chromium/Default/Preferences
COPY policies.json /etc/chromium/policies/managed/policies.json
COPY openbox.xml /etc/neko/openbox.xml

# COPY --from=nginx-setup /etc/nginx /etc/nginx
RUN apt-get update && apt-get install -y nginx

# RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/