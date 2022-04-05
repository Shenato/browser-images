.PHONY: get-version build test tags

CURRENT_DIR:=$(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))
# TODO: reverse comments
# IMAGE_NAME="docker.pkg.github.com/nextools/images/firefox:rc"
IMAGE_NAME="ghcr.io/shenato/docker-chromium/firefox:rc"
# DOCKER_NAMESPACE="nextools"
DOCKER_NAMESPACE="eskimoe"

get-version:
	@docker pull ubuntu:bionic > /dev/null 2>&1
	@docker run --rm \
		ubuntu:bionic \
		sh -c "apt-get update --quiet=2 && apt-get --yes install software-properties-common > /dev/null 2>&1 && add-apt-repository --yes ppa:ubuntu-mozilla-daily/ppa > /dev/null 2>&1 && apt-get update --quiet=2 && apt-cache policy firefox-trunk | sed --regexp-extended --quiet 's/.*Candidate: ([0-9.]+)~([a-z0-9]+)\~.+/\1\2/p'"

build:
	@_version=`echo $(version) | sed --regexp-extended s/a/~a/`
	@docker buildx build --push --platform linux/arm64,linux/amd64 --tag $(IMAGE_NAME) --build-arg VERSION=$$_version $(CURRENT_DIR)
	@echo $(IMAGE_NAME)

test:
	@docker run --detach --publish 9222:9222 --name firefox $(IMAGE_NAME)
	@timeout 20s sh -c "trap 'docker container rm --force firefox' 0; until curl http://localhost:9222/json/version; do sleep 1; done"

# TODO: Change ghcr.io/shenato/docker-chromium/firefox to docker.pkg.github.com/nextools/images/firefox:$$tag;
tags:
	@docker pull ${IMAGE_NAME}
	@for tag in $(version) latest; do \
		echo $$tag; \
		docker tag $(IMAGE_NAME) ghcr.io/shenato/docker-chromium/firefox:$$tag; \
		docker tag $(IMAGE_NAME) ${DOCKER_NAMESPACE}/firefox:$$tag; \
		git tag --force firefox@$$tag >/dev/null 2>&1; \
	done
	@docker push ghcr.io/shenato/docker-chromium/firefox
	@docker push ${DOCKER_NAMESPACE}/firefox

