############################################################################
# Variables
############################################################################

# Set this to change the target installation path
DESTDIR ?= /usr/local
BINDIR   = ${DESTDIR}/bin
MANDIR   = ${DESTDIR}/share/man
DISTDIR ?= dist

# filename of the executable
exe = direnv$(shell go env GOEXE)

# Override the go executable
GO = go

# Change if you want to fork direnv
PACKAGE = github.com/direnv/direnv

# BASH_PATH can also be passed to hard-code the path to bash at build time

SHELL = bash

############################################################################
# Common
############################################################################

.PHONY: all
all: build man

export GO111MODULE=on
export GOFLAGS=-mod=vendor

############################################################################
# Build
############################################################################

.PHONY: build
build: direnv

.PHONY: clean
clean:
	rm -rf \
		.gopath \
		direnv

GO_LDFLAGS =

ifeq ($(shell uname), Darwin)
	# Fixes DYLD_INSERT_LIBRARIES issues
	# See https://github.com/direnv/direnv/issues/194
	GO_LDFLAGS += -linkmode=external
endif

ifdef BASH_PATH
	GO_LDFLAGS += -X main.bashPath=$(BASH_PATH)
endif

ifneq ($(strip $(GO_LDFLAGS)),)
	GO_BUILD_FLAGS = -ldflags '$(GO_LDFLAGS)'
endif

direnv: stdlib.go *.go
	$(GO) build $(GO_BUILD_FLAGS) -o $(exe)

stdlib.go: stdlib.sh
	cat $< | ./script/str2go main StdLib $< > $@

version.go: version.txt
	echo package main > $@
	echo >> $@
	echo "// Version is direnv's version"
	echo 'const Version = "$(shell cat $<)"' >> $@

############################################################################
# Format all the things
############################################################################
.PHONY: fmt fmt-go fmt-sh
fmt: fmt-go fmt-sh

fmt-go:
	$(GO) fmt

fmt-sh:
	@command -v shfmt >/dev/null || (echo "Could not format stdlib.sh because shfmt is missing. Run: go get -u mvdan.cc/sh/cmd/shfmt"; false)
	shfmt -i 2 -w stdlib.sh

############################################################################
# Documentation
############################################################################

man_md = $(wildcard man/*.md)
roffs = $(man_md:.md=)

.PHONY: man
man: $(roffs)

%.1: %.1.md
	@command -v go-md2man >/dev/null || (echo "Could not generate man page because go-md2man is missing. Run: go get -u github.com/cpuguy83/go-md2man"; false)
	go-md2man -in $< -out $@

############################################################################
# Testing
############################################################################

tests = \
				test-shellcheck \
				test-stdlib \
				test-go \
				test-go-lint \
				test-go-fmt \
				test-bash \
				test-elvish \
				test-fish \
				test-tcsh \
				test-zsh

.PHONY: $(tests)
test: build $(tests)
	@echo
	@echo SUCCESS!

test-shellcheck:
	shellcheck stdlib.sh

test-stdlib: build
	./test/stdlib.bash

test-go:
	$(GO) test -v ./...

test-go-lint:
	golangci-lint run

test-bash:
	bash ./test/direnv-test.bash

# Needs elvish 0.12+
test-elvish:
	elvish ./test/direnv-test.elv

test-fish:
	fish ./test/direnv-test.fish

test-tcsh:
	tcsh -e ./test/direnv-test.tcsh

test-zsh:
	zsh ./test/direnv-test.zsh

############################################################################
# Installation
############################################################################

.PHONY: install
install: all
	install -d $(BINDIR)
	install $(exe) $(BINDIR)
	install -d $(MANDIR)/man1
	cp -R man/*.1 $(MANDIR)/man1

.PHONY: dist
dist:
	@command -v gox >/dev/null || (echo "Could not generate dist because gox is missing. Run: go get -u github.com/mitchellh/gox"; false)
	gox -output "$(DISTDIR)/direnv.{{.OS}}-{{.Arch}}"

