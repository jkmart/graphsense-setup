FROM golang:1.18.2-alpine3.15 AS gcsfuse

RUN apk add --no-cache git
ENV GOPATH /go
RUN go install github.com/googlecloudplatform/gcsfuse@latest

FROM alpine:3.12
LABEL maintainer="contact@graphsense.info"

ARG UID=10000

RUN apk --no-cache add make bash boost boost-program_options libevent libressl fuse shadow && \
    useradd -r -u $UID dockeruser &&  \
    mkdir -p /opt/graphsense/data && \
    chown -R dockeruser /opt/graphsense

# Add the GCS Fuse in order to mount GCP Cloud Storage
COPY --from=gcsfuse /go/bin/gcsfuse /usr/local/bin

ADD docker/Makefile /tmp/Makefile

RUN apk --no-cache --virtual build-dependendencies add \
        linux-headers \
        libressl-dev \
        g++ \
        boost-dev \
        file \
        autoconf \
        automake \
        libtool \
        libevent-dev \
        git \
        coreutils \
        binutils \
        grep && \
    cd /tmp; make install && \
    rm -rf /tmp/src && \
    strip /usr/local/bin/*bitcoin* && \
    apk del build-dependendencies

USER dockeruser
CMD ["bitcoind", "-conf=/opt/graphsense/client.conf", "-datadir=/opt/graphsense/data", "-rest"]
