# Lumrel Agent Guidelines

This document defines the architectural, implementation, testing, security, and compatibility rules that coding agents must follow when modifying Lumrel.

Read this file before making changes.

If implementation details conflict with this document, treat this document as the intended architecture unless a maintainer explicitly changes the decision.

## Project

Lumrel is an open-source Rust ecosystem providing reusable backend building blocks.

The project is designed primarily for production backend applications and emphasizes:

- framework-agnostic core crates
- explicit adapters
- strong domain boundaries
- secure defaults
- robust APIs
- composability
- low mandatory dependency cost
- predictable behavior
- long-term API stability

Lumrel is not intended to become a monolithic web framework.

## Repository Model

Lumrel is a Cargo virtual workspace.

The repository is organized by domain:

```text
crates/
├── auth/
│   ├── core/
│   ├── password/
│   ├── passkey/
│   ├── api-key/
│   ├── magic-link/
│   └── ...
│
├── mail/
│   └── ...
│
├── sessions/
│   └── ...
│
└── tokens/
    └── ...
```

Physical directory names do not have to match published crate names.

For example:

```text
crates/auth/core
```

publishes as:

```text
lumrel-auth
```

and:

```text
crates/auth/password
```

would publish as:

```text
lumrel-auth-password
```

The root workspace currently uses:

```toml
[workspace]
resolver = "3"
members = [
    "crates/*/*",
]
```

Examples may later be added as workspace members once real examples exist.

## Rust Versions

The workspace uses Rust Edition 2024.

The minimum supported Rust version is:

```text
1.85
```

The development and CI toolchain is pinned separately through:

```text
rust-toolchain.toml
```

The currently selected development toolchain is:

```text
1.98.1
```

Do not confuse the development toolchain with the MSRV.

Code must not require a Rust version newer than the declared MSRV unless the MSRV is intentionally changed.

## Development Environment

The repository contains a Dev Container configuration.

Development should work inside the provided container without requiring additional Rust toolchains on the host system.

The development environment includes:

- Rust
- rust-analyzer
- rustfmt
- Clippy
- rust-src
- LLDB integration
- TOML support
- YAML support

Do not introduce unnecessary development dependencies or services into the Dev Container.

PostgreSQL, Redis, mail servers, or similar infrastructure should only be introduced when an actual crate requires them.

## Core Architectural Rule

Core crates must remain independent from infrastructure whenever practical.

A core crate must not know about:

- Axum
- Actix Web
- Rocket
- SQLx
- Diesel
- PostgreSQL
- Redis
- SMTP providers
- cloud vendors
- HTTP transport details

Infrastructure belongs in adapters.

Dependency direction must point toward domain abstractions.

Never make a core crate depend on one of its adapters.

## Domain Boundaries

Different backend concerns must remain separate domains.

For example:

```text
lumrel-users
```

owns user concepts.

```text
lumrel-auth
```

owns authentication concepts.

Authentication must not absorb:

- profile data
- display names
- avatars
- preferences
- user suspension policy
- user lifecycle management

Authentication refers to the authenticated entity through its own independent identifier:

```rust
PrincipalId
```

This keeps authentication independent from the users domain.

## Authentication Architecture

`lumrel-auth` is the framework-agnostic authentication core.

It must remain small and stable.

It does not implement specific authentication mechanisms.

Mechanism-specific behavior belongs in separate crates such as:

```text
lumrel-auth-password
lumrel-auth-passkey
lumrel-auth-api-key
lumrel-auth-magic-link
```

Provider-specific implementations must also remain outside the core.

The core should not know about:

```text
oauth:github
oauth:google
oidc:auth0
```

Provider integrations define their own method identifiers.

## Authentication Storage

Do not create a universal credential repository abstraction.

Password credentials, passkeys, API keys, and other mechanisms have fundamentally different storage requirements.

Each authentication mechanism should define its own storage interfaces where necessary.

Avoid abstractions such as:

```rust
trait CredentialRepository
```

unless a real cross-mechanism requirement is demonstrated.

## Core Authentication Types

The planned public domain model contains:

```text
PrincipalId
PrincipalIdError

AuthenticationMethodId
AuthenticationMethodIdError

AuthenticationAssurance
AuthenticationAssuranceError

AuthenticationMethods
AuthenticationMethodsError

AuthenticationAssurances
AuthenticationAssurancesError

AuthenticatedPrincipal
StepUpError

AuthenticationFailure
AuthenticationOutcome

Authenticator
```

Do not introduce overlapping types without architectural justification.

## PrincipalId

`PrincipalId` is an opaque newtype over `String`.

It must not impose UUID, ULID, database ID, or other identifier formats.

Rules:

- maximum 256 Unicode characters
- Unicode is allowed
- whitespace is forbidden
- control characters are forbidden
- Unicode normalization is not performed
- equality is exact
- input is never silently transformed

Length is measured using logical Unicode characters, not UTF-8 bytes.

Validation errors must not contain the invalid input.

They may contain safe structural information such as:

```text
max
actual
character index
```

Indices refer to logical character positions.

## AuthenticationMethodId

`AuthenticationMethodId` is an opaque newtype over `String`.

Rules:

- maximum 64 characters
- ASCII only
- permitted characters:

```text
A-Z a-z 0-9 . _ : -
```

- first character must be alphanumeric
- last character must be alphanumeric
- separators may repeat
- identifiers are case-sensitive
- official Lumrel identifiers use lowercase

Examples:

```text
password
passkey
api-key
magic-link
totp
recovery-code
oauth:github
company.sso
```

The core provides canonical constructors only for generic Lumrel concepts.

Provider-specific values must live outside the core.

## AuthenticationAssurance

`AuthenticationAssurance` follows the same identifier rules as `AuthenticationMethodId`.

Maximum length:

```text
64
```

Generic Lumrel values may include concepts such as:

```text
single-factor
multi-factor
phishing-resistant
```

Assurances are extensible identifiers.

Do not implement a global ordering between assurances.

In particular, do not implement:

```text
PartialOrd
Ord
```

A consumer's security policy determines which assurances satisfy a requirement.

## AuthenticationMethods

Authentication methods form a non-empty ordered set with explicit primary semantics.

Conceptually:

```rust
pub struct AuthenticationMethods {
    primary: AuthenticationMethodId,
    additional: Vec<AuthenticationMethodId>,
}
```

Rules:

- exactly one primary method
- zero or more additional methods
- methods must be unique
- primary must not appear in additional
- order of additional methods is preserved
- maximum 16 total methods
- there is no `Default`
- invalid states must not be constructible through public APIs

Iteration order is:

```text
primary
additional[0]
additional[1]
...
```

Expected API includes:

```text
new
single
primary
additional
len
is_single_factor
contains
iter
push
with_method
IntoIterator
```

## AuthenticationAssurances

Authentication assurances are an ordered unique collection.

Unlike methods, the collection may be empty.

Rules:

- zero or more assurances
- duplicates forbidden
- insertion order preserved
- maximum 16 assurances

An empty collection means that the authenticator declares no specific assurance properties.

Expected API includes:

```text
new
empty
default
contains
len
is_empty
iter
push
with_assurance
IntoIterator
```

## AuthenticatedPrincipal

`AuthenticatedPrincipal` is an immutable value representing a successful authentication.

Conceptually:

```rust
pub struct AuthenticatedPrincipal {
    principal_id: PrincipalId,
    methods: AuthenticationMethods,
    assurances: AuthenticationAssurances,
    authenticated_at: SystemTime,
}
```

All fields remain private.

Construction occurs through controlled constructors.

The type exposes read-only accessors.

Expected helpers include:

```text
principal_id
methods
assurances
authenticated_at
uses_method
has_assurance
is_multi_factor
```

The type should implement:

```text
Clone
Debug
PartialEq
Eq
```

It should not implement `Hash`.

## authenticated_at Semantics

`authenticated_at` represents the most recent effective authentication event.

A step-up authentication updates the timestamp.

A reauthentication updates the timestamp.

The core does not validate whether the timestamp is:

- in the future
- too old
- sufficiently recent

Those rules belong to policy layers.

## Step-Up Authentication

`AuthenticatedPrincipal` is immutable.

Adding a new authentication factor creates a new value rather than mutating the existing principal.

A step-up:

- preserves the same `PrincipalId`
- adds one new unique authentication method
- merges new assurances with existing assurances
- rejects duplicate assurances
- updates `authenticated_at`

A step-up can fail because method or assurance invariants are violated.

Use:

```text
StepUpError
```

to represent those failures.

## Reauthentication

Reauthentication is distinct from step-up authentication.

A reauthentication:

- preserves the same principal
- preserves the same authentication methods
- replaces the previous assurance collection
- updates `authenticated_at`

Reauthenticating with the same method must not add duplicate authentication methods.

## AuthenticationFailure

Expected authentication rejection is not a system error.

Use:

```rust
#[non_exhaustive]
pub enum AuthenticationFailure {
    InvalidCredentials,
    CredentialExpired,
    CredentialRevoked,
    CredentialNotYetValid,
}
```

Do not add user-domain policy failures such as:

```text
PrincipalDisabled
UserSuspended
```

Do not add application policy failures such as:

```text
MethodNotAllowed
MfaRequired
InsufficientAssurance
AuthenticationTooOld
```

Those belong to other layers.

`AuthenticationFailure` should be:

```text
Clone
Copy
Debug
PartialEq
Eq
Display
Error
```

Internal `Display` messages may be specific.

External adapters are responsible for avoiding information leakage.

## AuthenticationOutcome

Authentication distinguishes rejection from operational failure.

Conceptually:

```rust
#[non_exhaustive]
pub enum AuthenticationOutcome {
    Authenticated(AuthenticatedPrincipal),
    Rejected(AuthenticationFailure),
}
```

An expected credential rejection is therefore:

```text
Ok(AuthenticationOutcome::Rejected(...))
```

not an operational `Err`.

The type should implement:

```text
Clone
Debug
PartialEq
Eq
```

Expected helpers include:

```text
is_authenticated
is_rejected
principal
failure
into_principal
into_failure
```

Conversions should exist from:

```text
AuthenticatedPrincipal
AuthenticationFailure
```

## Authenticator

Authentication mechanisms implement a strongly typed trait.

The core must not introduce a universal `Credential` enum.

Each authenticator owns its own credentials type.

Conceptually:

```rust
pub trait Authenticator: Send + Sync {
    type Credentials: Send + 'static;
    type Error: std::error::Error + Send + Sync + 'static;

    fn method_id(&self) -> &AuthenticationMethodId;

    fn authenticate(
        &self,
        credentials: Self::Credentials,
    ) -> impl Future<
        Output = Result<AuthenticationOutcome, Self::Error>
    > + Send;
}
```

Requirements:

- credentials are consumed by value
- credentials are not required to implement `Clone`
- credentials are not required to implement `Debug`
- errors are not required to implement `Clone`
- authenticator futures must be `Send`
- authenticators must be `Send + Sync`
- do not force object safety
- do not introduce dynamic dispatch until a real use case requires it

## Sensitive Credentials

Credential types may contain secrets.

Avoid deriving or requiring:

```text
Debug
Clone
Serialize
```

for secret-bearing values unless the specific mechanism proves it safe.

Passwords, API keys, tokens, recovery codes, and similar values must not accidentally appear in logs.

## Error Types

Public validation errors should:

- be specific to their value type
- implement `Debug`
- implement `Clone`
- implement `PartialEq`
- implement `Eq`
- implement `Display`
- implement `std::error::Error`
- be marked `#[non_exhaustive]`

Do not include complete invalid input values in error variants.

Safe diagnostic metadata is encouraged.

## Public API Stability

Public API design requires additional scrutiny.

Avoid exposing implementation details.

Prefer:

- opaque newtypes
- private struct fields
- validated constructors
- read-only getters
- explicit conversion traits
- small public APIs

Public enums intended to evolve should generally be `#[non_exhaustive]`.

Do not introduce public modules merely because the source code uses multiple files.

Internal modules should remain private and public types should normally be re-exported from the crate root.

Consumers should write:

```rust
use lumrel_auth::{
    AuthenticatedPrincipal,
    AuthenticationMethodId,
    PrincipalId,
};
```

rather than depending on physical source layout.

## Newtype API

Where appropriate, validated string newtypes should provide:

```text
new
as_str
into_inner
Display
AsRef<str>
Borrow<str>
TryFrom<String>
TryFrom<&str>
FromStr
From<T> for String
```

All constructors and conversion paths must reuse the same validation logic.

There must be no public path that bypasses validation.

## Serde

Serde support is optional.

Default features remain empty:

```toml
[features]
default = []
serde = ["dep:serde"]
```

The core should have zero mandatory external dependencies.

With the `serde` feature enabled, stable domain values may support serialization.

Do not serialize:

- the `Authenticator` trait
- implementation-specific operational errors
- arbitrary credentials

## Serde Invariants

Deserialization must never bypass domain validation.

Do not derive `Deserialize` in a way that permits invalid internal values.

All deserialization must reconstruct values through their validated rules.

Core rule:

> If a Lumrel value exists, it is valid by construction.

## Stable Serde Contract

Once published, the serde representation is part of Lumrel's public API.

Incompatible representation changes are breaking changes.

Unknown fields in structs should normally be tolerated for forward compatibility.

Do not enable:

```text
deny_unknown_fields
```

without a deliberate compatibility decision.

Enums use stable `snake_case` names.

## Serde Representation

String newtypes serialize transparently:

```json
"user-123"
```

not:

```json
{
  "value": "user-123"
}
```

`AuthenticationMethods` serializes conceptually as:

```json
{
  "primary": "password",
  "additional": [
    "totp"
  ]
}
```

`AuthenticationAssurances` serializes as a list:

```json
[
  "multi-factor",
  "phishing-resistant"
]
```

`AuthenticationFailure` serializes as a snake_case string:

```json
"invalid_credentials"
```

`AuthenticationOutcome` uses an explicit tagged representation with:

```text
status
```

For example:

```json
{
  "status": "rejected",
  "failure": "invalid_credentials"
}
```

## SystemTime Serialization

`SystemTime` must not depend on Chrono or `time` in the core.

Serde uses a canonical Unix timestamp representation containing:

```text
seconds
nanoseconds
```

Nanoseconds must always be:

```text
0..=999_999_999
```

Times before Unix epoch must be supported.

The representation must be canonical and round-trippable.

## Dependencies

Core crates should have the smallest practical dependency graph.

`lumrel-auth` should have:

```text
zero mandatory external dependencies
```

Optional functionality such as serde may introduce optional dependencies.

Before adding a dependency, consider:

- security history
- maintenance status
- license
- transitive dependencies
- compile time
- MSRV impact
- whether the dependency belongs in an adapter instead

Do not add dependencies for trivial convenience.

## Unsafe Rust

Core Lumrel crates should use:

```rust
#![forbid(unsafe_code)]
```

Do not introduce unsafe code into a core crate.

If a future low-level crate requires unsafe code, it must be explicitly designed and reviewed separately.

## Tests

Every invariant should have tests.

Tests should cover:

- valid construction
- every validation error
- boundary lengths
- Unicode behavior
- ordering guarantees
- duplicate detection
- maximum collection sizes
- conversions
- iteration
- step-up behavior
- reauthentication behavior
- serde round trips
- invalid serde input
- timestamps before and after Unix epoch

Bug fixes should include regression tests whenever practical.

## Required Checks

Before completing a change, run:

```bash
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo deny check
```

All must pass.

## CI

GitHub Actions executes independent checks in parallel.

Current CI categories are:

```text
Cargo check
Cargo test
Cargo fmt
Cargo clippy
Dependency audit
```

Do not merge code that intentionally leaves CI failing.

## Dependency Policy

The repository uses `cargo-deny`.

Dependency changes must comply with:

```text
deny.toml
```

The policy checks:

- security advisories
- yanked crates
- license allowlist
- wildcard dependencies
- dependency sources
- duplicate dependency versions

Do not weaken dependency policy merely to make a new dependency pass.

Document and justify exceptions.

## Documentation

Public types and methods should have useful Rustdoc documentation.

Documentation should explain:

- semantics
- invariants
- errors
- security consequences
- examples where valuable

Do not write comments that merely restate code.

Update project Markdown documentation when a change modifies architecture, guarantees, public API expectations, or contributor workflow.

## Commits

Prefer conventional commit style.

Examples:

```text
feat(auth): add principal identifier
feat(auth): add authentication method collection
fix(auth): reject duplicate authentication methods
test(auth): add serde invariant coverage
docs: update authentication architecture
ci: add dependency auditing
build(devcontainer): update Rust development environment
```

Keep commits logically focused.

## Security

Do not expose secrets in:

- errors
- logs
- panic messages
- snapshots
- test fixtures
- serde output

Do not implement custom cryptographic primitives.

Security-sensitive cryptography must use established libraries and protocols.

## Do Not

Do not:

- couple core crates to web frameworks
- couple core crates to databases
- add a universal credential enum
- add a universal credential repository
- make auth own user profiles
- silently normalize identifiers
- bypass constructors during deserialization
- expose struct fields merely for convenience
- add unnecessary dependencies
- add `unsafe` to authentication core
- weaken CI or cargo-deny without justification
- expose secret-bearing credentials through Debug
- prematurely create shared "common" utility crates

## Prefer

Prefer:

- validated domain types
- explicit invariants
- small crates
- narrow responsibilities
- adapters around infrastructure
- compile-time guarantees
- secure failure behavior
- deterministic tests
- additive extensibility
- explicit semantics
- boring, maintainable code over clever abstractions
