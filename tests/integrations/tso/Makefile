# Copyright 2023 TiKV Project Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ROOT_PATH := ../../..
GO_TOOLS_BIN_PATH := $(ROOT_PATH)/.tools/bin
PATH := $(GO_TOOLS_BIN_PATH):$(PATH)
SHELL := env PATH='$(PATH)' GOBIN='$(GO_TOOLS_BIN_PATH)' $(shell which bash)

static: install-tools
	@ gofmt -s -l -d . 2>&1 | awk '{ print } END { if (NR > 0) { exit 1 } }'
	@ golangci-lint run ./...
	@ revive -formatter friendly -config $(ROOT_PATH)/revive.toml .

tidy:
	@ go mod tidy
	git diff go.mod go.sum | cat
	git diff --quiet go.mod go.sum

test: failpoint-enable
	CGO_ENABLED=1 go test -v -tags deadlock -race -cover || { $(MAKE) failpoint-disable && exit 1; }
	$(MAKE) failpoint-disable

ci-test-job:
	CGO_ENABLED=1 go test -tags deadlock -race -covermode=atomic -coverprofile=covprofile -coverpkg=$(ROOT_PATH)/... github.com/tikv/pd/tests/integrations/tso

install-tools:
	cd $(ROOT_PATH) && $(MAKE) install-tools

failpoint-enable:
	cd $(ROOT_PATH) && $(MAKE) failpoint-enable
	go mod tidy

failpoint-disable:
	cd $(ROOT_PATH) && $(MAKE) failpoint-disable
	go mod tidy
