# claude-plugin-dev-assistant

A Claude Code plugin providing Go development tools.

## Installation

### From GitHub

```bash
/plugin install rootwarp/claude-plugin-dev-assistant
```

### Local Development

```bash
claude --plugin-dir ./path/to/claude-plugin-dev-assistant
```

## Available Commands

| Command | Description |
|---------|-------------|
| `/golang:init-dir` | Initialize Go project with directories, Makefile, and .gitignore |

## Available Skills

| Skill | Description |
|-------|-------------|
| `/go-build` | Build the Go project |
| `/go-test` | Run tests with coverage |
| `/go-lint` | Run linters (golangci-lint or go vet) |
| `/go-mod` | Tidy Go modules |
| `/go-fmt` | Format Go source files |

## Usage

After installation, use the commands and skills in Claude Code:

```
/golang:init-dir
/go-build
/go-test
/go-lint
/go-mod
/go-fmt
```

## MCP Servers

This plugin includes a GitHub MCP server for enhanced GitHub integration.

### Setup

Set your GitHub Personal Access Token:

```bash
export GITHUB_PERSONAL_ACCESS_TOKEN=your_token_here
```

Or add it to your `.mcp.json` (not recommended for shared repos).

### Capabilities

The GitHub MCP server provides:
- Repository search and browsing
- Issue and PR management
- File content retrieval
- Commit history access

## LSP Configuration

This plugin includes `gopls` (Go language server) configuration with:

- **staticcheck** - Advanced static analysis
- **gofumpt** - Stricter formatting than gofmt
- **Analyses** - unusedparams, shadow, nilness, unusedwrite
- **Hints** - Variable types, parameter names, constant values

Requires `gopls` installed:
```bash
go install golang.org/x/tools/gopls@latest
```

## Rules

This plugin includes `CLAUDE.md` with rules for:

- **Code Style** - Effective Go guidelines, formatting, naming conventions
- **Project Layout** - Standard directory structure based on golang-standards/project-layout
- **TDD** - Test-Driven Development workflow (Red-Green-Refactor)
- **Security** - Input validation, secrets management, secure coding practices

## Plugin Structure

```
.
├── .claude-plugin/
│   └── plugin.json      # Plugin manifest
├── .lsp.json            # LSP server config (gopls)
├── .mcp.json            # MCP server config
├── commands/            # Custom commands
│   └── golang/
│       └── init-dir.md
├── rules/               # Modular rules
│   ├── code-style.md
│   ├── project-layout.md
│   ├── tdd.md
│   └── security.md
├── skills/
│   ├── go-build/
│   │   └── SKILL.md
│   ├── go-fmt/
│   │   └── SKILL.md
│   ├── go-lint/
│   │   └── SKILL.md
│   ├── go-mod/
│   │   └── SKILL.md
│   └── go-test/
│       └── SKILL.md
├── CHANGELOG.md
├── CLAUDE.md            # Imports rules/*
├── LICENSE
└── README.md
```

## License

MIT
