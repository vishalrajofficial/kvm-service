# Copyright 2021 Authors of kvmservice
# SPDX-License-Identifier: Apache-2.0

ifeq (,$(shell which govvv))
$(shell go install github.com/ahmetb/govvv@latest)
endif

CURDIR          := $(shell pwd)
CONTRIBDIR      := $(realpath $(CURDIR)/../contribution)
GO_EXEC         := $(shell which go)
LOGNAME         := $(shell logname)
NETNEXT         := 1
DLV_EXEC         = $(GOPATH)/bin/dlv
DLV_LPORT       := 2345
DLV_RPORT       := $(shell expr $(DLV_LPORT) + $(NETNEXT))
KVMSERVICE_PID   = $(shell pgrep kvmservice)
GIT_INFO        := $(shell govvv -flags)

.PHONY: build
build:
	cd $(CURDIR); go mod tidy
	cd $(CURDIR); go build -ldflags "-w -s ${GIT_INFO}" -o kvmservice main.go

.PHONY: build-test
build-test:
	cd $(CURDIR); go mod tidy
	cd $(CURDIR); sudo -E $(GO_EXEC) test -covermode=atomic -coverpkg=./... -c . -o kvmservice

.PHONY: run
run: build
	cd $(CURDIR); sudo rm -f /tmp/kvmservice.log
	cd $(CURDIR); sudo -E ./kvmservice -port=32769 -ipAddress="1.1.1.1" #-logPath=/tmp/kvmservice.log

.PHONY: run-per-pod
run-per-pod: build
	cd $(CURDIR); sudo rm -f /tmp/kvmservice.log
	cd $(CURDIR); sudo -E ./kvmservice #-logPath=/tmp/kvmservice.log -enableEnforcerPerPod

.PHONY: run-with-host
run-with-host: build
	cd $(CURDIR); sudo rm -f /tmp/kvmservice.log
	cd $(CURDIR); sudo -E ./kvmservice #-logPath=/tmp/kvmservice.log -enableHostPolicy -enableExternalWorkloadPolicy

.PHONY: test
test:
	cd $(CURDIR)/feeder; go mod tidy
	cd $(CURDIR)/feeder; go clean -testcache .; go test -v .

.PHONY: testall
testall:
	cd $(CURDIR)/feeder; go mod tidy
	cd $(CURDIR)/feeder; go clean -testcache .; go test -v -coverprofile=.coverprofile .
	cd $(CURDIR)/enforcer; go mod tidy
	cd $(CURDIR)/enforcer; go clean -testcache .; sudo -E $(GO_EXEC) test -v -coverprofile=.coverprofile .
	cd $(CURDIR)/monitor; go mod tidy
	cd $(CURDIR)/monitor; go clean -testcache .; sudo -E $(GO_EXEC) test -v -coverprofile=.coverprofile .

.PHONY: gofmt
gofmt:
	cd $(CURDIR); gofmt -s -d $(shell find . -type f -name '*.go' -print)

.PHONY: golint
golint:
	golangci-lint run
#ifeq (, $(shell which golint))
#	@{ \
#	set -e ;\
#	GOLINT_TMP_DIR=$$(mktemp -d) ;\
#	cd $$GOLINT_TMP_DIR ;\
#	go mod init tmp ;\
#	go get -u golang.org/x/lint/golint ;\
#	rm -rf $$GOLINT_TMP_DIR ;\
#	}
#endif
#	cd $(CURDIR); golint ./...

.PHONY: gosec
gosec:
ifeq (, $(shell which gosec))
	@{ \
	set -e ;\
	GOSEC_TMP_DIR=$$(mktemp -d) ;\
	cd $$GOSEC_TMP_DIR ;\
	go mod init tmp ;\
	go get -u github.com/securego/gosec/v2/cmd/gosec ;\
	rm -rf $$GOSEC_TMP_DIR ;\
	}
endif
	cd $(CURDIR); gosec ./...

.PHONY: clean
clean:
	cd $(CURDIR); sudo rm -f kvmservice /tmp/kvmservice.log
	cd $(CURDIR); find . -name .coverprofile | xargs -I {} rm {}
	$(CURDIR)/build/clean_source_files.sh

.PHONY: vagrant-check
vagrant-check:
ifeq ($(LOGNAME), vagrant)
	$(error rule must be called from outside the vagrant environment)
endif

.PHONY: vagrant-up
vagrant-up: vagrant-check
	cd $(CONTRIBDIR)/vagrant; NETNEXT=$(NETNEXT) DLV_RPORT=$(DLV_RPORT) vagrant up; true

.PHONY: vagrant-status
vagrant-status: vagrant-check
	cd $(CONTRIBDIR)/vagrant; NETNEXT=$(NETNEXT) DLV_RPORT=$(DLV_RPORT) vagrant status; true

.PHONY: vagrant-reload
vagrant-reload: vagrant-check
	cd $(CONTRIBDIR)/vagrant; NETNEXT=$(NETNEXT) DLV_RPORT=$(DLV_RPORT) vagrant reload; true

.PHONY: vagrant-ssh
vagrant-ssh: vagrant-check
	cd $(CONTRIBDIR)/vagrant; NETNEXT=$(NETNEXT) DLV_RPORT=$(DLV_RPORT) vagrant ssh; true

.PHONY: vagrant-halt
vagrant-halt: vagrant-check
	cd $(CONTRIBDIR)/vagrant; NETNEXT=$(NETNEXT) DLV_RPORT=$(DLV_RPORT) vagrant halt; true

.PHONY: vagrant-destroy
vagrant-destroy: vagrant-check
	cd $(CONTRIBDIR)/vagrant; NETNEXT=$(NETNEXT) DLV_RPORT=$(DLV_RPORT) vagrant destroy; true

$(DLV_EXEC):
	go get -u github.com/go-delve/delve/cmd/dlv

.PHONY: debug-attach
debug-attach: $(DLV_EXEC)
ifeq ($(KVMSERVICE_PID), )
	$(error kvmservice must be running - execute 'make run' first)
endif
	sudo $(DLV_EXEC) attach $(KVMSERVICE_PID) --headless -l=:$(DLV_LPORT) --log --api-version 2 $(CURDIR)
