VERSION=1.0-3.9.3
PROJECT=kuberneteszookeeper
DOCKER_IMAGE=kubernetes-zookeeper


all: build

build:
	docker build --pull -t ${PROJECT}/${DOCKER_IMAGE}:${VERSION} .

push: build
	gcloud docker -- push ${PROJECT}/${DOCKER_IMAGE}:${VERSION}

.PHONY: all build push
