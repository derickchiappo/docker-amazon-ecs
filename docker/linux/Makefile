include .env

.DELETE_ON_ERROR:
.ONESHELL:
SHELL=bash
.SHELLFLAGS := -e -o pipefail -c
MAKEFLAGS += --warn-undefined-variables
MAKEFLAGS += --no-builtin-rules

.PHONY: help clean build publish

MODULE :=  $(MODULE)


## AWS
AWS_REGION ?= eu-west-2
AWS_ACCOINT := $(AWS_ACCOUNT)
IMAGE_DOMAIN ?= $(AWS_ACCOUNT).dkr.ecr.eu-west-2.amazonaws.com
IMAGE_REPO ?= $(IMAGE_DOMAIN)/docker-test-2

IMAGE_TAG := $(MODULE)

help: # generates a list of all targets with their descriptions
	@grep ":" ./Makefile | awk "/#/ && !/grep/" | sed -E 's/(.+:).*#(.+)/\x1b[36m\1\x1b[0m\2/g'

build:  # build image that requires lock files
	docker buildx build  --platform linux/amd64 -t $(IMAGE_REPO):$(IMAGE_TAG) -f Dockerfile.$(MODULE) --target prod .

publish: build #_ecr__login # publish docker images with lock files
	docker push $(IMAGE_REPO):$(IMAGE_TAG)

clean: # deletes all local intermediate files or cache
	@rm -r ./tmp || true
