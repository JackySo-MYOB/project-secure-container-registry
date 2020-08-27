# project-secure-container-registry

The deliverables for this milestone are a Dockerfile and a Makefile.

The Dockerfile should have no Static Analysis issues, should implement a health check, and should use a checksum to verify downloaded content.

The Makefile should have targets to build the container image, build the OrgDocs website, run the OrgDocs Hugo server, and perform Static Analysis on the Hugo build container. Other targets that help during the process may be useful too.

Upload a link to your files (preferably hosted on GitHub) in the blank below and choose submit. After submitting, you can view an example solution in the next section.

Your answers may not look the same as the Dockerfile or Makefile shown here, but should implement the core aspects outlined in the Submit your Work section.

##Dockerfile

In the Dockerfile, I addressed the Hadolint issues in the following ways:

DL3007 - Added a variable for the alpine version of the base box to use. I used an ARG with a default value to enable me to pass the value in from a build pipeline.
DL4000 - Removed the MAINTAINER keyword and replaced it with a LABEL so I can inspect this in the built container.
DL3003 - Added the WORKDIR, which creates the directory if it doesn’t exist, and then I removed the relevant parts of the Bash script.
DL4006 - Added the SHELL ["/bin/ash", "-eo", "pipefail", "-c"] as recommended in the Hadolint rule documentation for an alpine container.
DL3018 - Ignored this, as dealt with in the command I use when analyzing. See Makefile below.
I added a HEALTHCHECK command that checks the hugo env.

I then removed the minify download as it is not needed, before using a checksum to verify the Hugo download in the build.

```
ARG ALPINE_VERSION=3.11.5

FROM alpine:${ALPINE_VERSION}

LABEL maintainer="psellars@gmail.com"

RUN apk add --no-cache \
    curl \
    git \
    openssh-client \
    rsync

ENV VERSION 0.64.0

WORKDIR /usr/local/src
SHELL ["/bin/ash", "-eo", "pipefail", "-c"]

RUN wget \
  https://github.com/gohugoio/hugo/releases/download/v${VERSION}/hugo_${VERSION}_Linux-64bit.tar.gz

RUN wget \
  https://github.com/gohugoio/hugo/releases/download/v${VERSION}/hugo_${VERSION}_checksums.txt \
    && sed -i '/hugo_[0-9].*Linux-64bit.tar.gz/!d' \
       hugo_${VERSION}_checksums.txt \
    && sha256sum -cs hugo_${VERSION}_checksums.txt \
    && tar -xzvf hugo_"${VERSION}"_Linux-64bit.tar.gz \

    && mv hugo /usr/local/bin/hugo \

    && addgroup -Sg 1000 hugo \
    && adduser -SG hugo -u 1000 -h /src hugo

WORKDIR /src

EXPOSE 1313

HEALTHCHECK --interval=10s --timeout=10s --start-period=15s \
  CMD hugo env || exit 1
```
##Makefile

In the Makefile I created a number of targets to start building the container pipeline.

The default target is the all target, which runs the build and lint targets. The thing to note here is the use of the lp namespace for the container build and the --ignore DL3018 during the lint stage. Note too that the hadolint container has a defined version.

The hugo_build and start_server targets highlight how to use either bind mounts or shared volumes and port mappings to build and serve the OrgDocs site.

Additionally, I added targets to stop the server, check the health of and inspect the labels of a running container.

It is a very simple Makefile as I am trying to remove complexity and replace it with clarity.

```
default: all

all: build lint

build:
    @echo "Building Hugo Builder container..."
    @docker build -t lp/hugo-builder .
    @echo "Hugo Builder container built!"
    @docker images lp/hugo-builder

lint:
    @echo "Linting the Hugo Builder container..."
    @docker run --rm -i hadolint/hadolint:v1.17.5-alpine \
        hadolint --ignore DL3018 - < Dockerfile
    @echo "Linting completed!"

#hugo_build:
#   @echo "Building the OrgDocs Hugo site..."
#   @docker run --rm -it -v $(PWD)/orgdocs:/src -u hugo lp/hugo-builder hugo
#   @echo "OrgDocs Hugo site built!"

hugo_build:
    @echo "Building the OrgDocs Hugo site..."
    @docker run --rm -it \
        --mount type=bind,src=${PWD}/orgdocs,dst=/src -u hugo \
        lp/hugo-builder hugo
    @echo "OrgDocs Hugo site built!"

#start_server:
#   @echo "Serving the OrgDocs Hugo site..."
#   @docker run -d --rm -it -v $(PWD)/orgdocs:/src -p 1313:1313 -u hugo \
#       --name hugo_server lp/hugo-builder hugo server -w --bind=0.0.0.0
#   @echo "OrgDocs Hugo site being served!"
#   @docker ps --filter name=hugo_server

start_server:
    @echo "Serving the OrgDocs Hugo site..."
    @docker run -d --rm -it --name hugo_server \
        --mount type=bind,src=${PWD}/orgdocs,dst=/src -u hugo \
        -p 1313:1313 lp/hugo-builder hugo server -w --bind=0.0.0.0
    @echo "OrgDocs Hugo site being served!"
    @docker ps --filter name=hugo_server

check_health:
    @echo "Checking the health of the Hugo Server..."
    @docker inspect --format='{{json .State.Health}}' hugo_server

    
stop_server:
    @echo "Stopping the OrgDocs Hugo site..."
    @docker stop hugo_server
    @echo "OrgDocs Hugo site stopped!"

inspect_labels:
    @echo "Inspecting Hugo Server Container labels..."
    @echo "maintainer set to..."
    @docker inspect --format '{{ index .Config.Labels "maintainer" }}' \
        hugo_server
    @echo "Labels inspected!"

.PHONY: build lint hugo_build start_server check_health stop_server \
  inspect_labels
```

[docker-container-security-lp-private-milestone_1.zip](https://liveproject-resources.s3.amazonaws.com/139/29228/2020-08-04-11-28-38/docker-container-security-lp-private-milestone_1.zip)
