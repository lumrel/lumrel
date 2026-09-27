# Lumrel

**Composable backend building blocks for Rust.**

Lumrel is an open-source ecosystem of reusable Rust crates for building backend applications without repeatedly implementing the same foundational infrastructure.

The project is designed around a small set of principles:

> Framework-agnostic cores. Explicit adapters. Strong domain boundaries. Secure defaults. Production-oriented design.

Lumrel is not a monolithic backend framework.

Applications should be able to adopt individual components without committing to the entire ecosystem.

## Status

Lumrel is currently in early active development.

The architecture is being designed for production use from the beginning, but public APIs may still change while the initial `0.x` crates are established.

Do not assume API stability until individual crates document their stability guarantees.

## Goals

Lumrel aims to provide:

- reusable backend primitives
- coherent APIs across crates
- framework-independent domain logic
- explicit infrastructure adapters
- strong type safety
- secure defaults
- small mandatory dependency graphs
- predictable error handling
- thorough automated testing
- stable serialization contracts
- incremental adoption
- clear domain ownership

## Repository

Lumrel is maintained as a Cargo virtual workspace.

The repository is organized by domain rather than as one flat list of crates.

```text
lumrel/
├── .devcontainer/
│   └── devcontainer.json
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── crates/
│   ├── auth/
│   │   ├── core/
│   │   ├── password/
│   │   ├── passkey/
│   │   ├── api-key/
│   │   └── ...
│   │
│   ├── mail/
│   │   └── ...
│   │
│   ├── sessions/
│   │   └── ...
│   │
│   └── tokens/
│       └── ...
│
├── Cargo.toml
├── Cargo.lock
├── rust-toolchain.toml
├── deny.toml
├── AGENTS.md
├── ARCHITECTURE.md
├── CONTRIBUTING.md
├── SECURITY.md
├── CODE_OF_CONDUCT.md
├── TRADEMARKS.md
├── LICENSE
└── README.md
```

Physical directory names and published crate names are intentionally separate.

For example:

```text
crates/auth/core
```

contains the package:

```text
lumrel-auth
```

## Architecture

Lumrel separates domain behavior from infrastructure.

Conceptually:

```text
Application
    │
    ├── HTTP / framework adapters
    ├── database adapters
    ├── provider adapters
    │
    ▼
Mechanism crates
    │
    ▼
Framework-agnostic core crates
```

Core crates must not depend on infrastructure adapters.

See [ARCHITECTURE.md](ARCHITECTURE.md) for the detailed architecture.

## Authentication

The first Lumrel domain under development is authentication.

The core package is:

```text
lumrel-auth
```

located at:

```text
crates/auth/core
```

`lumrel-auth` defines authentication domain concepts only.

It does not implement:

- passwords
- passkeys
- API keys
- magic links
- OAuth providers
- databases
- sessions
- HTTP frameworks

Specific mechanisms belong in separate crates such as:

```text
lumrel-auth-password
lumrel-auth-passkey
lumrel-auth-api-key
lumrel-auth-magic-link
```

## Users and Authentication Are Separate

Lumrel intentionally treats users and authentication as different domains.

A future users domain may own concepts such as:

- user lifecycle
- profiles
- display names
- preferences
- account state

Authentication does not own those concepts.

`lumrel-auth` refers to an authenticated entity through:

```rust
PrincipalId
```

This keeps authentication reusable without depending on a particular user model.

## Authentication Model

The planned `lumrel-auth` domain model includes:

```text
PrincipalId
AuthenticationMethodId
AuthenticationAssurance
AuthenticationMethods
AuthenticationAssurances
AuthenticatedPrincipal
AuthenticationFailure
AuthenticationOutcome
Authenticator
```

Authentication supports multiple mechanisms and multi-factor flows without requiring the core to know every possible mechanism.

An authenticated principal contains:

```text
principal identity
authentication methods
authentication assurances
authentication timestamp
```

## Authentication Methods

Authentication methods are extensible identifiers.

Examples include:

```text
password
passkey
api-key
magic-link
totp
recovery-code
```

Provider-specific method identifiers can be defined outside the core.

For example:

```text
oauth:github
```

does not require a change to `lumrel-auth`.

## Authentication Assurances

Assurances describe properties established by an authentication event.

Examples may include:

```text
single-factor
multi-factor
phishing-resistant
```

A principal may have multiple assurances simultaneously.

Lumrel does not define a global strength ordering between assurance values.

Applications decide which assurances satisfy their own security policies.

## Authentication Result

Successful authentication produces an immutable:

```rust
AuthenticatedPrincipal
```

Expected authentication rejection is represented separately from operational failure.

Conceptually:

```rust
Result<AuthenticationOutcome, AuthenticatorError>
```

where:

```rust
AuthenticationOutcome::Authenticated(...)
```

represents success and:

```rust
AuthenticationOutcome::Rejected(...)
```

represents valid execution with rejected credentials.

An unavailable database or external provider is an operational error, not a credential rejection.

## Step-Up Authentication

Lumrel distinguishes between:

```text
step-up authentication
```

and:

```text
reauthentication
```

A step-up adds a new authentication factor and merges new assurance properties.

A reauthentication keeps the existing authentication methods but replaces the assurance set with the guarantees established by the new authentication event.

Both produce a new immutable `AuthenticatedPrincipal`.

## Framework Agnostic

The authentication core must not depend on Axum or any other web framework.

A future Axum integration belongs in an adapter crate.

The same rule applies to:

- PostgreSQL
- SQLx
- Redis
- SMTP
- external identity providers

## Dependencies

Core Lumrel crates are designed to keep mandatory dependency graphs small.

`lumrel-auth` is intended to have:

```text
zero mandatory external dependencies
```

by default.

Optional capabilities may use Cargo features.

For example, serde support is optional.

## Serde

Where supported, serialization is enabled explicitly:

```toml
lumrel-auth = {
    version = "0.1",
    features = ["serde"],
}
```

The default feature set remains empty.

Lumrel treats published serde representations as part of the public compatibility contract.

Deserialization must preserve all domain invariants.

Invalid serialized input must not be capable of constructing invalid Lumrel values.

## Rust

Lumrel uses:

```text
Rust Edition 2024
```

The current MSRV is:

```text
Rust 1.85
```

The development and CI toolchain is pinned independently in:

```text
rust-toolchain.toml
```

## Dev Container

Lumrel provides a Dev Container for reproducible development.

Open the repository in a compatible editor and select:

```text
Dev Containers: Reopen in Container
```

The container provides the Rust toolchain and project-specific editor extensions.

This allows contributors to work on Lumrel without installing the full Rust development environment directly on their host system.

## Development

Run:

```bash
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo deny check
```

before submitting changes.

## Continuous Integration

GitHub Actions runs independent checks in parallel:

```text
Cargo check
Cargo test
Cargo fmt
Cargo clippy
Dependency audit
```

The dependency audit uses `cargo-deny`.

## Dependency Security

Lumrel uses:

```text
deny.toml
```

to enforce dependency policy.

Checks include:

- known security advisories
- yanked dependencies
- allowed licenses
- wildcard dependency declarations
- dependency sources
- duplicate dependency versions

## Security

Security is a core project priority.

Please do not publicly disclose suspected vulnerabilities through normal issues.

See [SECURITY.md](SECURITY.md).

## Contributing

Contributions are welcome.

Lumrel uses the Developer Certificate of Origin.

See [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

Coding agents and AI-assisted contributors should also read:

[AGENTS.md](AGENTS.md)

## Code of Conduct

Participation in the project is governed by:

[CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)

## License

Lumrel source code is licensed under the MIT License unless explicitly stated otherwise.

See [LICENSE](LICENSE).

## Trademark

The source code license does not grant unrestricted rights to Lumrel branding.

See [TRADEMARKS.md](TRADEMARKS.md).
