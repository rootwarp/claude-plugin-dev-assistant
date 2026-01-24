# Security

- Never hardcode secrets, credentials, or API keys
- Use environment variables or secure vaults for configuration
- Validate and sanitize ALL user input
- Use parameterized queries to prevent SQL injection
- Escape output to prevent XSS in web applications
- Use `crypto/rand` for cryptographic randomness, not `math/rand`
- Set appropriate timeouts on HTTP clients and servers
- Use TLS for all network communication
- Avoid `unsafe` package unless absolutely necessary
- Run `go vet` and `staticcheck` to catch potential issues
- Keep dependencies updated; audit with `go mod verify`
- Follow principle of least privilege for file permissions
- Log security events but never log sensitive data
