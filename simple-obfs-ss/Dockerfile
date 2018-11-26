FROM shadowsocks/shadowsocks-libev:v3.2.1
LABEL MAINTAINER="Mo <googolmo@gmail.com>"

USER root

ENV SIMPLE_OBFS_VER 0.0.5

RUN set -ex && \
    apk add --no-cache --virtual .build-deps \
            autoconf \
            make \
            libtool \
            automake \
            zlib-dev \
            openssl \
            asciidoc \
            xmlto \
            libpcre32 \
            g++ \
            linux-headers \
            git && \
    apk add --no-cache --virtual .run-deps \
        libev-dev && \
    cd /tmp && \
    git clone https://github.com/shadowsocks/simple-obfs.git && \
    cd simple-obfs && \
    git checkout v$SIMPLE_OBFS_VER && \
    git submodule update --init --recursive && \
    ./autogen.sh && \
    ./configure --disable-documentation &&\
    make install && \
    runDeps="$( \
        scanelf --needed --nobanner /usr/bin/obfs-* \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | xargs -r apk info --installed \
            | sort -u \
    )" && \
    apk add --no-cache --virtual .run-deps2 \
        rng-tools \
        $runDeps && \
    apk del .build-deps && \
    rm -rf /tmp/*

ENV OBFS http
ENV FAILOVER www.taobao.com
USER nobody

CMD exec ss-server \
      -s $SERVER_ADDR \
      -p $SERVER_PORT \
      -k ${PASSWORD:-$(hostname)} \
      -m $METHOD \
      -t $TIMEOUT \
      --fast-open \
      -d $DNS_ADDRS \
      -u \
      --plugin /usr/local/bin/obfs-server \
      --plugin-opts obfs=$OBFS;obfs-host=$FAILOVER \
      $ARGS

