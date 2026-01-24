# Security

- Never hardcode secrets, credentials, or API keys
- Use environment variables or secure vaults for configuration
- Validate and sanitize ALL user input; use strong types
- Use parameterized queries with `sqlx` or similar to prevent SQL injection
- Escape output properly to prevent XSS in web applications
- Use `rand` crate with `OsRng` for cryptographic randomness
- Set appropriate timeouts on network operations
- Use TLS for all network communication (`rustls` or `native-tls`)
- Minimize use of `unsafe`; document safety invariants when required
- Run `cargo clippy` and `cargo audit` to catch potential issues
- Keep dependencies updated; audit with `cargo deny` or `cargo audit`
- Follow principle of least privilege for file permissions
- Log security events but never log sensitive data
- Use constant-time comparison for secrets (`subtle` crate)
- Avoid deserializing untrusted data without validation
- Use `secrecy` crate for sensitive values in memory

## Unsafe Code Guidelines

When `unsafe` is necessary:

```rust
/// Converts a byte slice to a string without UTF-8 validation.
///
/// # Safety
///
/// The caller must ensure that `bytes` contains valid UTF-8.
pub unsafe fn str_from_utf8_unchecked(bytes: &[u8]) -> &str {
    // SAFETY: Caller guarantees bytes is valid UTF-8
    std::str::from_utf8_unchecked(bytes)
}
```

- Always document `# Safety` requirements
- Add `// SAFETY:` comments explaining why invariants are upheld
- Prefer safe abstractions that encapsulate unsafe code
- Audit all `unsafe` blocks during code review

## Dependency Security

```bash
# Install security audit tools
cargo install cargo-audit cargo-deny

# Check for known vulnerabilities
cargo audit

# Check licenses and security advisories
cargo deny check
```

Example `deny.toml`:

```toml
[advisories]
vulnerability = "deny"
unmaintained = "warn"

[licenses]
allow = ["MIT", "Apache-2.0", "BSD-3-Clause"]

[bans]
multiple-versions = "warn"
```
