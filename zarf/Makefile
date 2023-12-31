# The version of Zarf to use. To keep this repo as portable as possible the Zarf binary will be downloaded and added to
# the build folder.
ZARF_VERSION := v0.30.1

DUBBD_VERSION := 0.12.1

# Figure out which Zarf binary we should use based on the operating system we are on
ZARF_BIN := zarf
UNAME_S := $(shell uname -s)
UNAME_M := $(shell uname -m)
ifeq ($(UNAME_M),x86_64)
    ARCH := amd64
else ifeq ($(UNAME_M),amd64)
    ARCH := amd64
else ifeq ($(UNAME_M),arm64)
    ARCH := arm64
else
    $(error Unsupported architecture: $(UNAME_M))
endif

# Silent mode by default. Run `make VERBOSE=1` to turn off silent mode.
ifndef VERBOSE
.SILENT:
endif

# Optionally add the "-it" flag for docker run commands if the env var "CI" is not set (meaning we are on a local machine and not in github actions)
TTY_ARG :=
ifndef CI
	TTY_ARG := -it
endif

.DEFAULT_GOAL := help

# Idiomatic way to force a target to always run, by having it depend on this dummy target
FORCE:

.PHONY: help
help: ## Show a list of all targets
	grep -E '^\S*:.*##.*$$' $(MAKEFILE_LIST) \
	| sed -n 's/^\(.*\): \(.*\)##\(.*\)/\1:\3/p' \
	| column -t -s ":"

########################################################################
# Build Section
########################################################################

.PHONY: build/all
build/all: build build/zarf build/namespaces-pkg build/gitlab-deps-v1 build/gitlab-deps-v2 ## Build everything

build: ## Create build directory
	mkdir -p build

.PHONY: clean
clean: ## Clean up build files
	rm -rf ./build

.PHONY: build/zarf
build/zarf: | build ## Download the Zarf to the build dir
	if [ -f build/zarf ] && [ "$$(build/zarf version)" = "$(ZARF_VERSION)" ] ; then exit 0; fi && \
	echo "Downloading zarf" && \
	curl -sL https://github.com/defenseunicorns/zarf/releases/download/$(ZARF_VERSION)/zarf_$(ZARF_VERSION)_$(UNAME_S)_$(ARCH) -o build/zarf && \
	chmod +x build/zarf

build/namespaces-pkg: | build ## Build namespaces package
	cd build && ./zarf package create ../../zarf/examples/namespaces/ --confirm --output-directory .

build/gitlab-deps-v1: | build ## Build gitlab-deps-v1 packages
	cd build && ./zarf package create ../../zarf/examples/gitlab-deps-v1/minio --confirm --output-directory .
	cd build && ./zarf package create ../../zarf/examples/gitlab-deps-v1/postgres --confirm --output-directory .
	cd build && ./zarf package create ../../zarf/examples/gitlab-deps-v1/redis --confirm --output-directory .

build/gitlab-deps-v2: | build ## Build gitlab-deps-v1 packages
	cd build && ./zarf package create ../../zarf/examples/gitlab-deps-v2/minio --confirm --output-directory .
	cd build && ./zarf package create ../../zarf/examples/gitlab-deps-v2/postgres --confirm --output-directory .
	cd build && ./zarf package create ../../zarf/examples/gitlab-deps-v2/redis --confirm --output-directory .


########################################################################
# Deploy Section
########################################################################

deploy/zarf-init: ## Run zarf init
	cd ./build && ./zarf init --set REGISTRY_HPA_ENABLE=false --components git-server --confirm

deploy/dubbd: ## Deploy DUBBD
	cd ./build && ./zarf package deploy oci://ghcr.io/defenseunicorns/packages/dubbd-rke2:$(DUBBD_VERSION)-amd64 --confirm

deploy/namespace-pkg: ## Deploy Namespace package
	cd ./build && ./zarf package deploy zarf-package-gitlab-namespaces-amd64-0.0.1.tar.zst --confirm

deploy/gitlab-deps-v1: ## gitlab-deps-v1 packages
	cd ./build && ./zarf package deploy zarf-package-gitlab-minio-amd64-0.0.1.tar.zst --confirm
	cd ./build && ./zarf package deploy zarf-package-gitlab-postgres-amd64-0.0.1.tar.zst --confirm
	cd ./build && ./zarf package deploy zarf-package-gitlab-redis-amd64-0.0.1.tar.zst --confirm

deploy/gitlab-deps-v2: ## gitlab-deps-v1 packages
	cd ./build && ./zarf package deploy zarf-package-gitlab-minio-amd64-0.0.2.tar.zst --confirm
	cd ./build && ./zarf package deploy zarf-package-gitlab-postgres-amd64-0.0.2.tar.zst --confirm
	cd ./build && ./zarf package deploy zarf-package-gitlab-redis-amd64-0.0.2.tar.zst --confirm

deploy/uds-capability-gitlab: ## uds-capability-gitlab
	cd ./build && ./zarf package deploy oci://ghcr.io/defenseunicorns/uds-capability/gitlab:0.1.2-amd64 --confirm

########################################################################
# Macro Section
########################################################################

zarf-walkthrough/v1: build/all deploy/zarf-init deploy/namespace-pkg deploy/gitlab-deps-v1 ## Build and deploy

zarf-walkthrough/v2: build/all deploy/zarf-init deploy/dubbd deploy/namespace-pkg deploy/gitlab-deps-v2 deploy/uds-capability-gitlab ## Build and deploy

.PHONY: rebuild
rebuild: clean build/all
