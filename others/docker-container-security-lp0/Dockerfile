FROM alpine:3.12

LABEL maintainer="solvaholic <me@foo.bar>"

RUN apk add --no-cache \
    curl \
    git \
    openssh-client \
    rsync

ENV VERSION 0.64.0
ENV SHASUM 99c4752bd46c72154ec45336befdf30c28e6a570c3ae7cc237375cf116cba1f8
ENV HUGOTGZ hugo_${VERSION}_Linux-64bit.tar.gz

HEALTHCHECK --interval=1m --timeout=10s \
  CMD curl -f http://localhost:1313 || exit 1

SHELL ["/bin/ash", "-eo", "pipefail", "-c"]

# TODO: Put downloads in a volume.

WORKDIR /usr/local/src/hugo

ADD https://github.com/gohugoio/hugo/releases/download/v${VERSION}/${HUGOTGZ} .
RUN sha256sum ${HUGOTGZ} | grep -q ${SHASUM}
RUN tar -xzf ${HUGOTGZ} \
    && mv hugo /usr/local/bin/hugo \
    && addgroup -Sg 1000 hugo \
    && adduser -SG hugo -u 1000 -h /src hugo

WORKDIR /src

EXPOSE 1313
