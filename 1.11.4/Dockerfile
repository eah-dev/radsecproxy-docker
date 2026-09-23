
# https://www.alpinelinux.org/
ARG ALPINE_VERSION=3.24
# https://github.com/radsecproxy/radsecproxy
ARG RADSECPROXY_VERSION=1.11.4

ARG TZ=Europe/Berlin

# ---- Build-Stage ----
FROM alpine:${ALPINE_VERSION} AS build

ARG RADSECPROXY_VERSION
ENV RADSECPROXY_VERSION=${RADSECPROXY_VERSION}

RUN apk add --no-cache \
    build-base \
    autoconf \
    automake \
    libtool \
    openssl-dev \
    pkgconfig \
    nettle-dev \
    bison \
    flex \
    curl \
    gnupg \
    tar

WORKDIR /src

# download version and extract it
RUN curl -sLo radsecproxy-${RADSECPROXY_VERSION}.tar.gz  \
        https://github.com/radsecproxy/radsecproxy/releases/download/${RADSECPROXY_VERSION}/radsecproxy-${RADSECPROXY_VERSION}.tar.gz && \
    curl  -sLo radsecproxy-${RADSECPROXY_VERSION}.tar.gz.asc \
        https://github.com/radsecproxy/radsecproxy/releases/download/${RADSECPROXY_VERSION}/radsecproxy-${RADSECPROXY_VERSION}.tar.gz.asc && \
    curl -sS https://radsecproxy.github.io/fabian.mauchle.asc | gpg --import - && \
    gpg --verify radsecproxy-${RADSECPROXY_VERSION}.tar.gz.asc \
                 radsecproxy-${RADSECPROXY_VERSION}.tar.gz && \
    rm  radsecproxy-${RADSECPROXY_VERSION}.tar.gz.asc && \
    tar xvf radsecproxy-${RADSECPROXY_VERSION}.tar.gz && \
    rm radsecproxy-${RADSECPROXY_VERSION}.tar.gz && \
    cd radsecproxy-${RADSECPROXY_VERSION} && \
    ./configure --prefix=/ && \
    make && \
    make check && \
    make install DESTDIR=/tmp/radsecproxy-install

# ---- Runtime-Stage ----
FROM alpine:${ALPINE_VERSION}

ARG RADSECPROXY_VERSION
ENV RADSECPROXY_VERSION=${RADSECPROXY_VERSION}
ARG TZ
ENV TZ=${TZ}

LABEL \
    org.opencontainers.image.authors="Carsten Hölbing <carsten.hoelbing@eah-jena.de>" \
    org.opencontainers.image.vendor="Ernst-Abbe-Hochschule Jena" \
    org.opencontainers.image.source="https://github.com/eah-dev/radsecproxy-docker" \
    org.opencontainers.image.url="https://github.com/eah-dev/radsecproxy-docker" \
    org.opencontainers.image.licenses="GPL-2.0" \
    org.opencontainers.image.revision="${GIT_COMMIT_HASH:-unknown}" \
    org.opencontainers.image.version="${RADSECPROXY_VERSION}"

RUN apk add --no-cache \
    ca-certificates \
    openssl \
    nettle \
    tzdata && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone

RUN addgroup -S radsec && adduser -S radsec -G radsec

COPY --from=build /tmp/radsecproxy-install/ /

RUN mkdir -p /etc/radsecproxy && \
    chown -R radsec:radsec /etc/radsecproxy

COPY radsecproxy.conf /etc/radsecproxy/radsecproxy.conf

ENV RADSECPROXY_CONFIG=/etc/radsecproxy/radsecproxy.conf

# ports
EXPOSE  \
    1812/udp \
    2083/tcp

USER radsec

CMD ["radsecproxy", "-f", "-c", "/etc/radsecproxy/radsecproxy.conf"]
