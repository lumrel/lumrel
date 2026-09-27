# Contributing to Lumrel

Thank you for your interest in contributing to Lumrel.

Lumrel is designed as a production-oriented ecosystem of reusable Rust backend components.

Correctness, security, maintainability, explicit architecture, and long-term compatibility are prioritized over implementation speed.

## Read the Architecture First

Before making architectural or public API changes, read:

- `README.md`
- `ARCHITECTURE.md`
- `AGENTS.md`

`AGENTS.md` contains detailed implementation rules that apply equally to human and AI-assisted contributions.

## Development Environment

Lumrel provides a Dev Container.

Using it is recommended because it provides the same Rust development environment expected by the project.

Open the repository and select:

```text
Dev Containers: Reopen in Container
```

The development toolchain is pinned through:

```text
rust-toolchain.toml
```

## Workspace Structure

Crates are grouped by domain:

```text
crates/<domain>/<component>
```

For example:

```text
crates/auth/core
```

is published as:

```text
lumrel-auth
```

Do not create unrelated crates at the repository root.

## Before Implementing a Large Change

For:

- new domains
- new authentication mechanisms
- public API redesigns
- new adapters
- architectural changes
- new shared abstractions

open a discussion or issue before investing in a large implementation.

Small fixes, tests, and documentation improvements may generally go directly through a pull request.

## Architectural Principles

Contributions should preserve:

- framework-agnostic core crates
- explicit infrastructure adapters
- clear domain ownership
- small crate responsibilities
- secure defaults
- type-safe APIs
- private invariants
- minimal mandatory dependencies
- predictable failure behavior

## Core vs Infrastructure

A core crate should not depend directly on:

- Axum
- Actix Web
- SQLx
- Diesel
- PostgreSQL
- Redis
- cloud provider SDKs
- mail providers

Such integrations belong in adapter crates.

## Avoid Premature Abstractions

Do not introduce generic repositories, utility crates, or common traits merely because two implementations look superficially similar.

Prefer abstractions extracted from demonstrated repetition.

In particular, authentication mechanisms should normally define their own credential storage contracts.

## Public APIs

Public APIs should:

- remain as small as practical
- avoid exposing internal representation
- use validated constructors
- keep struct fields private
- document invariants
- avoid unnecessary generic complexity
- remain extensible without forcing unrelated consumers to change

Public enums that are expected to grow should normally be `#[non_exhaustive]`.

## Core Value Types

Invalid states should be difficult or impossible to construct.

Do not add alternate constructors that bypass validation.

Conversion implementations and serde deserialization must reuse the same invariants.

## Serde Compatibility

Serde support is optional.

When a serialized representation has been published, treat it as part of the public API.

Do not casually change:

- field names
- enum names
- tagging schemes
- timestamp representation
- newtype representation

Incompatible changes require deliberate versioning decisions.

## Dependencies

New dependencies require justification.

Before adding one, consider:

- maintenance
- security history
- license
- transitive dependencies
- MSRV impact
- compile-time impact
- whether the dependency belongs in an adapter
- whether the functionality can remain dependency-free

Core crates should remain particularly conservative.

## Security-Sensitive Code

Authentication, authorization, tokens, sessions, secrets, credentials, and cryptographic code require additional scrutiny.

Do not:

- invent cryptographic algorithms
- log secrets
- expose credentials through `Debug`
- serialize secrets unnecessarily
- distinguish sensitive external authentication errors in ways that enable account enumeration

## Unsafe Rust

Core authentication code uses:

```rust
#![forbid(unsafe_code)]
```

Do not introduce unsafe Rust into a core crate.

## Formatting

Run:

```bash
cargo fmt --all -- --check
```

To automatically format:

```bash
cargo fmt --all
```

## Compilation

Run:

```bash
cargo check --workspace
```

## Tests

Run:

```bash
cargo test --workspace
```

New functionality should include tests.

Tests should cover invariants and failure paths, not only happy paths.

Bug fixes should include regression coverage whenever practical.

## Clippy

Run:

```bash
cargo clippy --workspace --all-targets -- -D warnings
```

Do not suppress warnings without a documented reason.

## Dependency Audit

Run:

```bash
cargo deny check
```

Do not weaken `deny.toml` merely to allow a dependency without justification.

## Required Local Checks

Before opening a pull request, the following should all pass:

```bash
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo deny check
```

## Continuous Integration

GitHub Actions runs these categories independently and in parallel:

```text
Cargo check
Cargo test
Cargo fmt
Cargo clippy
Dependency audit
```

Pull requests should remain green before merge.

## Documentation

Public APIs should include useful Rustdoc.

Documentation should describe:

- semantics
- invariants
- errors
- security implications
- examples where useful

Update Markdown documentation when changing architecture or compatibility guarantees.

## Commits

Use clear conventional commit messages when practical.

Examples:

```text
feat(auth): add principal identifier
feat(auth): add authentication outcomes
fix(auth): reject duplicate methods
test(auth): cover invalid serde input
docs: document authentication invariants
ci: split Rust checks into parallel jobs
```

Keep commits focused.

## Developer Certificate of Origin

Lumrel uses the Developer Certificate of Origin 1.1.

Commits must include a sign-off:

```text
Signed-off-by: Your Name <your-email@example.com>
```

Git can add this automatically:

```bash
git commit -s
```

By contributing, you certify that you have the right to submit the contribution under the project's license.

## Pull Requests

A pull request should explain:

- the problem
- the approach
- important design decisions
- compatibility implications
- security implications when relevant
- tests added or changed

Avoid unrelated refactors in the same pull request.

## AI-Assisted Contributions

AI-assisted development is allowed.

The contributor remains responsible for all submitted code.

Before submitting AI-assisted work, verify:

- correctness
- architecture compliance
- licensing
- security
- tests
- dependency choices
- documentation
- API existence

Do not submit generated code you do not understand.

Coding agents should follow `AGENTS.md`.

## Security Reports

Do not disclose security vulnerabilities through normal public issues.

See `SECURITY.md`.

## Conduct

All contributors must follow `CODE_OF_CONDUCT.md`.

## License

Unless explicitly stated otherwise, contributions are accepted under the MIT License.
