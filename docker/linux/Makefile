include .env

.DELETE_ON_ERROR:
.ONESHELL:
SHELL=bash
.SHELLFLAGS := -e -o pipefail -c
MAKEFLAGS += --warn-undefined-variables
MAKEFLAGS += --no-builtin-rules

.PHONY: help clean build__gambi _ecr__login

export AWS_ACCESS_KEY_ID := $(AWS_ACCESS_KEY_ID)
export AWS_SECRET_ACCESS_KEY := $(AWS_SECRET_ACCESS_KEY)
export DOCKER_DEFAULT_PLATFORM = linux/amd64

MODULE ?= $(MODULE_NAME)
LOCK_FILE ?= ./env/$(MODULE)/$(MODULE).linux64.conda-lock.yml

## AWS
AWS_REGION ?= eu-west-2
AWS_USERNAME ?= pipelines-cicd
IMAGE_DOMAIN ?= $(AWS_ACCOUNT).dkr.ecr.eu-west-2.amazonaws.com
IMAGE_REPO ?= $(IMAGE_DOMAIN)/nautilus-pipeline-dbcreator

# Get hash of image
UNAME := $(shell uname)
ifneq ("$(wildcard $(LOCK_FILE))","")
	ifeq ($(UNAME), Darwin)
		LOCK_FILE_HASH := $(shell md5 $(LOCK_FILE) | rev | cut -f1 -d" " )
	else
		LOCK_FILE_HASH := $(shell md5sum $(LOCK_FILE) | cut -f1 -d" " )
	endif
endif

IMAGE_TAG := $(MODULE)-$(LOCK_FILE_HASH)


help: # generates a list of all targets with their descriptions
	@grep ":" ./Makefile | awk "/#/ && !/grep/" | sed -E 's/(.+:).*#(.+)/\x1b[36m\1\x1b[0m\2/g'

_ecr__login:
	aws ecr get-login-password --region $(AWS_REGION) | docker login --username AWS --password-stdin $(IMAGE_DOMAIN)

build:  # build image that requires lock files
	docker buildx build  --platform linux/amd64 -t $(IMAGE_REPO):$(IMAGE_TAG) -f Dockerfile.$(MODULE) --target prod .

publish: build _ecr__login # publish docker images with lock files
	docker push $(IMAGE_REPO):$(IMAGE_TAG)

clean: # deletes all local intermediate files or cache
	@rm -r ./tmp || true
