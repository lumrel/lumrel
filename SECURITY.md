# Lumrel Security Policy

Security is a core design priority for Lumrel.

Because Lumrel may provide foundational components for authentication, authorization, sessions, tokens, databases, and other backend infrastructure, security reports are taken seriously.

## Reporting a Vulnerability

Please **do not open a public GitHub issue** for suspected security vulnerabilities.

When GitHub Private Vulnerability Reporting is enabled for the repository, use it to submit the report privately.

If private vulnerability reporting is unavailable, use the private security contact published by the Lumrel project or repository maintainers.

Do not include exploit details, credentials, secrets, personal information, or proof-of-concept attack code in a public issue.

## What to Include

A useful security report should contain, when possible:

- the affected crate
- the affected version or commit
- a description of the vulnerability
- the expected behavior
- the observed behavior
- reproduction steps
- security impact
- environmental requirements
- relevant logs with secrets removed
- a minimal proof of concept, when appropriate
- suggested mitigations, if known

Clear reports make investigation significantly easier.

## Responsible Disclosure

Please allow maintainers a reasonable opportunity to investigate and address a vulnerability before publishing technical details.

The maintainers may coordinate disclosure timing with the reporter when a vulnerability affects released versions.

Once a fix is available, the project may publish:

- a security advisory
- affected versions
- fixed versions
- upgrade instructions
- mitigation guidance
- appropriate technical details

## Supported Versions

Lumrel is currently under active development.

Until stable releases are established, security fixes may be provided primarily for the latest maintained release or development branch.

Once the project reaches stable releases, this document will contain an explicit support matrix.

A future support table may look like:

| Version         | Supported                    |
| --------------- | ---------------------------- |
| Latest stable   | Yes                          |
| Previous stable | Defined by support policy    |
| Older releases  | No, unless explicitly stated |

This table is illustrative until stable versioning begins.

## Security Scope

Security issues may include, but are not limited to:

- authentication bypass
- authorization bypass
- privilege escalation
- session fixation
- session hijacking
- token validation flaws
- insecure JWT handling
- secret leakage
- credential exposure
- injection vulnerabilities
- unsafe deserialization
- cryptographic misuse
- incorrect password handling
- rate-limit bypass
- sensitive data exposure
- dangerous default configuration
- cross-tenant data access
- memory-safety issues caused by unsafe code

General bugs without security impact should be reported through the normal issue tracker.

## Cryptography

Lumrel should not invent custom cryptographic primitives or protocols.

Cryptographic functionality should rely on established implementations and recognized algorithms appropriate for the relevant use case.

Security-sensitive cryptographic choices should be documented.

## Unsafe Rust

Use of `unsafe` code should be minimized.

When `unsafe` is necessary, it should:

- be narrowly scoped
- document its safety assumptions
- preserve Rust's required invariants
- include appropriate tests
- receive additional review

Crates may adopt stricter `unsafe` policies where practical.

## Secrets

Lumrel should never intentionally log:

- passwords
- raw authentication tokens
- private keys
- API secrets
- session secrets
- database credentials

Code handling secrets should minimize unnecessary copies and exposure where practical.

## Dependencies

Dependency vulnerabilities are considered part of the Lumrel security surface.

The project may use automated tooling and dependency auditing to identify known vulnerabilities.

Security-sensitive dependencies should be selected conservatively.

## Security Is a Shared Responsibility

Lumrel can provide secure primitives and defaults, but application security also depends on correct integration and deployment.

Applications remain responsible for areas including:

- TLS configuration
- secret management
- infrastructure security
- access controls
- database configuration
- deployment configuration
- dependency updates
- monitoring
- incident response

Lumrel documentation should make important integration requirements explicit rather than assuming secure deployment automatically.

## Public Discussion After Disclosure

After a vulnerability has been fixed and responsibly disclosed, technical discussion is welcome.

Before coordinated disclosure is complete, please avoid publishing details that could enable exploitation of affected users.
