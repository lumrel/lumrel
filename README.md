# Lumrel

**Composable backend building blocks for Rust.**

Lumrel is an open-source collection of reusable Rust crates for building backend applications without repeatedly implementing the same foundational infrastructure.

The project provides independent but coherent modules for common backend concerns such as authentication, users, permissions, sessions, JWTs, email, rate limiting, configuration, logging, and database integration.

Lumrel is designed around a simple idea:

> Backend infrastructure should be reusable, composable, framework-agnostic, and secure by default.

## Status

Lumrel is currently under active development.

The project is being designed with production use in mind, but APIs may change while the initial architecture stabilizes.

Do not assume semantic-versioning compatibility guarantees before individual crates reach stable releases.

## Goals

Lumrel aims to provide:

- A coherent architecture across all crates.
- Framework-agnostic core functionality.
- Explicit adapters for frameworks, databases, and external services.
- Strong security defaults.
- Flexible abstractions without unnecessary coupling.
- Predictable and consistent APIs.
- Clear error handling.
- Thorough automated testing.
- Production-oriented design.
- Independent crates that can be adopted incrementally.

Lumrel does **not** aim to become a monolithic backend framework.

Applications should be able to use only the components they need.

## Architecture

Lumrel follows a layered architecture.

Core crates should remain independent from specific web frameworks, database implementations, email providers, or infrastructure vendors whenever practical.

Framework-specific and provider-specific behavior belongs in adapters.

Conceptually:

```text
Application
    │
    ├── Framework adapters
    │
    ├── Provider adapters
    │
    ▼
Lumrel crates
    │
    ▼
Framework-agnostic domain abstractions
```

This separation allows applications to replace infrastructure without rewriting their core business logic.

## Planned Crates

The exact crate structure may evolve, but the initial ecosystem is expected to include modules such as:

```text
lumrel-auth
lumrel-users
lumrel-permissions
lumrel-session
lumrel-jwt
lumrel-mail
lumrel-rate-limit
lumrel-config
lumrel-logging
lumrel-db
```

Additional adapter crates may provide integrations such as:

```text
lumrel-axum
lumrel-sqlx
lumrel-postgres
```

Names and boundaries may change as the architecture develops.

## Design Principles

### Framework agnostic

Core functionality should not depend on Axum, Actix Web, Rocket, or any other HTTP framework unless the dependency belongs to an explicitly framework-specific adapter.

### Composable

Each crate should be useful independently.

Using `lumrel-mail` should not require adopting Lumrel authentication, database abstractions, or unrelated components.

### Coherent

Independent does not mean inconsistent.

Lumrel crates should share common design conventions for:

- errors
- traits
- configuration
- naming
- testing
- feature flags
- async behavior
- observability

### Flexible

Lumrel provides abstractions and sensible defaults while allowing applications to replace implementations where appropriate.

The project should avoid unnecessary assumptions about application architecture.

### Security conscious

Security-sensitive components must favor safe behavior over convenience.

Security-related changes should receive additional scrutiny, tests, and documentation.

### Explicit

Important behavior should be visible in APIs and configuration.

Lumrel should avoid surprising implicit behavior, hidden global state, and security-sensitive magic.

## Example

A future application might combine several Lumrel modules while choosing its own HTTP framework and infrastructure:

```rust
use lumrel_auth::AuthService;
use lumrel_mail::Mailer;
use lumrel_permissions::PermissionService;

async fn example() {
    // Application-specific adapters and configuration
    // can be provided independently.
}
```

The public APIs shown in documentation during early development may change.

## Workspace

Lumrel is maintained as a Cargo workspace containing multiple crates.

A typical repository layout is:

```text
lumrel/
├── crates/
│   ├── lumrel-auth/
│   ├── lumrel-users/
│   ├── lumrel-permissions/
│   ├── lumrel-session/
│   ├── lumrel-jwt/
│   ├── lumrel-mail/
│   ├── lumrel-rate-limit/
│   ├── lumrel-config/
│   ├── lumrel-logging/
│   └── lumrel-db/
├── examples/
├── docs/
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── TRADEMARKS.md
├── LICENSE
└── README.md
```

## Development

The minimum supported Rust version will be documented once the initial compatibility policy is established.

Typical development commands will include:

```bash
cargo build --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

Contributors should run the relevant checks before submitting changes.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the complete contribution guidelines.

## Security

Please do **not** publicly disclose suspected vulnerabilities through normal GitHub issues.

Follow the private reporting process described in [SECURITY.md](SECURITY.md).

## Contributing

Contributions are welcome.

Lumrel uses a lightweight contribution process based on the Developer Certificate of Origin (DCO).

By contributing, you certify that you have the right to submit your contribution under the project's license.

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Code of Conduct

Participation in the Lumrel community is governed by [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## License

Lumrel source code is licensed under the **MIT License**, unless a specific file or directory explicitly states otherwise.

See [LICENSE](LICENSE).

## Trademark

The MIT License applies to the source code.

It does **not** grant permission to use the Lumrel name, logo, or other project branding in a way that suggests an unofficial project, fork, product, or service is officially associated with Lumrel.

Refer to [TRADEMARKS.md](TRADEMARKS.md) for details.
