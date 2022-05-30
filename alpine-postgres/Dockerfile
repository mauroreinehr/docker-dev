FROM alpine:3.16.0

ENV MUSL_LOCPATH="/usr/share/i18n/locales/musl"

RUN apk --no-cache add libintl && \
    apk --no-cache --virtual .locale_build add cmake make musl-dev gcc gettext-dev git && \
    git clone https://gitlab.com/rilian-la-te/musl-locales && \
    cd musl-locales && \
    cmake -DLOCALE_PROFILE=OFF -DCMAKE_INSTALL_PREFIX:PATH=/usr . && \
    make && make install && \
    cd .. && \
    rm -r musl-locales && \
    apk del .locale_build make make musl-dev gcc gettext-dev git

RUN set -eux; \
    addgroup -g 70 -S postgres; \
    adduser -u 70 -S -D -G postgres -H -h /var/lib/postgresql -s /bin/sh postgres; \
    mkdir -p /var/lib/postgresql; \
    chown -R postgres:postgres /var/lib/postgresql

ENV LANG pt_BR.utf8

RUN mkdir /docker-entrypoint-initdb.d

ENV PG_MAJOR 15
ENV PG_VERSION 15beta1

RUN set -eux; \
    \
    wget -O postgresql-$PG_VERSION.tar.bz2.sha256 "https://ftp.postgresql.org/pub/source/v$PG_VERSION/postgresql-$PG_VERSION.tar.bz2.sha256"; \
    wget -O postgresql-$PG_VERSION.tar.bz2 "https://ftp.postgresql.org/pub/source/v$PG_VERSION/postgresql-$PG_VERSION.tar.bz2"; \
    echo "$(cat postgresql-$PG_VERSION.tar.bz2.sha256)" | sha256sum -c -; \
    mkdir -p /usr/src/postgresql; \
    tar \
    --extract \
    --file postgresql-$PG_VERSION.tar.bz2 \
    --directory /usr/src/postgresql \
    --strip-components 1 \
    ; \
    rm postgresql-$PG_VERSION.tar.bz2; \
    rm postgresql-$PG_VERSION.tar.bz2.sha256; \
    \
    apk add --no-cache --virtual .build-deps \
    bison \
    coreutils \
    dpkg-dev dpkg \
    flex \
    gcc \
    krb5-dev \
    libc-dev \
    libedit-dev \
    libxml2-dev \
    libxslt-dev \
    linux-headers \
    llvm-dev clang g++ \
    make \
    openldap-dev \
    openssl-dev \
    perl-dev \
    perl-ipc-run \
    perl-utils \
    python3-dev \
    tcl-dev \
    util-linux-dev \
    zlib-dev \
    icu-dev \
    lz4-dev \
    ; \
    \
    cd /usr/src/postgresql; \
    awk '$1 == "#define" && $2 == "DEFAULT_PGSOCKET_DIR" && $3 == "\"/tmp\"" { $3 = "\"/var/run/postgresql\""; print; next } { print }' src/include/pg_config_manual.h > src/include/pg_config_manual.h.new; \
    grep '/var/run/postgresql' src/include/pg_config_manual.h.new; \
    mv src/include/pg_config_manual.h.new src/include/pg_config_manual.h; \
    gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
    wget -O config/config.guess 'https://git.savannah.gnu.org/cgit/config.git/plain/config.guess?id=7d3d27baf8107b630586c962c057e22149653deb'; \
    wget -O config/config.sub 'https://git.savannah.gnu.org/cgit/config.git/plain/config.sub?id=7d3d27baf8107b630586c962c057e22149653deb'; \
    ./configure \
    --build="$gnuArch" \
    --enable-integer-datetimes \
    --enable-thread-safety \
    --enable-tap-tests \
    --disable-rpath \
    --with-uuid=e2fs \
    --with-gnu-ld \
    --with-pgport=5432 \
    --with-system-tzdata=/usr/share/zoneinfo \
    --prefix=/usr/local \
    --with-includes=/usr/local/include \
    --with-libraries=/usr/local/lib \
    --with-krb5 \
    --with-gssapi \
    --with-ldap \
    --with-tcl \
    --with-perl \
    --with-python \
    --with-openssl \
    --with-libxml \
    --with-libxslt \
    --with-icu \
    --with-llvm \
    --with-lz4 \
    ; \
    make -j "$(nproc)" world; \
    make install-world; \
    make -C contrib install; \
    \
    runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
    | tr ',' '\n' \
    | sort -u \
    | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-cache --virtual .postgresql-rundeps \
    $runDeps \
    bash \
    su-exec \
    tzdata \
    zstd \
    ; \
    apk del --no-network .build-deps; \
    cd /; \
    rm -rf \
    /usr/src/postgresql \
    /usr/local/share/doc \
    /usr/local/share/man \
    ; \
    \
    postgres --version

RUN set -eux; \
    cp -v /usr/local/share/postgresql/postgresql.conf.sample /usr/local/share/postgresql/postgresql.conf.sample.orig; \
    sed -ri "s!^#?(listen_addresses)\s*=\s*\S+.*!\1 = '*'!" /usr/local/share/postgresql/postgresql.conf.sample; \
    grep -F "listen_addresses = '*'" /usr/local/share/postgresql/postgresql.conf.sample

RUN mkdir -p /var/run/postgresql && chown -R postgres:postgres /var/run/postgresql && chmod 2777 /var/run/postgresql

ENV PGDATA /var/lib/postgresql/data
RUN mkdir -p "$PGDATA" && chown -R postgres:postgres "$PGDATA" && chmod 777 "$PGDATA"
VOLUME /var/lib/postgresql/data

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

STOPSIGNAL SIGINT

EXPOSE 5432
CMD ["postgres"]

#BUILD
#docker build  --rm -f "alpine-postgres\Dockerfile" -t alpine-pg:3.16 "alpine-postgres"

#RUN
#docker run --name pg15 -e POSTGRES_PASSWORD=manager -e POSTGRES_HOST_AUTH_METHOD=trust -p 5433:5432 -d alpine-pg:3.16
