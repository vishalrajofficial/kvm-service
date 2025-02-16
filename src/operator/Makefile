# Copyright 2021 Authors of kvms-operator
# SPDX-License-Identifier: Apache-2.0

CURDIR          := $(shell pwd)
CONTRIBDIR      := $(realpath $(CURDIR)/../contribution)
GO_EXEC         := $(shell which go)
LOGNAME         := $(shell logname)
NETNEXT         := 1
DLV_EXEC         = $(GOPATH)/bin/dlv
DLV_LPORT       := 2345
DLV_RPORT       := $(shell expr $(DLV_LPORT) + $(NETNEXT))
KVMSOPERATOR_PID    = $(shell pgrep kvms-operator)

.PHONY: build
build:
	cd $(CURDIR); go mod tidy
	cd $(CURDIR); go build -o kvms-operator main.go

.PHONY: build-test
build-test:
	cd $(CURDIR); go mod tidy
	cd $(CURDIR); sudo -E $(GO_EXEC) test -covermode=atomic -coverpkg=./... -c . -o kvms-operator

.PHONY: run
run: build
	cd $(CURDIR); sudo rm -f /tmp/kvms-operator.log
	cd $(CURDIR); sudo -E ./kvms-operator #-logPath=/tmp/kvms-operator.log

.PHONY: run-per-pod
run-per-pod: build
	cd $(CURDIR); sudo rm -f /tmp/kvms-operator.log
	cd $(CURDIR); sudo -E ./kvms-operator #-logPath=/tmp/kvms-operator.log -enableEnforcerPerPod

.PHONY: run-with-host
run-with-host: build
	cd $(CURDIR); sudo rm -f /tmp/kvms-operator.log
	cd $(CURDIR); sudo -E ./kvms-operator #-logPath=/tmp/kvms-operator.log -enableHostPolicy -enableExternalWorkloadPolicy

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
ifeq (, $(shell which golint))
	@{ \
	set -e ;\
	GOLINT_TMP_DIR=$$(mktemp -d) ;\
	cd $$GOLINT_TMP_DIR ;\
	go mod init tmp ;\
	go get -u golang.org/x/lint/golint ;\
	rm -rf $$GOLINT_TMP_DIR ;\
	}
endif
	cd $(CURDIR); golint ./...

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
	cd $(CURDIR); sudo rm -f kvms-operator /tmp/kvms-operator.log
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
ifeq ($(KVMSOPERATOR_PID), )
	$(error kvms-operator must be running - execute 'make run' first)
endif
	sudo $(DLV_EXEC) attach $(KVMSOPERATOR_PID) --headless -l=:$(DLV_LPORT) --log --api-version 2 $(CURDIR)
