#!/usr/bin/make -f

SHELL = /bin/bash
NS = lp/hugo-builder
PORTS = -p 1313:1313
VOLUMES = -v "$(realpath ./orgdocs)":/src

srcdir = .

default: build

build: static-analysis docker-build
start: hugo-server-start
stop: hugo-server-stop

static-analysis:
	@echo "Performing static analysis..."
	hadolint Dockerfile
	@echo "Static analysis performed!"

docker-build:
	@echo "Building Hugo Builder container..."
	docker build -t lp/hugo-builder .
	@echo "Hugo Builder container built!"
	docker images lp/hugo-builder

hugo-build:
	@echo "Building thing..."
	@docker run -dt --rm $(VOLUMES) lp/hugo-builder hugo
	sleep 1 # Pause a second to let this container stop.
	@echo "Thing built!"
	
hugo-server-start: hugo-build
	@echo "Starting server..."
	docker run -dt --rm $(VOLUMES) $(PORTS) lp/hugo-builder hugo server -w --bind=0.0.0.0
	@echo "Server started!"
	docker ps

hugo-server-stop:
	@echo "Stopping server..."
	docker stop `docker container ls -qf "ancestor=lp/hugo-builder"`
	@echo "Server stopped!"
	docker ps
