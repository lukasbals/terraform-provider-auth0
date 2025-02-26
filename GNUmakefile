PKG_NAME = auth0
PKGS ?= $$(go list ./...)
FILES ?= $$(find . -name '*.go' | grep -v vendor)
TESTS ?= ".*"
COVERS ?= "c.out"
GOOS ?= $$(go env GOOS)
GOARCH ?= $$(go env GOARCH)
TEST_COUNT?=1
ACCTEST_TIMEOUT?=15m
ACCTEST_PARALLELISM?=10
VERSION="1.0.0"

default: build

build: fmtcheck
	@go install

install: build
	@mkdir -p ~/.terraform.d/plugins
	@cp $(GOPATH)/bin/terraform-provider-auth0 ~/.terraform.d/plugins

install-local:
	@go build -gcflags="all=-N -l"
	@mkdir -p ~/.terraform.d/plugins/local/alekc/auth0/$(VERSION)/$(GOOS)_$(GOARCH)/
	@mv terraform-provider-auth0 ~/.terraform.d/plugins/local/alekc/auth0/$(VERSION)/$(GOOS)_$(GOARCH)/terraform-provider-auth0_v$(VERSION)

sweep:
	@echo "WARNING: This will destroy infrastructure. Use only in development accounts."
	@go test ./auth0 -v -sweep="phony" $(SWEEPARGS)

test: fmtcheck
	@go test -i $(PKGS) || exit 1
	@echo $(PKGS) | \
		xargs -t -n4 go test $(TESTARGS) -timeout=30s -parallel=10 -run ^$(TESTS)$

testacc: fmtcheck
	TF_ACC=1 go test ./$(PKG_NAME) -v -count $(TEST_COUNT) -parallel $(ACCTEST_PARALLELISM) $(TESTARGS) -timeout $(ACCTEST_TIMEOUT) -coverprofile=$(COVERS)

vet:
	@echo "go vet ."
	@go vet $$(go list ./... | grep -v vendor/) ; if [ $$? -eq 1 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

fmt:
	@gofmt -w $(FILES)

fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

errcheck:
	@sh -c "'$(CURDIR)/scripts/errcheck.sh'"

docgen:
	go run scripts/gendocs.go -resource auth0_<resource>

.PHONY: build test testacc vet fmt fmtcheck errcheck docgen
