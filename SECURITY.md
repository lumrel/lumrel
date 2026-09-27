# Lumrel Security Policy

Security is a core design priority for Lumrel.

Lumrel may provide foundational components for authentication, sessions, credentials, tokens, databases, mail, and other backend infrastructure.

Security reports are therefore taken seriously.

## Reporting a Vulnerability

Do not open a normal public GitHub issue for a suspected vulnerability.

Use GitHub Private Vulnerability Reporting when it is available for the Lumrel repository.

If private vulnerability reporting is temporarily unavailable, contact the project maintainers through an appropriate private channel rather than publishing technical exploit details publicly.

## Include

A useful security report should include, when possible:

- affected crate
- affected version or commit
- vulnerability description
- expected behavior
- observed behavior
- reproduction steps
- security impact
- required environment
- relevant logs with secrets removed
- minimal proof of concept when appropriate
- suggested mitigation when known

Do not include unrelated secrets, credentials, or personal information.

## Responsible Disclosure

Please allow maintainers a reasonable opportunity to investigate and release a fix before publishing exploitation details.

When appropriate, maintainers may coordinate disclosure with the reporter.

A published advisory may contain:

- affected versions
- fixed versions
- upgrade guidance
- mitigations
- technical explanation
- credits when desired by the reporter

## Supported Versions

Lumrel is currently in early development.

Until stable release support policies are established, security fixes may target the most recent maintained release or development branch.

A formal support matrix will be published once stable release lines exist.

## Security Scope

Examples of security issues include:

- authentication bypass
- authorization bypass
- privilege escalation
- insecure credential handling
- secret leakage
- password handling flaws
- session vulnerabilities
- token validation flaws
- rate-limit bypass
- unsafe deserialization
- cross-tenant data exposure
- dangerous defaults
- cryptographic misuse
- dependency vulnerabilities
- unexpected unsafe code
- information disclosure through errors or logs

Normal correctness bugs without security impact should use the normal issue tracker.

## Authentication Security

Authentication code should distinguish:

```text
expected credential rejection
```

from:

```text
operational system failure
```

Authentication adapters should avoid exposing internal rejection distinctions to untrusted clients when doing so could facilitate account enumeration or related attacks.

## Credentials and Secrets

Secrets must not intentionally appear in:

- logs
- panic messages
- `Debug` output
- error strings
- snapshots
- telemetry
- serialized public structures
- test fixtures committed to the repository

Credential types should avoid implementing `Debug`, `Clone`, or serialization unless the mechanism specifically requires and safely supports it.

## Cryptography

Lumrel must not implement custom cryptographic primitives or protocols.

Cryptographic functionality should use established, reviewed libraries and recognized algorithms appropriate for the specific mechanism.

## Unsafe Rust

Core authentication crates forbid unsafe Rust.

If another future crate has a legitimate need for `unsafe`, the unsafe code must:

- remain narrowly scoped
- document its safety invariants
- receive dedicated review
- include appropriate tests

## Dependencies

Dependency security is part of the Lumrel threat surface.

The repository uses:

```text
cargo-deny
```

to check:

- security advisories
- yanked dependencies
- licenses
- dependency sources
- wildcard declarations
- duplicate versions

Dependency audit failures should be investigated rather than routinely ignored.

## Serialization

Deserialization must not bypass domain invariants.

Invalid external data must not be capable of constructing invalid core domain values.

Serde compatibility is treated as part of the public API where the feature is enabled.

## Framework and Adapter Security

Core crates cannot guarantee secure deployment by themselves.

Applications and adapters remain responsible for areas such as:

- TLS
- HTTP security
- cookies
- CSRF protections
- CORS
- infrastructure access
- secret storage
- database security
- provider configuration
- deployment hardening
- monitoring
- incident response

Documentation should make important integration requirements explicit.

## Security Is Shared

Lumrel aims to provide secure primitives and safe defaults.

Using Lumrel does not automatically make an application secure.

Application developers remain responsible for correct integration and deployment.
