# /go-fmt

Format Go source files.

## Instructions

Format all Go files in the project:

```bash
gofmt -w .
```

Or use goimports if available for better import management:

```bash
if command -v goimports &> /dev/null; then
    goimports -w .
else
    gofmt -w .
fi
```
