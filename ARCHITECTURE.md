# Lumrel Architecture

This document describes the architectural boundaries and design rules of the Lumrel ecosystem.

## Philosophy

Lumrel prefers composition over framework ownership.

The ecosystem should provide reusable backend capabilities without forcing applications into one:

- HTTP framework
- database
- runtime architecture
- deployment environment
- provider ecosystem

Core domain logic remains independent.

Infrastructure is connected through adapters.

## Workspace Organization

The repository groups crates by domain:

```text
crates/<domain>/<component>
```

For example:

```text
crates/auth/core
crates/auth/password
crates/auth/passkey
```

Published package names use the Lumrel namespace:

```text
lumrel-auth
lumrel-auth-password
lumrel-auth-passkey
```

This allows the repository to remain easy to navigate while keeping crates.io names descriptive and consistent.

## Dependency Direction

Dependencies should point inward toward stable abstractions.

Correct:

```text
lumrel-auth-password
        │
        ▼
    lumrel-auth
```

Correct:

```text
lumrel-auth-axum
        │
        ▼
    lumrel-auth
```

Incorrect:

```text
lumrel-auth
    │
    ▼
lumrel-auth-axum
```

Core crates must never depend on their infrastructure adapters.

## Core Crates

A core crate owns domain concepts.

Core crates should contain:

- value objects
- domain errors
- domain traits
- invariants
- framework-independent behavior

Core crates should not contain:

- database clients
- HTTP request types
- provider SDKs
- cloud configuration
- framework extractors
- transport-specific errors

## Mechanism Crates

Authentication mechanisms live outside `lumrel-auth`.

Examples:

```text
lumrel-auth-password
lumrel-auth-passkey
lumrel-auth-api-key
lumrel-auth-magic-link
```

A mechanism crate may define:

- credentials
- mechanism-specific storage traits
- validation rules
- mechanism-specific errors
- authenticator implementations

A mechanism should reuse the common authentication contract without forcing the core to understand its internals.

## Storage Abstractions

Lumrel intentionally avoids a universal credential repository.

For example, password storage and passkey storage have different domain requirements.

Prefer:

```text
PasswordCredentialStore
PasskeyCredentialStore
ApiKeyStore
```

over:

```text
CredentialRepository<T>
```

unless a shared abstraction emerges from real repeated requirements.

## Framework Adapters

Framework-specific integrations belong in adapter crates.

An Axum adapter may provide:

- extractors
- middleware
- request integration
- response conversion

It must depend on domain crates, not the reverse.

## Users Domain

Authentication and users remain independent.

A users domain may own:

```text
UserId
User
Profile
AccountStatus
```

Authentication instead uses:

```text
PrincipalId
```

An application or adapter maps between those domains.

This prevents authentication from becoming coupled to a specific account model.

## PrincipalId

`PrincipalId` is intentionally opaque.

It may represent:

- a user
- a service account
- a machine identity
- another authenticated principal

The core does not impose UUID or ULID semantics.

## Authentication Method Extensibility

Lumrel does not use one closed enum containing every possible authentication mechanism.

Instead:

```rust
AuthenticationMethodId
```

is extensible.

This allows external crates to introduce methods without changing `lumrel-auth`.

## Authenticator Contract

Each mechanism implements a strongly typed authenticator.

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

Credentials remain mechanism-specific.

The core must not introduce a universal credentials enum.

## Rejection vs Operational Failure

Authentication has two fundamentally different failure classes.

A normal credential rejection is domain behavior:

```text
InvalidCredentials
CredentialExpired
CredentialRevoked
CredentialNotYetValid
```

Operational failures include:

- database unavailable
- provider unavailable
- configuration failure
- internal service failure

Those are returned through the authenticator's associated error type.

This distinction prevents expected authentication failures from being treated as infrastructure errors.

## Authentication Methods

`AuthenticationMethods` represents the methods that established the current authenticated principal.

It contains:

```text
one primary method
zero or more additional methods
```

It preserves authentication order and prohibits duplicates.

The maximum number of methods is:

```text
16
```

## Authentication Assurances

`AuthenticationAssurances` describes properties established by authentication.

It may be empty.

Examples:

```text
multi-factor
phishing-resistant
hardware-backed
```

The maximum number of assurances is:

```text
16
```

Values are unique and ordered.

There is deliberately no global assurance ordering.

## AuthenticatedPrincipal

An authenticated principal contains:

```text
PrincipalId
AuthenticationMethods
AuthenticationAssurances
SystemTime
```

The value is immutable.

All fields remain private.

Consumers interact through validated constructors and read-only methods.

## Step-Up

A step-up:

```text
same principal
+ new unique method
+ merged assurances
+ new authenticated_at
```

It produces a new principal value.

## Reauthentication

A reauthentication:

```text
same principal
same methods
new assurance collection
new authenticated_at
```

It also produces a new principal value.

## Value Object Invariants

Lumrel aims to make invalid states unrepresentable where practical.

Validated types must not expose constructors or deserialization paths that bypass their invariants.

Input values are not silently normalized.

## Serde Design

Serde is optional.

Serialization belongs only to stable domain values.

The serialization format is a compatibility contract.

Newtype identifiers serialize transparently as strings.

Methods preserve:

```text
primary
additional
```

Assurances serialize as an ordered list.

Authentication outcomes use an explicit status tag.

Enums use snake_case.

Unknown struct fields should generally be tolerated.

## Time Representation

`AuthenticatedPrincipal` uses:

```rust
std::time::SystemTime
```

The core does not require Chrono or the `time` crate.

Serde represents the timestamp canonically using:

```text
seconds
nanoseconds
```

with support for times before Unix epoch.

## Dependency Philosophy

Core crates should avoid mandatory dependencies where possible.

A dependency should be added because it provides material value, not minor convenience.

Mechanism and adapter crates may naturally have larger dependency graphs than domain cores.

## Shared Crates

Do not create a generic:

```text
lumrel-core
lumrel-common
lumrel-utils
```

simply to share small helpers.

Shared abstractions should emerge only after multiple domains demonstrate a genuine common requirement.

This avoids turning a common crate into a dumping ground and prevents unnecessary coupling between domains.

## Security

Security-sensitive behavior should be explicit and testable.

Lumrel does not implement custom cryptographic primitives.

Core authentication code forbids unsafe Rust.

Credentials and secrets must not be accidentally printable or serializable.

## Compatibility

The project uses semantic versioning.

During `0.x`, APIs may evolve more rapidly.

Even during early development, breaking changes should remain deliberate and documented.

Once a serde representation is published, incompatible representation changes are treated as API compatibility changes.
