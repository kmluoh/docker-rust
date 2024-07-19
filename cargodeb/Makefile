SHELL = /bin/bash
.PHONY: all src build
all: src

src:
	patch ../Dockerfile-debian.template patches/debian-cargodeb.patch -o Dockerfile-debian.template
	patch ../Dockerfile-alpine.template patches/alpine-cargodeb.patch -o Dockerfile-alpine.template
	patch ../Dockerfile-slim.template patches/slim-cargodeb.patch -o Dockerfile-slim.template
	./x.py update

build: src
	@while IFS=$$'\t' read -r name context platforms tags _; do \
		pushd $$context; \
		IFS=',' read -ra TAGS <<< "$$tags"; \
		for tag in "$${TAGS[@]}"; do \
		  echo "docker build --platform $$platforms -t $$tag ."; \
		  docker build --platform $$platforms -t $$tag .; \
		done; \
		popd; \
	done < <(yq e '.matrix.include[] | [.name, .context, .platforms, .tags] | @tsv' build.yml)

