FROM alpine:edge

EXPOSE 1883
EXPOSE 9883


RUN addgroup -S mosquitto && \
    adduser -S -H -h /var/empty -s /sbin/nologin -D -G mosquitto mosquitto

ENV PATH=/usr/local/bin:/usr/local/sbin:$PATH
ENV MOSQUITTO_VERSION=v1.4.14

COPY libressl.patch /
RUN buildDeps='git postgresql-dev build-base libressl-dev libwebsockets-dev c-ares-dev util-linux-dev hiredis-dev curl-dev libxslt docbook-xsl'; \
    mkdir -p /var/lib/mosquitto && \
    touch /var/lib/mosquitto/.keep && \
    mkdir -p /etc/mosquitto.d && \
    apk update && \
    apk add $buildDeps hiredis libwebsockets libuuid c-ares libpq libressl curl ca-certificates && \
    git clone https://github.com/eclipse/mosquitto.git && \
    cd mosquitto && \
    git checkout ${MOSQUITTO_VERSION} -b ${MOSQUITTO_VERSION} && \
    sed -i -e "s|(INSTALL) -s|(INSTALL)|g" -e 's|--strip-program=${CROSS_COMPILE}${STRIP}||' */Makefile */*/Makefile && \
    sed -i "s@/usr/share/xml/docbook/stylesheet/docbook-xsl/manpages/docbook.xsl@/usr/share/xml/docbook/xsl-stylesheets-1.79.1/manpages/docbook.xsl@" man/manpage.xsl && \
    sed -i 's/ -lanl//' config.mk && \
    patch -p1 < /libressl.patch && \
    make WITH_MEMORY_TRACKING=no WITH_SRV=yes WITH_WEBSOCKETS=yes WITH_TLS_PSK=no && \
    make install && \
    git clone git://github.com/jpmens/mosquitto-auth-plug.git && \
    cd mosquitto-auth-plug && \
    cp config.mk.in config.mk && \
    sed -i "s/BACKEND_REDIS ?= no/WITH_SRV ?= no/" config.mk && \
    sed -i "s/BACKEND_HTTP ?= no/BACKEND_FILES ?= yes/" config.mk && \
    sed -i "s/BACKEND_POSTGRES ?= no/BACKEND_POSTGRES ?= yes/" config.mk && \
    sed -i "s/BACKEND_MYSQL ?= yes/BACKEND_MYSQL ?= no/" config.mk && \
    sed -i "s/MOSQUITTO_SRC =/MOSQUITTO_SRC = ..\//" config.mk && \
    sed -i "s/EVP_MD_CTX_new/EVP_MD_CTX_create/g" cache.c && \
    sed -i "s/EVP_MD_CTX_free/EVP_MD_CTX_destroy/g" cache.c && \
    make && \
    cp auth-plug.so /usr/local/lib/ && \
    cp np /usr/local/bin/ && chmod +x /usr/local/bin/np && \
    cd / && rm -rf mosquitto && rm /libressl.patch && \
    apk del $buildDeps && rm -rf /var/cache/apk/*

ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-alpine-linux-amd64-$DOCKERIZE_VERSION.tar.gz

COPY run.sh /
RUN chmod +x /run.sh
COPY ./mosquitto.conf /etc/mosquitto/
COPY ./acls /etc/mosquitto/
COPY ./users /etc/mosquitto/
COPY certs/* /etc/mosquitto/
COPY auth-plugin.conf.tmpl /etc/mosquitto.d/

ENTRYPOINT ["/run.sh"]
CMD ["mosquitto"]
