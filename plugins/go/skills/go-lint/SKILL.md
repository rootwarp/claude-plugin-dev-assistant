# /go-lint

Run linters on the Go project.

## Instructions

Run golangci-lint if available, otherwise use go vet:

```bash
if command -v golangci-lint &> /dev/null; then
    golangci-lint run ./...
else
    go vet ./...
fi
```

Analyze any lint warnings or errors and suggest fixes.
