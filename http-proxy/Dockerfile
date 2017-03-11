FROM alpine
MAINTAINER Goritcly <goritcly@163.com>

ARG SS_VER=3.0.3
ARG SS_URL=https://github.com/shadowsocks/shadowsocks-libev/releases/download/v$SS_VER/shadowsocks-libev-$SS_VER.tar.gz

ARG KCP_VER=20170310
ARG KCP_URL=https://github.com/xtaci/kcptun/releases/download/v$KCP_VER/kcptun-linux-amd64-$KCP_VER.tar.gz

RUN set -ex && \
    sed -i 's/http:\/\/dl-cdn.alpinelinux.org/https:\/\/mirrors.ustc.edu.cn/g' /etc/apk/repositories && \
    apk add --no-cache --virtual .build-deps \
                                autoconf \
                                build-base \
                                curl \
                                libev-dev \
                                libtool \
                                linux-headers \
                                udns-dev \
                                libsodium-dev \
                                mbedtls-dev \
                                pcre-dev \
                                tar \
                                udns-dev && \
    mkdir -p /tmp/ss && \
    cd /tmp/ss && \
    curl -sSL $SS_URL | tar xz --strip 1 && \
    ./configure --prefix=/usr --disable-documentation && \
    make install && \
    cd / && \

    runDeps="$( \
        scanelf --needed --nobanner /usr/bin/ss-* \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | xargs -r apk info --installed \
            | sort -u \
    )" && \
    apk add --no-cache --virtual .run-deps $runDeps && \
    cd /tmp && \
    curl -sSL $KCP_URL | tar xz && \
    cp client_linux_amd64 /usr/bin/kcptun-client && \
    apk del .build-deps && \
    cd .. && \
    rm -rf /tmp/* && \
    apk add --no-cache privoxy supervisor && \
    sed -i 's/^listen-address.*$/listen-address  0.0.0.0:8118/g' /etc/privoxy/config && \
    echo 'forward-socks5 / 127.0.0.1:1080 .' >> /etc/privoxy/config

COPY kcptun.json /etc/
COPY shadowsocks.json /etc/
COPY supervisord.conf /etc/

EXPOSE 8118

CMD ["/usr/bin/supervisord"]
