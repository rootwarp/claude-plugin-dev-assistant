# claude-code-plugins

A collection of Claude Code plugins for Go, Rust, Python development and multi-agent team orchestration.

## Installation

### From GitHub

```bash
# Install specific plugin
/plugin install rootwarp/claude-code-plugins/plugins/go
/plugin install rootwarp/claude-code-plugins/plugins/rust
/plugin install rootwarp/claude-code-plugins/plugins/python
/plugin install rootwarp/claude-code-plugins/plugins/dev-team
```

### Local Development

```bash
claude --plugin-dir ./plugins/go
claude --plugin-dir ./plugins/rust
claude --plugin-dir ./plugins/python
claude --plugin-dir ./plugins/dev-team
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

### Dev-Team Plugin

Multi-agent development team orchestration with planning and execution pipelines.

| Skill | Description |
|-------|-------------|
| `/dev-plan` | Full planning pipeline: PRD, research, architecture, project plan, and issue estimation |
| `/dev-run` | Execution pipeline: implement issues via code-writer with code-reviewer quality gates |

#### Agents

| Agent | Role |
|-------|------|
| `prd-writer` | Gather requirements and write Product Requirements Documents |
| `researcher` | Investigate technologies, landscape, feasibility, and best practices |
| `software-architect` | Design modular system architecture with clear boundaries and interfaces |
| `project-planner` | Plan phases, milestones, and dependencies |
| `issue-estimator` | Break down plans into parallel, sprint-ready issues |
| `code-writer` | Implement issues via TDD in isolated git worktrees |
| `code-reviewer` | Review lead: code quality + orchestrates security and bug review + merges |
| `security-auditor` | Scan for vulnerabilities and OWASP issues |
| `bug-hunter` | Find potential bugs, edge cases, and race conditions |

<details>
<summary>Usage Examples</summary>

```bash
# Plan a new project from an idea
/dev-plan Build a REST API for task management with user auth

# Implement all issues from a phase
/dev-run plan/issues/01-phase-1-foundation.md

# Implement all planned issues
/dev-run all
```

#### `/dev-plan` Pipeline

1. **Kickoff** - Clarify requirements and constraints
2. **PRD** - `prd-writer` drafts the Product Requirements Document
3. **Research** - `researcher` investigates technologies and feasibility
4. **Architecture** - `software-architect` designs the system
5. **Project Plan** - `project-planner` defines phases and milestones
6. **Issue Estimation** - `issue-estimator` breaks down into sprint-ready issues

Each stage requires user approval before proceeding.

#### `/dev-run` Pipeline

1. **Load Issues** - Parse phase files and build execution queues
2. **Write** - `code-writer` implements each issue in an isolated worktree
3. **Pre-Review Gate** - Coverage >= 70%, tests passing, lint clean
4. **Review** - `code-reviewer` runs quality, security, and bug review
5. **Fix Cycle** - Up to 3 iterations to resolve findings
6. **Merge** - Fast-forward merge into `develop` after approval

</details>

## Rules

Each language plugin includes `CLAUDE.md` with language-specific rules:

- **Code Style** - Language conventions, formatting, naming
- **Concurrency** - Concurrent programming patterns and safety
- **Project Layout** - Standard directory structure
- **TDD** - Test-Driven Development workflow
- **Security** - Input validation, secrets management
- **Architecture** - SOLID principles, Clean Architecture

## Plugin Structure

```
plugins/
├── dev-team/
│   ├── .claude-plugin/plugin.json
│   ├── agents/
│   │   ├── bug-hunter.md
│   │   ├── code-reviewer.md
│   │   ├── code-writer.md
│   │   ├── issue-estimator.md
│   │   ├── prd-writer.md
│   │   ├── project-planner.md
│   │   ├── researcher.md
│   │   ├── security-auditor.md
│   │   └── software-architect.md
│   └── skills/
│       ├── dev-plan/
│       └── dev-run/
├── go/
│   ├── .claude-plugin/plugin.json
│   ├── CLAUDE.md
│   ├── commands/golang/
│   ├── rules/
│   └── skills/
├── rust/
│   ├── .claude-plugin/plugin.json
│   ├── CLAUDE.md
│   ├── commands/rust/
│   ├── rules/
│   └── skills/
└── python/
    ├── .claude-plugin/plugin.json
    ├── CLAUDE.md
    ├── commands/python/
    ├── rules/
    └── skills/
```

## License

MIT
