SHELL = /bin/bash
DISTROS := alpine debian slim
TEMPLATES := $(patsubst %,Dockerfile-%.template,$(DISTROS))
STABLE_RUST_VERSION := $(shell sed -n 's/^stable_rust_version = "\([^"]*\)"/\1/p' x.py)
BUILD := $(shell git rev-list --count HEAD)
DOCKERFILES := $(shell find . -name Dockerfile -type f | grep -v nightly | grep -v slim)
SLIM_DOCKFILES := $(shell find . -name Dockerfile -type f | grep -v nightly | grep slim)


.PHONY: all dockerfile dockerfile.clean build $(DOCKERFILES) $(SLIM_DOCKERFILES)

all: dockerfile

$(TEMPLATES): %:../% patches/%.patch
	@patch $^ -o $@

clean.dockerfile:
	rm -rf nightly $(STABLE_RUST_VERSION) $(TEMPLATES)

dockerfile: $(TEMPLATES) x.py
	./x.py update

build: $(DOCKERFILES) $(SLIM_DOCKERFILES)

$(DOCKERFILES):
	base=`sed -n 's/^FROM \(.*\)/\1/p' $@` && \
	plats=`docker manifest inspect $$base | jq '.manifests.[].platform | [.os, .architecture, .variant] | @tsv' | sed 's,\\\t,\\/,g;s,",,g;s,/$$,,g' | grep -v unknown | grep -v mips64` && \
	platforms=`echo $$plats | tr ' ' ,` && \
	tag=$(patsubst $(STABLE_RUST_VERSION)/%,$(STABLE_RUST_VERSION)-%-$(BUILD),$(@D)) && \
	echo -n "docker build $(@D) -t $$tag --platform $$platforms ... " && \
	docker build $(@D) -t $$tag --platform $$platforms --progress=plain &> $(@D)/build.log && \
	echo " done." && \
	docker image tag $$tag kmluoh/cargo-deb:$$tag && \
	docker push kmluoh/cargo-deb:$$tag && \
	docker image tag $$tag kmluoh/rust:$$tag && \
	docker push kmluoh/rust:$$tag && \
	docker image rm $$tag kmluoh/cargo-deb:$$tag kmluoh/rust:$$tag && \
	git tag $$tag && git push origin $$tag

$(SLIM_DOCKERFILES):
	@base=`sed -n 's/^FROM \(.*\)/\1/p' $@` && \
	plats=`docker manifest inspect $$base | jq '.manifests.[].platform | [.os, .architecture, .variant] | @tsv' | sed 's,\\\t,\\/,g;s,",,g;s,/$$,,g' | grep -v unknown | grep -v mips64` && \
	platforms=`echo $$plats | tr ' ' ,` && \
	tag=$(patsubst $(STABLE_RUST_VERSION)/%/slim,$(STABLE_RUST_VERSION)-slim-%-$(BUILD),$(@D)) && \
	echo -n "docker build $(@D) -t $$tag --platform $$platforms ... " && \
	docker build $(@D) -t $$tag --platform $$platforms --progress=plain &> $(@D)/build.log && \
	echo " done." && \
	docker image tag $$tag kmluoh/cargo-deb:$$tag && \
	docker push kmluoh/cargo-deb:$$tag && \
	docker image tag $$tag kmluoh/rust:$$tag && \
	docker push kmluoh/rust:$$tag && \
	docker image rm $$tag kmluoh/cargo-deb:$$tag kmluoh/rust:$$tag && \
	git tag $$tag && git push origin $$tag
