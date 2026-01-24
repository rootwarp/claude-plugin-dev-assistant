# /golang:init-dir

Initialize a standard Go project with directory structure, Makefile, and .gitignore.

## Instructions

### Step 1: Gather Information

Ask the user:
1. **Project type:**
   - Minimal (recommended) - `cmd/` and `internal/`
   - Library - `pkg/` and `internal/`
   - Full Service - Complete structure with API, configs, deployments

2. **Application name** (used for `cmd/<app-name>/` and binary name)

3. **Module name** (e.g., `github.com/username/project`)

### Step 2: Create Directories

Follow @rules/project-layout.md conventions.

**Minimal Project:**
```bash
mkdir -p cmd/<app-name> internal/<app-name>
```

**Library Project:**
```bash
mkdir -p pkg internal
```

**Full Service Project:**
```bash
mkdir -p cmd/<app-name> internal/<app-name> pkg api configs scripts build/package build/ci deployments test/data docs
```

Add `.gitkeep` to empty directories:
```bash
find . -type d -empty -exec touch {}/.gitkeep \;
```

### Step 3: Initialize Go Module

```bash
go mod init <module-name>
```

### Step 4: Create main.go

For non-library projects, create `cmd/<app-name>/main.go`:

```go
package main

func main() {
}
```

### Step 5: Write Makefile

Create `Makefile` with the following content (replace `<app-name>` with actual name):

```makefile
APP_NAME := <app-name>
BUILD_DIR := bin
GO := go

.PHONY: all build run test lint fmt clean help

all: build

build: ## Build the application
	$(GO) build -o $(BUILD_DIR)/$(APP_NAME) ./cmd/$(APP_NAME)

run: build ## Run the application
	./$(BUILD_DIR)/$(APP_NAME)

test: ## Run tests
	$(GO) test -v -race ./...

test-coverage: ## Run tests with coverage
	$(GO) test -v -race -coverprofile=coverage.out ./...
	$(GO) tool cover -html=coverage.out -o coverage.html

lint: ## Run linter
	golangci-lint run ./...

fmt: ## Format code
	$(GO) fmt ./...
	goimports -w .

clean: ## Clean build artifacts
	rm -rf $(BUILD_DIR)
	rm -f coverage.out coverage.html

mod-tidy: ## Tidy go modules
	$(GO) mod tidy

help: ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'
```

For library projects, use this Makefile instead:

```makefile
GO := go

.PHONY: all test lint fmt clean help

all: test

test: ## Run tests
	$(GO) test -v -race ./...

test-coverage: ## Run tests with coverage
	$(GO) test -v -race -coverprofile=coverage.out ./...
	$(GO) tool cover -html=coverage.out -o coverage.html

lint: ## Run linter
	golangci-lint run ./...

fmt: ## Format code
	$(GO) fmt ./...
	goimports -w .

clean: ## Clean artifacts
	rm -f coverage.out coverage.html

mod-tidy: ## Tidy go modules
	$(GO) mod tidy

help: ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'
```

### Step 6: Write .gitignore

Create `.gitignore`:

```gitignore
# Binaries
bin/
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test artifacts
*.test
coverage.out
coverage.html

# Dependency directories
vendor/

# IDE
.idea/
.vscode/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Environment files
.env
.env.local
*.env

# Build artifacts
dist/
build/output/

# Temporary files
tmp/
*.tmp
*.log
```

### Step 7: Write .golangci.yml

Create `.golangci.yml` with a balanced linter configuration (replace `<module-name>` with the actual module name):

```yaml
run:
  timeout: 5m

linters:
  enable:
    - errcheck
    - govet
    - staticcheck
    - gosimple
    - ineffassign
    - unused
    - typecheck
    - gofmt
    - goimports
    - errorlint
    - gosec
    - revive

linters-settings:
  errcheck:
    check-type-assertions: true
  govet:
    enable-all: true
  staticcheck:
    checks: ["all"]
  goimports:
    local-prefixes: <module-name>

issues:
  exclude-use-default: false
  max-issues-per-linter: 0
  max-same-issues: 0
```

**Linter rationale:**
- `errcheck`: Catches unchecked errors (aligns with error handling rules)
- `govet`: Detects suspicious constructs
- `staticcheck`: Comprehensive static analysis
- `gosimple/ineffassign/unused`: Identifies dead or unused code
- `typecheck`: Validates type correctness
- `gofmt/goimports`: Enforces formatting standards
- `errorlint`: Ensures proper error wrapping with `%w`
- `gosec`: Catches security issues
- `revive`: Extensible linter with sensible defaults (golint replacement)

### Step 8: Display Result

Show the created structure using `tree` or `ls -la` and confirm completion.

## Rules

- Never create `/src` directory
- Keep `cmd/` minimal; put logic in `internal/` or `pkg/`
- Use `internal/` for private code
- Use `pkg/` only when external import is intended
