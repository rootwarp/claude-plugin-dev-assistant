# Project Layout

Follow [golang-standards/project-layout](https://github.com/golang-standards/project-layout) conventions.

## Core Directories

| Directory | Purpose |
|-----------|---------|
| `cmd/` | Main applications; one subdirectory per executable |
| `internal/` | Private code; cannot be imported by external projects |
| `pkg/` | Public library code safe for external import |

## Service Directories

| Directory | Purpose |
|-----------|---------|
| `api/` | OpenAPI specs, protobuf definitions, JSON schemas |
| `web/` | Static assets, templates, SPAs |

## Support Directories

| Directory | Purpose |
|-----------|---------|
| `configs/` | Configuration templates and defaults |
| `scripts/` | Build, install, analysis scripts |
| `build/` | Packaging (`build/package/`) and CI (`build/ci/`) |
| `deployments/` | Docker, Kubernetes, Terraform configs |
| `test/` | External test apps and test data |
| `docs/` | Design and user documentation |
| `tools/` | Supporting tools for the project |
| `examples/` | Example usage code |

## Rules

- Never create `/src` directory - not idiomatic for Go
- Keep `cmd/` minimal; import logic from `internal/` or `pkg/`
- Use `internal/` by default; only use `pkg/` when external import is intended
- Small projects don't need all directories; start minimal and grow as needed
- Each `cmd/<app>/` should have its own `main.go`

## Recommended Starting Structure

For most projects, start with:

```
project/
├── cmd/
│   └── app/
│       └── main.go
├── internal/
│   └── app/
├── go.mod
└── README.md
```

Add directories as needed, not preemptively.
