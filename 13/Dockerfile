FROM wonderfall/nginx-php:7.2

# developed for Erebos by Florian Kleber
MAINTAINER Florian Kleber <kleberbaum@erebos.xyz>

ARG NEXTCLOUD_VERSION=14.0.3
ARG GPG_nextcloud="2880 6A87 8AE4 23A2 8372  792E D758 99B9 A724 937A"

LABEL description="A server software for creating file hosting services" \
      nextcloud="Nextcloud v${NEXTCLOUD_VERSION}"

ENV UID=991 GID=991 \
    UPLOAD_MAX_SIZE=10G \
    APC_SHM_SIZE=128M \
    OPCACHE_MEM_SIZE=128 \
    MEMORY_LIMIT=512M \
    CRON_PERIOD=15m \
    CRON_MEMORY_LIMIT=1g \
    TZ=Etc/UTC \
    DB_TYPE=sqlite3 \
    DOMAIN=localhost

# update, install and cleaning
RUN echo "## Installing base ##" \
    ; \
    echo "@main http://dl-cdn.alpinelinux.org/alpine/edge/main/" >> /etc/apk/main && \
    echo "@testing http://dl-cdn.alpinelinux.org/alpine/edge/testing/" >> /etc/apk/repositories && \
    echo "@community http://dl-cdn.alpinelinux.org/alpine/edge/community/" >> /etc/apk/repositories && \
    apk upgrade --update-cache --available \
    ; \
    apk add --no-cache --virtual=build-dependencies \
        bash \
        gnupg \
        build-base \
        autoconf \
        automake \
        pcre-dev \
        libtool \
        samba-dev \
    ; \
    apk add --force \
        libressl \
        ca-certificates \
        libsmbclient \
        tzdata \
        tini@community \
    ; \
    pecl install \
        smbclient \
        apcu \
        redis \
    \
    && echo "extension=smbclient.so" > /php/conf.d/smbclient.ini \
    && echo "extension=redis.so" > /php/conf.d/redis.ini \
    && mkdir /nextcloud \
    && cd /tmp \
    && NEXTCLOUD_TARBALL="nextcloud-${NEXTCLOUD_VERSION}.tar.bz2" \
    && wget -q https://download.nextcloud.com/server/releases/${NEXTCLOUD_TARBALL} \
    && wget -q https://download.nextcloud.com/server/releases/${NEXTCLOUD_TARBALL}.sha512 \
    && wget -q https://download.nextcloud.com/server/releases/${NEXTCLOUD_TARBALL}.asc \
    && wget -q https://nextcloud.com/nextcloud.asc \
    && echo "Verifying both integrity and authenticity of ${NEXTCLOUD_TARBALL}..." \
    && CHECKSUM_STATE=$(echo -n $(sha512sum -c ${NEXTCLOUD_TARBALL}.sha512) | tail -c 2) \
    && if [ "${CHECKSUM_STATE}" != "OK" ]; then echo "Warning! Checksum does not match!" && exit 1; fi \
    && gpg --import nextcloud.asc \
    && FINGERPRINT="$(LANG=C gpg --verify ${NEXTCLOUD_TARBALL}.asc ${NEXTCLOUD_TARBALL} 2>&1 \
        | sed -n "s#Primary key fingerprint: \(.*\)#\1#p")" \
    && if [ -z "${FINGERPRINT}" ]; then echo "Warning! Invalid GPG signature!" && exit 1; fi \
    && if [ "${FINGERPRINT}" != "${GPG_nextcloud}" ]; then echo "Warning! Wrong GPG fingerprint!" && exit 1; fi \
    && echo "All seems good, now unpacking ${NEXTCLOUD_TARBALL}..." \
    && tar xjf ${NEXTCLOUD_TARBALL} --strip 1 -C /nextcloud \
    && update-ca-certificates \
    && apk del build-dependencies \
    && rm -rf /tmp/* /var/cache/apk/* /var/cache/distfiles/* /root/.gnupg

COPY rootfs /

RUN chmod +x /usr/local/bin/* /etc/s6.d/*/* /etc/s6.d/.s6-svscan/*

VOLUME /data /config /apps2 /nextcloud/themes

EXPOSE 8888

# starting via tini as init
ENTRYPOINT ["/sbin/tini", "--", "run.sh"]
