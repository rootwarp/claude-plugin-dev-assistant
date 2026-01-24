# /python-doc

Look up Python documentation from docs.python.org or PyPI.

## Arguments

- `module`: The Python module or package to look up (e.g., `asyncio`, `collections`, `requests`)
- `version` (optional): Python version (e.g., `3.11`). Defaults to `3`.
- `symbol` (optional): Specific function, class, or method to focus on

## Instructions

When the user wants to verify or look up Python documentation:

### Standard Library

1. **Construct the URL** based on the module:
   - Base URL: `https://docs.python.org/3/library/`
   - With version: `https://docs.python.org/{version}/library/`
   - Example: `https://docs.python.org/3/library/asyncio.html`

2. **Fetch the documentation** using WebFetch with a prompt tailored to the user's needs.

### Third-Party Packages (PyPI)

1. **Construct the URL**:
   - PyPI: `https://pypi.org/project/{package}/`
   - Read the Docs: `https://{package}.readthedocs.io/` (if available)

2. **Fetch the documentation** using WebFetch.

## URL Structure Reference

| Type | URL Format | Example |
|------|------------|---------|
| Stdlib | `https://docs.python.org/3/library/{module}.html` | `https://docs.python.org/3/library/asyncio.html` |
| Stdlib with version | `https://docs.python.org/{version}/library/{module}.html` | `https://docs.python.org/3.11/library/typing.html` |
| PyPI | `https://pypi.org/project/{package}/` | `https://pypi.org/project/requests/` |
| Read the Docs | `https://{package}.readthedocs.io/` | `https://requests.readthedocs.io/` |

## Example Usage

### Look up asyncio module
```
/python-doc asyncio
```
Fetches: `https://docs.python.org/3/library/asyncio.html`

### Look up typing with specific version
```
/python-doc typing 3.11
```
Fetches: `https://docs.python.org/3.11/library/typing.html`

### Look up third-party package
```
/python-doc requests
```
Fetches: `https://pypi.org/project/requests/` and linked documentation

## Execution

When invoked, use WebFetch to retrieve the documentation:

```
WebFetch url="https://docs.python.org/3/library/{module}.html" prompt="Extract the documentation for this Python module. Include: 1) Module overview 2) Key classes and functions 3) Common usage patterns 4) Code examples. If a specific symbol was requested, focus on that item's documentation."
```

For third-party packages:

```
WebFetch url="https://pypi.org/project/{package}/" prompt="Extract information about this Python package. Include: 1) Package description 2) Installation instructions 3) Link to documentation 4) Basic usage examples if available."
```

## Error Handling

If the module is not found:
1. Verify the module name is correct
2. Check if it might be a submodule (e.g., `collections.abc`)
3. For third-party packages, search PyPI
4. Suggest related modules if exact match isn't found
