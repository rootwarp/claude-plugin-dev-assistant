# /go-doc

Look up Go standard library documentation from pkg.go.dev.

## Arguments

- `package`: The Go package path to look up (e.g., `fmt`, `net/http`, `encoding/json`)
- `version` (optional): Specific Go version (e.g., `go1.23`). Defaults to latest.
- `symbol` (optional): Specific function, type, or method to focus on

## Instructions

When the user wants to verify or look up Go standard library documentation:

1. **Construct the URL** based on the package path:
   - Base URL: `https://pkg.go.dev`
   - Standard library: `https://pkg.go.dev/{package}`
   - With version: `https://pkg.go.dev/{package}@{version}`

2. **Fetch the documentation** using WebFetch with a prompt tailored to the user's needs.

3. **Present the information** including:
   - Package overview and purpose
   - Relevant functions, types, or interfaces
   - Usage examples when available
   - Format verbs or special syntax (if applicable)

## URL Structure Reference

| Package Type | URL Format | Example |
|--------------|------------|---------|
| Top-level | `https://pkg.go.dev/{pkg}` | `https://pkg.go.dev/fmt` |
| Nested | `https://pkg.go.dev/{pkg}/{subpkg}` | `https://pkg.go.dev/net/http` |
| With version | `https://pkg.go.dev/{pkg}@go1.23` | `https://pkg.go.dev/fmt@go1.23` |
| Third-party | `https://pkg.go.dev/{full-module-path}` | `https://pkg.go.dev/github.com/gin-gonic/gin` |

## Documentation Sections

pkg.go.dev documentation pages typically contain:

- **Overview**: Package purpose and general usage patterns
- **Index**: Quick navigation to functions, types, and examples
- **Examples**: Runnable code samples with expected output
- **Functions**: Function signatures with descriptions
- **Types**: Struct and interface definitions with methods

## Example Usage

### Look up fmt package
```
/go-doc fmt
```
Fetches: `https://pkg.go.dev/fmt`

### Look up net/http with specific version
```
/go-doc net/http go1.22
```
Fetches: `https://pkg.go.dev/net/http@go1.22`

### Look up specific function
```
/go-doc encoding/json Marshal
```
Fetches documentation and focuses on the `Marshal` function.

## Execution

When invoked, use WebFetch to retrieve the documentation:

```
WebFetch url="https://pkg.go.dev/{package}" prompt="Extract the documentation for this Go package. Include: 1) Package overview 2) Key functions and their signatures 3) Important types and interfaces 4) Usage examples. If a specific symbol was requested, focus on that symbol's documentation."
```

If the user asks about a specific symbol (function, type, method), include it in the prompt:

```
WebFetch url="https://pkg.go.dev/{package}" prompt="Find the documentation for the '{symbol}' function/type in this package. Include its signature, description, parameters, return values, and any available examples."
```

## Error Handling

If the package is not found or the URL returns an error:
1. Verify the package path is correct (check for typos)
2. For standard library packages, try without a version suffix
3. Suggest related packages if the exact match isn't found
4. Check if it might be a third-party package requiring full module path
