# Contributing to Lumrel

Thank you for your interest in contributing to Lumrel.

Lumrel aims to become a reliable collection of reusable backend components for Rust. Contributions of code, documentation, tests, architecture improvements, bug reports, and constructive technical discussion are welcome.

Because many Lumrel components may eventually be used in security-sensitive production systems, correctness, maintainability, and reviewability are prioritized over implementation speed.

## Before Contributing

For small bug fixes, documentation improvements, and tests, feel free to open a pull request directly.

For significant changes, new crates, major abstractions, public API redesigns, or architectural changes, please open a discussion or issue first.

This helps avoid spending time implementing an approach that conflicts with the project's architecture.

## Core Principles

Contributions should preserve the main Lumrel design principles:

- framework-agnostic core logic
- explicit adapters
- loose coupling between crates
- coherent APIs
- secure defaults
- replaceable implementations
- clear ownership of responsibilities
- minimal unnecessary dependencies
- predictable behavior
- strong automated testing

## Crate Boundaries

Each crate should have a clear and narrow responsibility.

A crate should not depend on unrelated Lumrel modules merely for convenience.

For example, an email crate should not require the authentication crate unless the dependency is fundamental to its responsibility.

Shared abstractions should only be extracted when multiple crates genuinely need them.

Avoid creating generic "common" modules that accumulate unrelated functionality.

## Core vs. Adapters

Framework-specific, database-specific, and provider-specific behavior should normally live outside core crates.

For example:

```text
lumrel-auth
    Framework-independent authentication concepts

lumrel-axum
    Axum-specific integration

lumrel-sqlx
    SQLx-specific integration
```

Exact crate names may evolve, but the separation principle should remain.

## Public APIs

Public APIs require additional care because changes may affect downstream applications.

When introducing a public API:

- keep it as small as practical
- avoid exposing implementation details
- prefer explicit types over loosely structured values
- document security-relevant behavior
- consider future extensibility
- avoid unnecessary generic complexity
- avoid locking users into a specific framework or provider

Breaking changes should be deliberate and documented.

## Security-Sensitive Code

Changes affecting areas such as the following require additional scrutiny:

- authentication
- authorization
- sessions
- tokens
- cryptography
- password handling
- secrets
- rate limiting
- input validation
- database authorization boundaries

Security-sensitive code should include tests covering both expected behavior and failure cases.

Do not introduce custom cryptographic algorithms.

Use established, reviewed cryptographic libraries and protocols.

## Dependencies

New dependencies should provide clear value.

Before adding a dependency, consider:

- maintenance status
- security history
- API stability
- transitive dependency cost
- compile-time impact
- feature requirements
- whether the functionality is small enough to implement safely without it

Avoid adding large dependency trees for minor conveniences.

## Feature Flags

Optional integrations should use Cargo feature flags where appropriate.

Features should:

- have clear names
- avoid surprising behavior
- remain additive where possible
- avoid silently changing security behavior

Default features should remain conservative.

## Tests

New functionality should normally include tests.

Depending on the change, this may include:

- unit tests
- integration tests
- compile tests
- property tests
- regression tests
- failure-path tests

Bug fixes should include a regression test whenever practical.

Run:

```bash
cargo test --workspace
```

before submitting a pull request.

## Formatting

Code must be formatted with `rustfmt`.

Run:

```bash
cargo fmt --all -- --check
```

To automatically format the workspace:

```bash
cargo fmt --all
```

## Linting

The workspace should remain clean under Clippy.

Run:

```bash
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

Avoid suppressing Clippy warnings without a clear reason.

When suppression is necessary, keep it narrowly scoped and document why.

## Documentation

Public items should be documented when their purpose or behavior is not immediately obvious.

Documentation should explain:

- what the API does
- important invariants
- failure behavior
- security implications
- examples when useful

Avoid documentation that merely repeats an identifier's name.

## Error Handling

Libraries should return structured errors rather than panic during normal failure conditions.

Panics should be reserved for situations involving violated internal invariants or conditions that cannot be handled meaningfully.

Security-sensitive errors should avoid exposing secrets or unnecessary internal details.

## Commits

Keep commits focused and understandable.

Prefer commits that represent one logical change.

Commit messages should explain the intent of the change rather than merely describing modified files.

## Developer Certificate of Origin

Lumrel uses the Developer Certificate of Origin, version 1.1.

By contributing, you certify that you have the right to submit the contribution under the project's license.

Commits must include a `Signed-off-by` line:

```text
Signed-off-by: Your Name <your-email@example.com>
```

Git can add this automatically:

```bash
git commit -s
```

Use your real identity or another identity that you are legally permitted to use for the contribution.

A pull request containing commits without the required sign-off may need to be corrected before it can be merged.

## Pull Requests

A good pull request should:

- explain the problem being solved
- describe the chosen approach
- mention important design decisions
- include appropriate tests
- update documentation when necessary
- avoid unrelated changes
- pass formatting, tests, and linting

Large pull requests may be easier to review when divided into smaller logical changes.

## Breaking Changes

Breaking changes should include:

- a clear explanation of what changed
- the reason for the change
- migration guidance when appropriate
- relevant documentation updates

Once crates reach stable versions, Lumrel will follow semantic versioning.

## AI-Generated Contributions

AI-assisted development is allowed.

Contributors remain fully responsible for code they submit.

Before contributing AI-assisted code, verify:

- correctness
- licensing compatibility
- security properties
- tests
- documentation
- absence of fabricated APIs or assumptions

Do not submit generated code you do not understand or cannot maintain.

## Reporting Security Issues

Do not report vulnerabilities through normal public issues.

Follow [SECURITY.md](SECURITY.md).

## Community Conduct

All contributors must follow [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## License

By contributing to Lumrel, you agree that your contributions will be licensed under the same license that applies to the relevant Lumrel source code.

Unless explicitly stated otherwise, this is the MIT License.
