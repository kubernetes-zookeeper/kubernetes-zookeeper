VERSION=1.0-3.8.2
PROJECT=kuberneteszookeeper
DOCKER_IMAGE=kubernetes-zookeeper


all: build

build:
	docker build --pull -t ${PROJECT}/${DOCKER_IMAGE}:${VERSION} .

push: build
	gcloud docker -- push ${PROJECT}/${DOCKER_IMAGE}:${VERSION}

.PHONY: all build push
