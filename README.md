# claude-plugin-dev-assistant

A collection of Claude Code plugins for Go, Rust, and Python development.

## Installation

### From GitHub

```bash
# Install specific plugin
/plugin install rootwarp/claude-plugin-dev-assistant/plugins/go
/plugin install rootwarp/claude-plugin-dev-assistant/plugins/rust
/plugin install rootwarp/claude-plugin-dev-assistant/plugins/python
```

### Local Development

```bash
claude --plugin-dir ./plugins/go
claude --plugin-dir ./plugins/rust
claude --plugin-dir ./plugins/python
```

## Available Plugins

### Go Plugin

| Command/Skill | Description |
|---------------|-------------|
| `/golang:init-dir` | Initialize Go project with directories, Makefile, and .gitignore |
| `/go-build` | Build the Go project |
| `/go-test` | Run tests with coverage |
| `/go-lint` | Run linters (golangci-lint or go vet) |
| `/go-mod` | Tidy Go modules |
| `/go-fmt` | Format Go source files |
| `/go-doc` | Look up Go documentation |

<details>
<summary>Usage Examples</summary>

```bash
# Initialize a new Go project (interactive)
/golang:init-dir

# Build the project
/go-build

# Run tests with verbose output and coverage
/go-test

# Run linters (uses golangci-lint if available, otherwise go vet)
/go-lint

# Format code with goimports
/go-fmt

# Tidy module dependencies
/go-mod

# Look up standard library documentation
/go-doc fmt
/go-doc net/http
/go-doc encoding/json Marshal
```

</details>

### Rust Plugin

| Command/Skill | Description |
|---------------|-------------|
| `/rust:init-dir` | Initialize Rust project with Cargo configuration |
| `/cargo-build` | Build the Rust project |
| `/cargo-test` | Run tests |
| `/cargo-clippy` | Run Clippy linter |
| `/cargo-fmt` | Format Rust source files |
| `/cargo-check` | Fast type checking without building |
| `/cargo-doc` | Generate and view documentation |

<details>
<summary>Usage Examples</summary>

```bash
# Initialize a new Rust project (interactive)
/rust:init-dir

# Build the project (debug mode)
/cargo-build

# Build for release
/cargo-build --release

# Quick syntax and type check (faster than build)
/cargo-check

# Run all tests
/cargo-test

# Run specific test with output
/cargo-test test_name -- --nocapture

# Run Clippy linter
/cargo-clippy

# Run Clippy with strict mode (treat warnings as errors)
/cargo-clippy -- -D warnings

# Format code
/cargo-fmt

# Look up crate documentation
/cargo-doc tokio
/cargo-doc serde 1.0.0
/cargo-doc axum Router
```

</details>

### Python Plugin

| Command/Skill | Description |
|---------------|-------------|
| `/python:init-dir` | Initialize Python project with uv and modern tooling |
| `/pytest` | Run tests with pytest |
| `/ruff` | Run Ruff linter and formatter |
| `/mypy` | Run type checking |
| `/uv` | Manage dependencies with uv |
| `/python-doc` | Look up Python documentation |

<details>
<summary>Usage Examples</summary>

```bash
# Initialize a new Python project (interactive)
/python:init-dir

# Run tests
/pytest

# Run tests with coverage
/pytest --cov=mypackage --cov-report=html

# Run specific test file or function
/pytest tests/test_main.py
/pytest tests/test_main.py::test_function_name

# Run Ruff linter with auto-fix
/ruff check --fix .

# Format code with Ruff
/ruff format .

# Run type checking
/mypy src/

# Run strict type checking
/mypy --strict src/

# Add dependencies
/uv add fastapi httpx
/uv add --dev pytest-cov mypy

# Remove a dependency
/uv remove httpx

# Sync dependencies from lock file
/uv sync

# Look up documentation
/python-doc asyncio
/python-doc collections Counter
/python-doc requests 3.11
```

</details>

## MCP Servers

Each plugin includes a GitHub MCP server for enhanced GitHub integration.

### Setup

Set your GitHub Personal Access Token:

```bash
export GITHUB_PERSONAL_ACCESS_TOKEN=your_token_here
```

### Capabilities

- Repository search and browsing
- Issue and PR management
- File content retrieval
- Commit history access

## LSP Configuration

Each plugin includes language-specific LSP configuration:

| Plugin | LSP Server | Features |
|--------|------------|----------|
| Go | `gopls` | staticcheck, gofumpt, inlay hints |
| Rust | `rust-analyzer` | clippy on save, proc macros, inlay hints |
| Python | `pyright` | strict type checking, inlay hints |

### LSP Installation

```bash
# Go
go install golang.org/x/tools/gopls@latest

# Rust
rustup component add rust-analyzer

# Python
uv add --dev pyright
# or: npm install -g pyright
```

## Rules

Each plugin includes `CLAUDE.md` with language-specific rules:

- **Code Style** - Language conventions, formatting, naming
- **Project Layout** - Standard directory structure
- **TDD** - Test-Driven Development workflow
- **Security** - Input validation, secrets management
- **Architecture** - SOLID principles, Clean Architecture

## Plugin Structure

```
plugins/
├── go/
│   ├── .claude-plugin/plugin.json
│   ├── .lsp.json              # gopls config
│   ├── .mcp.json              # GitHub MCP
│   ├── CLAUDE.md
│   ├── commands/golang/
│   ├── rules/
│   └── skills/
├── rust/
│   ├── .claude-plugin/plugin.json
│   ├── .lsp.json              # rust-analyzer config
│   ├── .mcp.json              # GitHub MCP
│   ├── CLAUDE.md
│   ├── commands/rust/
│   ├── rules/
│   └── skills/
└── python/
    ├── .claude-plugin/plugin.json
    ├── .lsp.json              # pyright config
    ├── .mcp.json              # GitHub MCP
    ├── CLAUDE.md
    ├── commands/python/
    ├── rules/
    └── skills/
```

## License

MIT
