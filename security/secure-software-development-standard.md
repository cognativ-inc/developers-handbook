# Cognativ Secure Software Development Standard

| Field | Value |
| --- | --- |
| Document | Cognativ Secure Software Development Standard |
| Owner | Engineering |
| Applies to | Cognativ software engineering (employees, contractors, and any engineer contributing to Cognativ-delivered software) |
| Status | Active |
| Version | 1.0 |
| Last reviewed | 2026-08-24 |
| Review cadence | At least annually, and after any significant security, tooling, or process change |
| Source of record | <https://github.com/cognativ-inc/developers-handbook/> |

Part of the [Cognativ Developers Handbook](../README.md). See also the
[HECVAT / compliance mapping](compliance-mapping.md) and
[Onboarding and secure-development training](../onboarding/README.md).

---

## Requirement language

This standard uses the following terms deliberately:

- **MUST** — a mandatory Cognativ engineering requirement. Code that violates a MUST should not be merged.
- **SHOULD** — expected practice. Deviation is allowed only with a documented reason (for example, in the pull request description or an architecture note).
- **MAY** — optional or context-dependent.

Where a requirement depends on a capability the project does not have (for example, a scanner that is not available on a client-owned platform), engineers MUST record the gap with the project lead rather than silently ignoring the requirement.

---

## Table of contents

1. [Scope and purpose](#1-scope-and-purpose)
2. [Secure development principles](#2-secure-development-principles)
3. [OWASP alignment](#3-owasp-alignment)
4. [Authentication](#4-authentication)
5. [Authorization](#5-authorization)
6. [Input validation and output encoding](#6-input-validation-and-output-encoding)
7. [Database security](#7-database-security)
8. [Secrets management](#8-secrets-management)
9. [Sensitive data and PII](#9-sensitive-data-and-pii)
10. [Encryption](#10-encryption)
11. [Error handling](#11-error-handling)
12. [Security logging and monitoring](#12-security-logging-and-monitoring)
13. [Dependency and supply-chain security](#13-dependency-and-supply-chain-security)
14. [Source-control security](#14-source-control-security)
15. [Code review](#15-code-review)
16. [Testing](#16-testing)
17. [CI/CD security](#17-cicd-security)
18. [Vulnerability management](#18-vulnerability-management)
19. [Secure configuration](#19-secure-configuration)
20. [API security](#20-api-security)
21. [Frontend security](#21-frontend-security)
22. [File handling](#22-file-handling)
23. [Infrastructure and cloud security](#23-infrastructure-and-cloud-security)
24. [AI-assisted software development](#24-ai-assisted-software-development)
25. [Third-party services and APIs](#25-third-party-services-and-apis)
26. [Security incidents](#26-security-incidents)
27. [Exceptions](#27-exceptions)

---

## 1. Scope and purpose

### Purpose

This standard defines the minimum security requirements for software designed, written, reviewed, deployed, or maintained by Cognativ engineers. It exists so that:

- security decisions are made consistently across projects and technology stacks;
- engineers know what is expected of them without having to rediscover it per project;
- Cognativ can show clients and assessors a documented, auditable secure-coding standard.

### Who it applies to

This standard applies to every engineer who contributes to Cognativ-delivered software: employees, contractors, and temporary or partner engineers working inside Cognativ repositories or on Cognativ-managed deliverables. It applies to backend, frontend, mobile, data, and infrastructure work.

### What it applies to

- Application and service code, including scripts and infrastructure-as-code.
- Database schemas, migrations, and queries.
- Build, test, and deployment pipelines.
- Configuration that affects the security posture of a running system.
- Code produced with the assistance of AI tools (see [section 24](#24-ai-assisted-software-development)).

### Expectations of developers

Engineers MUST read this standard, apply it while implementing and reviewing code, and raise questions when a requirement is unclear or appears impossible to meet in a given project. "I did not know" is not an acceptable outcome for a documented requirement; asking is.

### Interaction with project and client rules

This standard is the Cognativ baseline. It applies unless a client, contract, regulatory regime, or project-specific standard imposes stricter requirements — in which case the stricter requirement wins.

Precedence, highest first:

1. Applicable law, regulation, and contractual security obligations.
2. Client or project-specific security standards and platform controls.
3. This standard.
4. General guidance in the rest of the handbook.

A project MUST NOT relax a requirement in this standard simply because a client did not ask for it. Where a project genuinely cannot meet a requirement, use the [exceptions](#27-exceptions) process.

---

## 2. Secure development principles

These principles are how the rest of this standard should be interpreted when a specific rule does not exist for the situation in front of you.

- **Least privilege.** Every user, service, token, database account, and CI job gets the minimum permissions needed for its job, and nothing more.
- **Defense in depth.** Assume any single control can fail. Validate at the boundary *and* enforce in the domain logic *and* constrain at the data layer.
- **Secure by default.** New features, endpoints, buckets, and configuration flags start in their safest state. Security must not depend on someone remembering to turn it on.
- **Deny by default.** Access decisions start from "no" and require an explicit, positive grant. Allow-lists over deny-lists.
- **Minimize attack surface.** Do not expose endpoints, admin tooling, debug routes, ports, or data that the product does not need.
- **Explicit trust boundaries.** Know where data crosses from untrusted to trusted (browser → API, service → service, third party → us, file upload → processing). Validate at every crossing.
- **Fail securely.** When something breaks — a token cannot be verified, a permission service times out — the result MUST be denial, not access.
- **Minimize sensitive-data exposure.** Do not collect, transmit, log, cache, or persist sensitive data you do not need. The safest data is the data you never held.
- **Never trust client-side enforcement.** Client-side validation, hidden fields, disabled buttons, and frontend role checks are usability features. Every security decision MUST be enforced server-side.
- **Keep it understandable.** Security logic that no reviewer can follow will eventually be wrong. Prefer clear, boring, well-reviewed mechanisms over clever ones.

---

## 3. OWASP alignment

Cognativ aligns its application security expectations with OWASP guidance, primarily:

- **OWASP Top 10** — the baseline risk categories every web engineer is expected to recognize: <https://owasp.org/www-project-top-ten/>
- **OWASP API Security Top 10** — for services and APIs: <https://owasp.org/www-project-api-security/>
- **OWASP Application Security Verification Standard (ASVS)** — for structured verification when a project or client needs it: <https://owasp.org/www-project-application-security-verification-standard/>
- **OWASP Cheat Sheet Series** — practical, technology-specific implementation guidance: <https://cheatsheetseries.owasp.org/>

Engineers MUST work against the **current published version** of these documents rather than a version pinned in this handbook. This standard deliberately does not restate a specific Top 10 edition, so that an OWASP release does not silently make Cognativ guidance obsolete.

Risk categories engineers are expected to understand and design against:

| Risk area | What we require | Detail |
| --- | --- | --- |
| Broken access control | Server-side, per-object authorization | [§5](#5-authorization) |
| Injection (SQL, NoSQL, OS command, LDAP, template, XSS) | Parameterized queries, safe APIs, context-aware output encoding | [§6](#6-input-validation-and-output-encoding), [§7](#7-database-security) |
| Identification and authentication failures | Established auth mechanisms, strong password storage, correct session/token handling | [§4](#4-authentication) |
| Insecure design | Threat-think before building sensitive features; security requirements are part of the ticket | [§2](#2-secure-development-principles), [§15](#15-code-review) |
| Security misconfiguration | Hardened, environment-specific configuration; no dev defaults in production | [§19](#19-secure-configuration) |
| Vulnerable and outdated components | Maintained dependencies, lockfiles, scanning, remediation | [§13](#13-dependency-and-supply-chain-security), [§18](#18-vulnerability-management) |
| Cryptographic failures | TLS everywhere, platform crypto, no homegrown algorithms | [§10](#10-encryption) |
| Security logging and monitoring failures | Log security-relevant events without leaking secrets or PII | [§12](#12-security-logging-and-monitoring) |
| Server-side request forgery (SSRF) | Treat user-supplied URLs as hostile; allow-list destinations | below |
| Software and data integrity failures | Trusted sources, verified artifacts, no untrusted install scripts | [§13](#13-dependency-and-supply-chain-security), [§17](#17-cicd-security) |

### SSRF

Any feature that fetches a URL, hostname, or file location supplied or influenced by a user (webhooks, "import from URL", link previews, avatar-by-URL, PDF/HTML renderers, image proxies) is an SSRF risk.

- Outbound requests to user-supplied destinations MUST be restricted by an allow-list of schemes, hosts, and ports where the product allows it.
- Requests to internal address ranges, link-local addresses, and cloud instance-metadata endpoints MUST be blocked.
- Redirects MUST be re-validated against the same rules, not followed blindly.
- Responses SHOULD be size- and time-limited, and MUST NOT be echoed back to the caller in raw form when that would leak internal detail.

---

## 4. Authentication

- Applications MUST use established, well-maintained authentication mechanisms — the framework's auth system, or a reputable identity provider / library (for example OIDC or SAML through a managed identity provider).
- Engineers MUST NOT design custom cryptographic authentication schemes, custom token formats with hand-rolled signing, or custom password-hashing constructions.
- Authentication MUST be enforced server-side on every protected route, endpoint, job, and message consumer — including internal service-to-service calls.

### Passwords and credential storage

- Passwords MUST be stored only as salted hashes produced by a modern password-hashing function: **Argon2id** (preferred), **scrypt**, or **bcrypt**. Work factors SHOULD follow current OWASP Password Storage guidance and be revisited periodically.
- General-purpose hash functions (MD5, SHA-1, SHA-256, SHA-512) MUST NOT be used to store passwords, with or without a salt.
- Password comparison MUST use the hashing library's verify function (constant-time), never string equality or a SQL `WHERE password = ...` comparison.
- Passwords MUST NOT appear in logs, URLs, analytics, error messages, or database columns in reversible form.
- Password policy SHOULD favor length and breached-password screening over forced composition rules and forced periodic rotation.
- Credential reset flows MUST use single-use, expiring, high-entropy tokens and MUST NOT reveal whether an account exists beyond what the product deliberately allows.

### Sessions and tokens

- Session identifiers and tokens MUST be generated with a cryptographically secure random source and MUST be rotated on privilege change (login, elevation, impersonation).
- Access tokens MUST have a short expiration. Refresh tokens, where used, MUST be revocable and SHOULD rotate on use.
- Server-side logout and revocation MUST invalidate the session or token; clearing it only in the browser is not logout.
- Browser sessions MUST use cookies marked `HttpOnly`, `Secure`, and an appropriate `SameSite` value. Long-lived tokens SHOULD NOT be stored in `localStorage` (see [§21](#21-frontend-security)).
- JWT (or equivalent) verification MUST validate signature, algorithm, issuer, audience, and expiry. `alg: none` and caller-selected algorithms MUST be rejected.
- Machine credentials (API keys, service tokens) MUST be scoped, revocable, and rotatable, and MUST be handled as secrets ([§8](#8-secrets-management)).

### Multi-factor authentication

- MFA MUST be enabled for privileged access where the platform supports it: source-control organizations, cloud consoles, CI/CD systems, secret managers, production databases and admin panels.
- Where Cognativ builds an application with administrative roles, the design SHOULD support MFA or delegate authentication to an identity provider that does.

---

## 5. Authorization

Authentication answers *"who is this?"*. Authorization answers *"is this actor allowed to perform this action on this specific resource?"*. They are separate checks and MUST both be implemented.

- Authorization MUST be enforced server-side, at a trusted boundary, on every request — including requests that the UI would never generate.
- **Object-level (resource) authorization MUST be checked on every access to a specific record**, not only on the route. Verify that the authenticated principal is entitled to *that* identifier. Do not rely on the identifier being unguessable.
- Roles, permissions, scopes, tenant IDs, and entitlements MUST be resolved from trusted server-side state (session, verified token claims, database) and MUST NOT be read from request bodies, query strings, or client-provided headers.
- UI behavior — hidden menus, disabled buttons, filtered lists — is UX. It MUST NOT be the only thing preventing an action.
- Access decisions MUST deny by default. A missing or unrecognized permission is a denial.
- Privilege escalation MUST be prevented in both directions:
  - **Horizontal** — user A MUST NOT read or modify user B's resources (including across tenants, organizations, or projects).
  - **Vertical** — a standard user MUST NOT reach administrative capability, and MUST NOT be able to grant themselves roles, change their own tenant, or edit protected fields through a mass-assignment or partial-update endpoint.
- Mass assignment MUST be controlled: only explicitly permitted fields may be written from request payloads. Fields such as `role`, `is_admin`, `owner_id`, `tenant_id`, `price`, and `status` MUST NOT be writable just because they exist on the model.
- Authorization logic SHOULD be centralized (policies, guards, middleware, a permission service) rather than duplicated ad hoc in each handler, and MUST be covered by tests ([§16](#16-testing)).
- Background jobs, scheduled tasks, webhooks, exports, and bulk operations MUST apply the same authorization rules as the interactive path.

---

## 6. Input validation and output encoding

Treat all input from outside the trust boundary as hostile: request bodies, query strings, path parameters, headers, cookies, uploaded files, webhook payloads, third-party API responses, message-queue payloads, and data previously stored by another party.

- Input MUST be validated at the trust boundary, on the server, before use — ideally with a schema validator that rejects unknown or malformed data rather than coercing it.
- Validation MUST check type, required/optional status, length or range, and format. Where the set of valid values is known, validation MUST use an allow-list (enum, pattern, bounded set) rather than trying to filter out bad values.
- Client-side validation MAY exist for user experience but MUST be duplicated server-side.
- Rejecting invalid input is preferred over silently "cleaning" it. Where normalization is required (trimming, Unicode normalization, canonicalizing paths), it MUST happen *before* validation, not after.
- Requests SHOULD have size limits (body size, array lengths, string lengths, page sizes, upload size, JSON nesting depth) to prevent resource exhaustion.

### Injection

- **SQL/NoSQL:** queries MUST use parameterized statements or a safe query builder/ORM API. String concatenation or interpolation of untrusted values into a query is prohibited. See [§7](#7-database-security).
- **OS commands:** invoking a shell with untrusted input MUST be avoided. Where a subprocess is genuinely required, use an argument-array API (no shell interpolation), validate arguments against an allow-list, and never pass user input as a full command string.
- **Path traversal:** file paths derived from user input MUST be validated and resolved, then confirmed to remain inside the intended base directory. Reject `..`, absolute paths, symlink escapes, and null bytes. Prefer generated identifiers over user-supplied filenames.
- **Template/expression injection:** untrusted input MUST NOT be evaluated as a template, expression, or code. `eval` and equivalent dynamic-code constructs (including `new Function`, and passing strings to `setTimeout`/`setInterval`) MUST NOT be used with data that is influenced by user input.
- **Deserialization:** untrusted data MUST NOT be deserialized into arbitrary types with formats that can instantiate classes or execute code (native/binary serializers, unsafe YAML loaders, `pickle`, `BinaryFormatter`). Use data-only formats such as JSON with an explicit schema.
- **Header/log injection:** untrusted values placed into HTTP headers, redirects, filenames, or log lines MUST be sanitized of control characters and newlines. Redirect targets MUST be validated against an allow-list.

### Output encoding

- Output MUST be encoded for the context it enters: HTML body, HTML attribute, JavaScript, URL, CSS, SQL identifier, shell argument, CSV/spreadsheet.
- Encoding MUST use the framework's context-aware escaping (for example React's default JSX escaping, or a template engine's auto-escaping) rather than hand-written replace chains.
- Where raw HTML from an untrusted source must be rendered, it MUST be sanitized by a maintained HTML sanitizer library before rendering. See [§21](#21-frontend-security).

### File uploads

Covered in detail in [§22](#22-file-handling).

---

## 7. Database security

- All queries MUST use parameter binding or a safe query API. Dynamic SQL MUST NOT interpolate untrusted values; where table/column names must be dynamic, they MUST be mapped from an allow-list, never taken from input.
- Application database credentials MUST be least-privileged: the application account SHOULD NOT own schema-migration rights, superuser rights, or access to unrelated schemas. Read-only workloads (reporting, analytics) SHOULD use read-only accounts.
- Database credentials and connection strings MUST NOT be committed to source control ([§8](#8-secrets-management)).
- Authentication MUST NOT be performed by comparing a password inside a SQL query. Fetch the credential record by identifier, then verify the hash in application code with the hashing library's verify function.
- **Migrations:**
  - MUST be version-controlled, reviewed, and reproducible.
  - SHOULD be forward-only in production, with a tested rollback or compensating migration for anything destructive.
  - Destructive changes (dropping columns/tables, type narrowing, backfills over large tables) MUST be called out explicitly in the pull request and SHOULD be run in an expand/contract sequence to avoid data loss and downtime.
  - MUST NOT contain hard-coded secrets, real user data, or production identifiers.
- **Sensitive columns:** sensitive fields (credentials, tokens, government identifiers, health or financial data) MUST be identified during design, MUST be excluded from generic `SELECT *` reads and default API serializers, SHOULD be encrypted or tokenized where the threat model calls for it, and MUST NOT be duplicated into logs, caches, or analytics stores without a documented reason.
- **Production data outside production:** production data MUST NOT be copied into development, test, demo, or local environments unless it has been anonymized, pseudonymized, or masked, and the copy is authorized by the project lead and the client's terms. Prefer synthetic or generated fixtures.
- **Backups:** where engineers are responsible for backups, backups MUST exist for production data stores, MUST be encrypted, MUST have access restricted to the same standard as the source data, and restores SHOULD be tested — an untested backup is not a backup. Retention SHOULD match the project's contractual and regulatory requirements.
- Deletion requirements (for example, honoring a deletion request) MUST consider replicas, caches, search indexes, exports, and backups, not just the primary table.

See also [standards/sql.md](../standards/sql.md) for SQL and PostgreSQL practice.

---

## 8. Secrets management

A "secret" is anything that grants access or proves identity: passwords, API keys, access and refresh tokens, private keys, certificates, signing keys, webhook signing secrets, database connection strings, cloud credentials, and service-account files.

### Hard rules

- Secrets MUST NOT be committed to Git — not in source files, config files, notebooks, test fixtures, seed data, Docker files, IaC templates, comments, or documentation. This applies to every branch, including branches that are never merged.
- Real credentials MUST NOT be pasted into issue trackers, chat, screenshots, screen recordings, log output, sample payloads, or AI prompts ([§24](#24-ai-assisted-software-development)).
- Production credentials MUST NOT be used on developer machines or in test environments.
- Secrets MUST be read at runtime from an approved mechanism for the project: a managed secret store (for example AWS Secrets Manager or SSM Parameter Store, Azure Key Vault, Google Secret Manager, HashiCorp Vault), the platform's encrypted environment/secret configuration, or the CI/CD system's secret store. Which mechanism is approved is a per-project decision recorded by the project lead.
- Configuration MUST be environment-specific. The same secret MUST NOT be shared across development, staging, and production.
- Secrets SHOULD be scoped as narrowly as the provider allows (per-service, per-environment, minimum permissions, IP or resource restrictions where available) and SHOULD be rotatable without a code change.

### Local development

- `.env` files (and equivalents) MUST be listed in `.gitignore` and MUST NOT be committed. Commit a `.env.example` with keys and placeholder values only.
- Local development SHOULD use non-production, low-value credentials against non-production resources.
- Developers SHOULD enable a pre-commit or local secret-scanning check where the project provides one, and MUST NOT bypass it (for example with `--no-verify`) to push a change containing a secret.
- Where a project has no secret-scanning tooling, engineers SHOULD still review their own diff before pushing. `git diff --staged` before committing catches most accidents.

### If a secret is exposed

Treat any exposure as a compromise, even inside a private repository, and even if the commit was amended or force-pushed. Git history, forks, clones, CI logs, and caches persist.

1. **Rotate or revoke the credential immediately.** Rotation is the fix; deleting the line is not.
2. Notify the project lead / engineering leadership per [§26](#26-security-incidents).
3. Remove the value from the code and move it to the approved secret mechanism.
4. Purge from history only after rotation, and only in coordination with the repository owner.
5. Check whether the credential was used by anyone else during the exposure window.

---

## 9. Sensitive data and PII

Cognativ projects can involve personal, student, health-adjacent, or financial information. Engineers are expected to handle it carefully **whether or not** a specific regulation has been identified for the project.

> Cognativ does not claim that every application it builds is subject to FERPA, HIPAA, PCI DSS, GDPR, or any other specific regime. Where a project *is* in scope for one of these, the client's or project's compliance requirements are authoritative and stricter than this section. Engineers MUST ask the project lead if they are unsure which regime applies.

### Requirements

- **Data minimization.** Collect, transmit, store, and retain only the data the product needs. If a field is not needed, do not add it; if it is no longer needed, plan its removal.
- **Classification.** Sensitive fields MUST be identified during design of any feature that touches personal data, and the design MUST state where that data is stored and who can read it.
- **Logs and telemetry.** Credentials, tokens, full payment data, government identifiers, health data, and full request/response bodies containing personal data MUST NOT be written to logs, traces, metrics labels, analytics events, or crash reports. See [§12](#12-security-logging-and-monitoring).
- **Masking and redaction.** Where sensitive values must be shown or logged for support purposes, they MUST be masked (for example, last four digits only). Redaction SHOULD be implemented centrally (a logging serializer or scrubber) so it cannot be forgotten at each call site.
- **Access control.** Access to personal data MUST be least-privilege and enforced by the application ([§5](#5-authorization)) and by the platform ([§23](#23-infrastructure-and-cloud-security)). Support and admin tooling that exposes user data MUST require appropriate privilege and SHOULD log the access.
- **Production data handling.** Engineers MUST NOT extract production personal data to local machines, personal cloud accounts, spreadsheets, or third-party tools. Where production access is required to diagnose an issue, it SHOULD be time-boxed, authorized, and logged.
- **Test data.** Test and demo data MUST be synthetic or anonymized. Fixtures and seeds MUST NOT contain real names, emails, phone numbers, or identifiers of real people.
- **Retention and deletion.** Where a project defines retention or deletion requirements, the implementation MUST honor them across primary stores, replicas, caches, search indexes, and exports, and MUST account for backups. Where no requirement is defined and personal data is involved, engineers SHOULD ask rather than assume indefinite retention is acceptable.
- **Transmission.** Personal data MUST be transmitted over TLS ([§10](#10-encryption)) and MUST NOT be placed in URLs, query strings, or referrer-exposed locations.

---

## 10. Encryption

- **In transit:** all network communication carrying application data MUST use TLS. This includes browser traffic, API calls, service-to-service calls, database connections where supported, and webhooks. Plain HTTP MUST NOT be used for anything but a redirect to HTTPS.
- TLS configuration SHOULD use current, supported protocol versions and cipher suites, and certificate validation MUST NOT be disabled. Code that sets a "skip certificate verification" flag MUST NOT reach a shared branch.
- HSTS SHOULD be enabled for production web applications.
- **At rest:** managed data stores, object storage, backups, and disk volumes holding application or personal data MUST have encryption at rest enabled where the platform provides it (this is usually a configuration flag, not code). Additional field-level or envelope encryption SHOULD be applied to particularly sensitive fields when the threat model calls for it.
- **Approved cryptography:** engineers MUST use the platform or a well-maintained, widely reviewed library for cryptographic operations — the language's standard crypto library, the framework's data-protection API, or a managed KMS. Prefer high-level, misuse-resistant APIs (authenticated encryption such as AES-GCM or ChaCha20-Poly1305) over assembling primitives.
- **No custom cryptography.** Engineers MUST NOT invent algorithms, key-derivation schemes, or protocols; MUST NOT use ECB mode; MUST NOT reuse a nonce/IV with the same key; MUST NOT use MD5 or SHA-1 for security purposes; and MUST NOT use encryption where a signature or a password hash is what is actually needed.
- Randomness used for security purposes (tokens, IDs, salts, nonces) MUST come from a cryptographically secure generator, not from a general-purpose `random()`.
- **Key handling:** keys are secrets ([§8](#8-secrets-management)). Keys MUST NOT be committed, MUST be stored in a secret manager or KMS, SHOULD be distinct per environment, SHOULD support rotation, and MUST NOT be logged. Where a KMS is available, keys SHOULD stay inside it rather than being exported to the application.

---

## 11. Error handling

- Production responses to end users MUST NOT expose stack traces, framework or library versions, SQL statements, file paths, internal hostnames, or raw exception messages. Debug error pages MUST be disabled in production ([§19](#19-secure-configuration)).
- External errors MUST be generic and useful: a stable error code or message plus, where helpful, a correlation ID the user can quote to support. The detail belongs in the internal log entry keyed by that same ID.
- Exception messages and error payloads MUST NOT contain secrets, tokens, credentials, or personal data — including in the "helpful debugging context" appended to an error.
- Security-relevant failures MUST NOT leak information through the *shape* of the error: authentication failures SHOULD NOT distinguish "no such user" from "wrong password" unless the product deliberately requires it, and authorization failures SHOULD NOT confirm the existence of a resource the caller may not see.
- Failure cases MUST be handled explicitly. Empty `catch` blocks, swallowed rejections, and ignored error returns are prohibited; a caught error MUST be handled, translated, or logged with enough context to act on.
- Errors MUST fail closed. If a permission check, token verification, signature check, or dependent security service errors out, the request MUST be denied rather than allowed through.
- Internal diagnostics SHOULD be structured and correlated (request ID, operation, actor identifier — not actor personal data) so incidents can be reconstructed.

See also [standards/general-best-practices.md](../standards/general-best-practices.md) for language-level error-handling practice.

---

## 12. Security logging and monitoring

### What to log

Security-relevant events SHOULD be logged with enough context to answer "who did what, to what, when, and from where":

- Authentication events: successes, failures, lockouts, MFA challenges, password and email changes, credential resets.
- Authorization failures, especially repeated denials against resources the actor does not own.
- Privileged and administrative actions: role and permission changes, impersonation, configuration changes, data exports, bulk deletions, manual data edits.
- Security-relevant lifecycle events: account creation and deletion, API key or token issuance and revocation, webhook secret changes.
- Integrity and validation failures that suggest attack: rejected signatures, malformed tokens, input rejected by security validation, rate-limit trips.
- Where practical, abnormal patterns worth alerting on: spikes in authentication failures, unusual export volume, unexpected traffic from a single principal.

### How to log

- Logs MUST NOT contain secrets, tokens, passwords, session identifiers, full payment data, or unnecessary personal data. Redaction SHOULD be centralized ([§9](#9-sensitive-data-and-pii)).
- Logs SHOULD be structured (for example JSON) with consistent field names, including a timestamp with timezone, an event/action name, an actor identifier, a target identifier, an outcome, and a correlation/request ID.
- Values interpolated into log messages MUST be sanitized of newlines and control characters to prevent log forging.
- Security-relevant logs SHOULD be retained long enough to investigate an incident, SHOULD be protected from modification by the application itself, and SHOULD be accessible to the people who would need them during an incident.
- Alerting SHOULD exist for the events a human would need to react to. A log nobody reads is not monitoring.

> Logging destination and tooling are project-level infrastructure decisions. This standard specifies *what* must be captured and *what must never appear*; it does not mandate a particular logging platform, since Cognativ projects run on different clouds and client-owned infrastructure.

---

## 13. Dependency and supply-chain security

- Every dependency MUST have a legitimate need. Prefer the standard library or a few lines of local code over a package for trivial functionality.
- Dependency count SHOULD be minimized. Each addition is code Cognativ ships and is responsible for.
- Dependencies MUST be actively maintained and MUST be pulled from the ecosystem's official registry. Abandoned packages, packages with a single unaccountable maintainer for security-critical functionality, and packages installed from arbitrary URLs or forks SHOULD be avoided; where a fork or vendored copy is unavoidable, it MUST be reviewed and recorded in the pull request.
- Versions MUST be pinned by a committed lockfile (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `poetry.lock`, `Pipfile.lock`, `packages.lock.json`, `Gemfile.lock`, `go.sum`, `Cargo.lock`, …). Lockfiles MUST be committed and MUST be regenerated by the package manager, not hand-edited.
- CI installs SHOULD use reproducible, lockfile-respecting commands (`npm ci`, `pnpm install --frozen-lockfile`, `dotnet restore --locked-mode`, `pip install -r requirements.txt --require-hashes` where feasible) rather than resolving fresh versions on every build.
- Before adding a significant dependency, engineers SHOULD check: is it maintained (recent releases, open issue handling)? How widely used is it? What is its own dependency footprint? What license does it carry? Does it need network, filesystem, or credential access? Major additions MUST be visible in code review ([§15](#15-code-review)).
- Beware name-confusion attacks: verify the exact package name and its publisher before installing. Typosquats and lookalike packages are a routine attack.
- **Install-time scripts:** engineers MUST NOT run `curl … | bash`, unvetted installer scripts, or postinstall-heavy packages from untrusted sources on development or CI machines. Where a tool must be installed this way, the script MUST be reviewed first and pinned to a known version.
- Dependency vulnerability scanning MUST be used where the platform provides it (see [§17](#17-cicd-security) for the distinction between required capability and available implementation), and reported vulnerabilities MUST be triaged per [§18](#18-vulnerability-management).
- Transitive dependencies count. A vulnerability reachable through a nested dependency is still the project's vulnerability.
- Generated or vendored artifacts (build output, minified bundles, container images) MUST NOT be treated as reviewed code and SHOULD NOT be committed unless the project deliberately requires it.

---

## 14. Source-control security

- Repository access MUST be least-privilege: contributors get the access their role needs, administrative and owner rights are limited to those who need them, and access to client-owned repositories follows the client's rules.
- Access MUST be removed promptly when someone leaves a project or the company, including personal access tokens, deploy keys, SSH keys, and third-party app authorizations. This is an operational process; engineers MUST cooperate with it and MUST NOT retain access they no longer need.
- MFA MUST be enabled on source-control accounts used for Cognativ work where the platform supports it.
- **Branch protection SHOULD be enabled** on long-lived branches (`main`/`master`, `dev`/`develop`, release branches) where the hosting platform and the repository owner allow it: no direct pushes, pull request required, status checks required where CI exists. On client-owned repositories, the client's configuration applies.
- Direct commits to protected/long-lived branches MUST NOT be made; all changes go through a pull request. See [standards/git.md](../standards/git.md).
- Secrets MUST NOT be committed ([§8](#8-secrets-management)).
- Sensitive generated artifacts MUST NOT be committed: `.env` files, credential and key files (`*.pem`, `*.p12`, `*.keystore`, service-account JSON), local database dumps, coverage or profiling output containing real data, editor and OS files with local paths. `.gitignore` MUST be maintained to reflect this.
- Repository history MUST NOT be rewritten on shared branches. Where history rewriting is genuinely required (for example after a secret exposure), it MUST be coordinated with the repository owner.
- Public visibility MUST be a deliberate decision. Cognativ or client code MUST NOT be pushed to a personal or public repository, gist, paste site, or fork outside the approved organization.
- **Commit signing** is not currently a Cognativ-wide requirement. Projects and clients MAY require signed commits or a signed-commit policy; where they do, engineers MUST comply. Engineers MAY sign commits regardless.

---

## 15. Code review

Code review is mandatory at Cognativ ([standards/code-review.md](../standards/code-review.md) covers process and etiquette). This section covers the security expectations of a reviewer.

- Reviewers MUST review for security, not only correctness and style. Approving a change means the reviewer looked at it.
- Authors MUST make security-relevant changes easy to review: a description of what changed and why, an explicit note when the change touches a sensitive area, and small, focused pull requests rather than large mixed ones.
- Changes in the following areas are **security-sensitive** and SHOULD receive review from someone with relevant experience — a project lead or senior engineer where available — and SHOULD NOT be merged on a "looks fine" skim:
  - authentication, session, and token handling;
  - authorization, roles, permissions, and tenancy;
  - cryptography, key handling, and password storage;
  - payments, billing, and financial calculations;
  - personal or otherwise sensitive data — new fields, new exports, new logging;
  - infrastructure, IaC, IAM, network, and deployment configuration;
  - file upload, download, parsing, and rendering;
  - dependency additions, upgrades of security-relevant packages, and lockfile changes;
  - externally exposed APIs, webhooks, and anything reachable without authentication;
  - AI-generated code in any of the above areas ([§24](#24-ai-assisted-software-development)).
- Reviewers SHOULD run through a short security checklist on those changes: Is input validated server-side? Is authorization checked per object? Are secrets absent? Are errors safe? Is anything sensitive logged? Are new dependencies justified? Are the failure paths handled?
- A security concern raised in review SHOULD be resolved before merge, or explicitly deferred with a tracked issue and the project lead's agreement. It MUST NOT be silently dismissed.
- Automated checks (linters, static analysis, dependency and secret scanning) support review but do not replace it; a clean pipeline is not an approval.

> This standard does not specify a fixed number of required approvals, because approval requirements are configured per repository and some Cognativ work happens in client-owned repositories with their own rules. The requirement is that changes are reviewed and that security-sensitive changes get proportionate scrutiny.

---

## 16. Testing

- Security-relevant behavior MUST be covered by automated tests where the project has a test suite. "It worked when I tried it in the browser" is not coverage.
- **Negative tests are required, not optional.** For each security control, test that it *denies* as well as that it permits.
- **Authorization tests** MUST cover: unauthenticated access to a protected route; an authenticated user accessing another user's or another tenant's object; a standard user attempting a privileged action; and attempts to write protected fields (role, owner, tenant, price, status) through update endpoints.
- **Validation tests** SHOULD cover boundary values, wrong types, missing required fields, oversized input, and unexpected extra fields.
- **Failure-path tests** SHOULD cover what happens when a dependency, token verification, or permission lookup fails — asserting that the system denies rather than proceeds.
- **Regression tests MUST be added for every fixed vulnerability**, reproducing the original issue so it cannot return silently. Reference the issue in the test.
- Integration tests SHOULD exercise security controls through the real boundary (HTTP layer, middleware, policies) rather than only unit-testing the check in isolation, since bypasses usually come from a route that forgot to apply the check.
- Tests MUST NOT contain real credentials or real personal data ([§8](#8-secrets-management), [§9](#9-sensitive-data-and-pii)).
- Tests MUST NOT be disabled or weakened to make a pipeline pass. A skipped security test MUST have a tracked issue.

---

## 17. CI/CD security

Cognativ works across its own and client-owned infrastructure, so this section separates the **required capability** from the **implementation**, which is a per-project decision.

### Required capabilities

Each project SHOULD have, and any project handling sensitive data or exposed to the internet MUST have, a documented answer for each of the following:

| Capability | Requirement | Notes |
| --- | --- | --- |
| Automated tests on every change | MUST run on pull requests where a test suite exists | Failing tests block merge |
| Static analysis / linting | SHOULD run automatically | Language-appropriate linter or SAST |
| Dependency vulnerability scanning | SHOULD run on every change and on a schedule | See [§13](#13-dependency-and-supply-chain-security) |
| Secret scanning | SHOULD run on the repository and on pull requests | Push protection where available |
| Container/image or IaC scanning | SHOULD run where containers or IaC are used | |
| Protected deployment path | MUST — production deploys happen through the pipeline from a protected branch | No manual deploys from a laptop |
| Environment separation | MUST — separate credentials and resources per environment | See [§19](#19-secure-configuration) |
| Controlled production deployment | MUST be traceable to a specific commit and an identifiable person | Approval gates where the project requires them |

Where a required capability is unavailable on a client platform, the gap MUST be recorded with the project lead rather than assumed to be acceptable.

### Pipeline hygiene

- Pipeline secrets MUST live in the CI system's secret store, MUST be scoped to the environments that need them, and MUST NOT be echoed into build logs.
- Build logs MUST be treated as potentially readable by everyone with repository access: no secrets, no personal data.
- Pipelines MUST NOT expose production credentials to untrusted contexts, including workflows triggered by pull requests from forks.
- CI/CD service accounts and deployment credentials MUST be least-privileged and scoped to a single environment.
- Third-party pipeline actions/plugins SHOULD be pinned to a specific version or commit rather than a mutable tag, and SHOULD come from trusted publishers.
- Production deployment MUST be revertible, and the pipeline SHOULD record what was deployed, from which commit, by whom, and when.

> Cognativ does not claim that any specific product — for example Dependabot, CodeQL, GitHub Advanced Security, SonarQube/SonarCloud, Snyk, or Trivy — is enabled on every repository. Which tool implements which capability is recorded per project by the project lead. Some Cognativ engineers use SonarLint/SonarQube for IDE locally ([onboarding/developer-setup.md](../onboarding/developer-setup.md)); that is local tooling, not an organization-wide gate.

---

## 18. Vulnerability management

This is the engineering workflow for handling a vulnerability, whether it came from a scanner, a code review, a client report, a penetration test, or an engineer noticing something.

1. **Identify and record.** Create a tracked issue in the project's tracker. Do not leave it in a chat message. If the vulnerability is sensitive, restrict visibility of the issue rather than describing exploitation details in a public channel.
2. **Assess.** Determine what is actually affected, whether it is reachable in a deployed environment, what data or capability is at risk, and whether there is evidence of exploitation. Note that scanner severity is a starting point, not a verdict — a "critical" in a dev-only dependency may be lower risk than a "medium" in an internet-facing authorization path.
3. **Prioritize.** Rank by real risk: exposure (internet-facing vs internal), data sensitivity, exploit availability, and blast radius. Critical and high-severity issues in production systems MUST take priority over feature work and MUST be raised to the project lead immediately rather than queued silently. Where a client contract or project policy defines remediation timeframes, those timeframes apply.
4. **Remediate.** Fix the root cause. Patch or upgrade the affected component; where an upgrade is not immediately possible, apply a documented compensating control (disable the feature, add a gateway rule, restrict access) and keep the issue open until the real fix lands.
5. **Test.** Add a regression test that reproduces the original issue ([§16](#16-testing)) and verify the fix in a non-production environment.
6. **Deploy.** Ship through the normal protected pipeline. Genuine emergencies MAY use the hotfix path in [standards/git.md](../standards/git.md), but MUST still be reviewed after the fact and MUST NOT skip the pull request record.
7. **Document and close.** Record what was wrong, what was changed, and what was verified in the issue and pull request. Where credentials or data may have been exposed, follow [§26](#26-security-incidents).

Additional expectations:

- Known-vulnerable dependencies MUST NOT be introduced into new code when a fixed version exists.
- Vulnerability findings MUST NOT be closed as "won't fix" by the engineer alone; that decision belongs with the project lead, with the reasoning recorded.
- Recurring classes of issue SHOULD produce a durable fix — a lint rule, a shared helper, a test, or an update to this handbook — not just a one-off patch.

> Cognativ does not publish organization-wide numeric remediation SLAs in this handbook. Timeframes are set by client contracts and project agreements; where none exist, critical and high-severity issues are treated as immediate work.

---

## 19. Secure configuration

- Production configuration MUST NOT rely on development defaults: default admin accounts, sample keys, default database passwords, permissive CORS, seeded test users, or framework starter secrets MUST be removed or replaced before a system is exposed.
- **Debug mode MUST be off in production**, along with verbose error pages, profiling endpoints, GraphQL introspection where the product does not require it, directory listing, and framework debug toolbars.
- Unnecessary services, ports, endpoints, sample applications, and admin interfaces MUST be disabled or restricted. Administrative interfaces SHOULD be limited by network and by role.
- **Security headers** SHOULD be configured for production web applications: `Content-Security-Policy` (avoid `unsafe-inline`/`unsafe-eval`; prefer nonces or hashes), `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, a restrictive `Permissions-Policy`, and appropriate framing protection (`frame-ancestors` in CSP).
- **CORS MUST be restrictive.** Allowed origins MUST be an explicit list; `*` MUST NOT be combined with credentialed requests; allowed methods and headers SHOULD be limited to what the client actually needs. Reflecting the request's `Origin` header is not an allow-list.
- **Rate limiting and abuse protection** SHOULD be applied to authentication endpoints, password reset, registration, search, export, and any expensive or externally reachable operation. Authentication endpoints MUST have some form of brute-force protection.
- **Environment isolation MUST be real:** separate credentials, separate data stores, separate cloud accounts or projects where practical, and no network path from a lower environment into production data.
- Configuration MUST come from the environment or a secret store, not from committed environment-specific files ([§8](#8-secrets-management)). Configuration that controls security behavior SHOULD fail closed when missing — the application should refuse to start rather than silently defaulting to insecure behavior.
- Cloud and IAM configuration follows [§23](#23-infrastructure-and-cloud-security). Infrastructure-as-code MUST be reviewed like application code.

---

## 20. API security

- Every endpoint MUST have a deliberate authentication and authorization decision, including "public by design", which MUST be an explicit choice visible in review.
- Requests MUST be validated against a schema ([§6](#6-input-validation-and-output-encoding)): known fields, correct types, bounded sizes. Unknown fields SHOULD be rejected rather than passed through to a model.
- **Resource ownership MUST be validated on every request that names a resource** ([§5](#5-authorization)). Sequential or guessable identifiers are common; opaque identifiers reduce enumeration but MUST NOT be treated as an authorization control.
- Collection endpoints MUST be scoped to what the caller may see, and MUST paginate with an enforced maximum page size. Filtering, sorting, and field-selection parameters MUST be mapped from an allow-list, never passed to the database as raw expressions.
- Responses MUST return the minimum data required. Serializers MUST be explicit about which fields are exposed; do not return whole database rows or internal objects. Password hashes, tokens, internal flags, and other users' data MUST never appear in a response.
- Errors MUST follow [§11](#11-error-handling): no stack traces, no SQL, no internal hostnames, no framework version disclosure. Verbose `Server`/`X-Powered-By` headers SHOULD be removed.
- **Rate limiting** SHOULD be applied per principal and per IP, and MUST be applied where an endpoint is expensive, sensitive, or unauthenticated.
- **Replay and idempotency:** state-changing operations that a client may retry (payments, order creation, webhook processing) SHOULD support an idempotency key so a retry cannot double-apply. Inbound webhooks MUST verify the provider's signature, SHOULD reject stale timestamps, and SHOULD guard against replayed deliveries.
- API versioning SHOULD be explicit, and deprecated versions with weaker security behavior MUST be retired rather than left running.
- Machine-to-machine access MUST use scoped, revocable credentials ([§4](#4-authentication)) — not a shared master key.
- Public API documentation MUST NOT expose internal endpoints, debug operations, or credentials in examples.

---

## 21. Frontend security

The frontend improves user experience; it is never the security boundary. Every rule below assumes the server independently enforces authentication, authorization, and validation.

- **XSS:** rely on the framework's automatic escaping. In React/Next.js, `dangerouslySetInnerHTML` MUST NOT receive untrusted content; where untrusted HTML must be rendered, it MUST be sanitized with a maintained sanitizer (for example DOMPurify) first. Direct DOM sinks (`innerHTML`, `outerHTML`, `document.write`, `insertAdjacentHTML`) MUST NOT be fed untrusted data.
- `eval`, `new Function`, and string-valued `setTimeout`/`setInterval` MUST NOT be used with data derived from user input, URLs, or `postMessage`.
- User-controlled URLs MUST be validated before use in `href`, `src`, or a redirect. `javascript:` and `data:` schemes MUST be rejected. External links opened in a new tab SHOULD use `rel="noopener noreferrer"`.
- **CSP** SHOULD be configured for production web applications, avoiding `unsafe-inline` and `unsafe-eval`; use nonces or hashes for necessary inline scripts ([§19](#19-secure-configuration)).
- **Browser storage:** `localStorage` and `sessionStorage` are readable by any script on the origin. Long-lived credentials, refresh tokens, and personal data SHOULD NOT be stored there. Prefer `HttpOnly`, `Secure`, `SameSite` cookies for session material.
- **Sensitive state:** secrets MUST NOT be embedded in frontend bundles or public build-time variables (`NEXT_PUBLIC_*` and equivalents are public). Server-side data fetching MUST NOT over-fetch and serialize sensitive fields into the client payload, hydration state, or server-rendered HTML.
- **CSRF:** cookie-authenticated state-changing requests MUST be protected — framework CSRF tokens and/or `SameSite` cookies plus origin checking. Token-in-header authentication is not automatically CSRF-exempt if the token is also sent as a cookie.
- **Cookies** MUST be set `Secure` and `HttpOnly` (for session material) with an appropriate `SameSite` value and the narrowest practical scope.
- Client-side route guards, hidden UI, and role checks in the bundle are UX only. The corresponding server-side check MUST exist.
- Third-party scripts (analytics, tag managers, widgets, chat) execute with full access to the page. Each one MUST be justified, SHOULD be loaded from a trusted origin with integrity controls where available, and MUST NOT be added to pages handling credentials or sensitive data without review.
- Client-side logging and error reporting MUST NOT ship tokens or personal data to third-party services ([§25](#25-third-party-services-and-apis)).

See also [standards/react.md](../standards/react.md).

---

## 22. File handling

Uploads are a common path to remote code execution, stored XSS, and data leakage. Treat every uploaded file as hostile.

- **Type validation:** the accepted set MUST be an allow-list of formats the feature needs. Validation MUST NOT rely on the client-supplied filename extension or `Content-Type` alone; where practical, the file's actual content (magic bytes / parser acceptance) MUST be checked and the stored content type MUST be derived server-side.
- **Size and quantity limits** MUST be enforced server-side: per-file size, request size, file count, and where relevant per-account quota. Decompression and image/PDF processing MUST be bounded (dimensions, page count, time, memory) to resist zip-bomb and decompression-bomb style abuse.
- **Names and paths:** stored filenames SHOULD be server-generated identifiers. User-supplied names MUST NOT be used to build a filesystem path or storage key without normalization and validation against traversal ([§6](#6-input-validation-and-output-encoding)); keep the original name as metadata for display only, escaped on output.
- **Storage:** uploads MUST be stored outside the web root or in object storage that is **private by default**. Public buckets and world-readable directories MUST be a deliberate, reviewed decision. Uploaded files MUST NOT be stored in a location where the server may execute them.
- **Serving:** downloads SHOULD be delivered through short-lived signed URLs or an authorized application route that re-checks permissions on every request. Responses MUST set a correct `Content-Type` and `X-Content-Type-Options: nosniff`, and SHOULD use `Content-Disposition: attachment` for anything not intended to render inline. User-uploaded HTML/SVG MUST NOT be served inline from an origin that holds session cookies.
- **Content processing:** parsers and converters (image, PDF, office, archive, XML) MUST run on untrusted input with untrusted-input settings — XML parsing MUST disable external entities and DTD resolution (XXE), and SVG MUST be sanitized or rasterized before display. Heavy processing SHOULD be sandboxed or isolated from the application process.
- **Malware scanning** MUST be applied where project risk or client requirements call for it — for example files shared between users, or uploads in regulated environments. Where no scanning exists and files are shared between users, the risk MUST be raised with the project lead.
- Access to stored files MUST follow the same authorization rules as the records they belong to, including for exports, thumbnails, and derived files.

---

## 23. Infrastructure and cloud security

These requirements are provider-agnostic. Cognativ projects run on different clouds and on client-owned infrastructure, so the specific service that implements each control is a per-project decision; examples given here are illustrative, not mandates.

- **Least-privilege IAM.** Human and machine identities MUST have only the permissions they need, scoped by resource and environment. Wildcard administrative policies (`*` on `*`) MUST NOT be attached to application or CI identities. Permissions SHOULD be reviewed when a role changes.
- **No shared credentials.** Each person and each workload MUST have its own identity. Shared logins, shared root/owner accounts, and shared long-lived access keys are prohibited. Long-lived static keys SHOULD be replaced by short-lived, federated credentials where the platform supports it (for example instance roles, workload identity, OIDC federation from CI).
- **MFA for privileged access** MUST be enabled on cloud consoles, root/owner accounts, secret stores, and CI/CD administration. Root/owner accounts MUST NOT be used for day-to-day work.
- **Private networking by default.** Databases, caches, queues, and internal services MUST NOT be publicly reachable unless the product genuinely requires it. Prefer private subnets and private endpoints, with access through a bastion, VPN, or an identity-aware proxy.
- **Firewalls and security groups** MUST be restrictive: specific ports, specific sources. `0.0.0.0/0` on management ports (SSH, RDP, database ports) is prohibited.
- **Logging.** Control-plane/audit logging MUST be enabled for production environments, along with application and access logs ([§12](#12-security-logging-and-monitoring)). Logs SHOULD be retained beyond the incident-detection window and protected from tampering.
- **Encryption.** Encryption at rest MUST be enabled on managed data stores, object storage, volumes, and backups; TLS MUST be enforced for traffic ([§10](#10-encryption)). Keys SHOULD be managed by the platform's KMS.
- **Backups** MUST exist for production data, MUST be encrypted, SHOULD be tested by restore, and SHOULD be isolated from the credentials that can delete the primary data.
- **Environment separation** MUST be enforced with separate accounts/projects/subscriptions where practical, and at minimum separate credentials, networks, and data stores. Production MUST NOT be reachable from a lower environment's credentials.
- **Controlled production access.** Interactive production access MUST be limited to those who need it, SHOULD be time-boxed and requested rather than standing, SHOULD be logged, and MUST NOT be used to make undocumented changes. Production changes go through the pipeline ([§17](#17-cicd-security)).
- **Infrastructure as code.** Infrastructure SHOULD be defined as code, reviewed like application code, and scanned where tooling exists. Manual console changes SHOULD be reconciled back into code rather than left as drift.
- Public storage exposure MUST be a deliberate, reviewed decision; buckets and blobs default to private.

---

## 24. AI-assisted software development

Cognativ engineers use AI coding assistants and repository-aware agents. They are legitimate tools, and they do not change who is accountable: **the engineer who submits the code owns it.**

### Prohibited

Engineers MUST NOT:

- paste production secrets, API keys, tokens, private keys, or credentials into an AI tool, prompt, or agent configuration;
- send customer PII, protected data, or client-confidential material to an AI service that has not been approved for that project — including proprietary source code where the client's contract restricts it;
- merge generated code they do not understand, or that they have not reviewed line by line;
- assume generated code is secure, current, or correct — models produce plausible code with real vulnerabilities (missing authorization checks, string-concatenated SQL, weak crypto, outdated APIs, hallucinated packages);
- let an agent bypass required controls: pushing directly to protected branches, skipping code review, disabling or weakening tests, using `--no-verify`, or ignoring failing security checks;
- commit AI-generated dependency additions without verifying the package actually exists, is the intended one, and is maintained ([§13](#13-dependency-and-supply-chain-security)) — hallucinated or lookalike package names are a real supply-chain risk.

### Required

- AI-generated code MUST be reviewed to the same standard as human-written code — by the author before submitting, and by a reviewer in the pull request. Security-sensitive AI-generated changes fall under [§15](#15-code-review).
- Engineers MUST verify that generated code follows this standard: server-side validation, per-object authorization, parameterized queries, no secrets, safe error handling, no sensitive logging.
- Only AI tools approved for the project MAY be used with project code. Where a client restricts AI tooling or data sharing, that restriction is authoritative. Engineers MUST ask the project lead when unsure rather than assuming a tool is acceptable.
- Prompts and shared context (rules files, agent instructions, pasted snippets) MUST be treated as data that leaves the machine: no secrets, no personal data, no client-confidential material.
- Where a repository provides machine-readable engineering guidance (for example an `AGENTS.md`, `CLAUDE.md`, or a project rules directory), engineers SHOULD keep it aligned with this standard so that agents inherit the same requirements. Such guidance is an aid, not a control — it does not replace review.
- Engineers SHOULD treat agent-executed actions (running commands, editing many files, touching infrastructure) with the same caution as doing it themselves, and SHOULD keep destructive or credential-bearing operations under human control.
- Content generated by AI tools MUST NOT be attributed to a person who did not write it, and MUST NOT be represented as reviewed when it has not been.

---

## 25. Third-party services and APIs

Before integrating an external service — analytics, email, payments, storage, AI, monitoring, chat, feature flags, a data enrichment API — engineers MUST consider and be able to answer:

- **What data leaves our system?** Which fields, at what volume, how often. Personal data, credentials, and client-confidential content require explicit justification.
- **Is it permitted to leave?** Client contracts, data-processing terms, and regulatory requirements may prohibit sending certain data to certain vendors or regions. When in doubt, ask the project lead before sending anything.
- **Retention and reuse.** How long does the vendor retain the data, is it used for model training or product improvement, and can it be deleted on request?
- **Vendor security posture.** Is the vendor reputable and maintained? Does it offer TLS, scoped credentials, audit logging, and a documented security program?
- **Credentials and scopes.** Credentials MUST be secrets ([§8](#8-secrets-management)), MUST be per-environment, and MUST be requested with the **narrowest scope** that works — read-only where reading is all that is needed.
- **Failure behavior.** What happens when the vendor is slow, down, or returns garbage? Calls MUST have timeouts, SHOULD have retries with backoff, and MUST fail securely ([§11](#11-error-handling)) — a monitoring outage must not open access.
- **Data returned.** Third-party responses are untrusted input: validate them ([§6](#6-input-validation-and-output-encoding)) and never render them as raw HTML.
- **Webhooks in.** Inbound webhooks MUST verify signatures and MUST NOT be treated as authenticated just because they hit a hard-to-guess URL ([§20](#20-api-security)).
- **Exit.** Can the integration be removed and its credentials revoked without breaking the product? Note where the vendor becomes a single point of failure.

Adding or materially expanding a third-party integration SHOULD be raised with the project lead, not decided silently in a pull request.

---

## 26. Security incidents

This section covers what an **engineer** does on discovering or suspecting a security problem: a leaked credential, unauthorized access, exposed data, a live exploitation attempt, a lost or compromised device, or a serious vulnerability in production.

1. **Stop further exposure.** Take the immediate action that reduces harm: revoke the credential, disable the endpoint or feature flag, restrict access, take the affected component offline if that is the proportionate step. Prefer reversible containment over destructive action.
2. **Do not hide it.** Quietly patching, force-pushing history, deleting logs, or "fixing it before anyone notices" is not acceptable. Reporting a mistake is expected behavior; concealing it is the actual problem.
3. **Notify promptly.** Escalate to the project lead and engineering leadership immediately — and, where a client's system or data is involved, follow that client's incident-reporting requirements. Use a channel appropriate to the sensitivity; do not describe exploitable detail in a wide public channel. If you are unsure who to contact, escalate to your project lead and to engineering leadership at the same time rather than waiting.
4. **Rotate compromised credentials.** Any secret that may have been exposed MUST be rotated or revoked, not just removed from the code ([§8](#8-secrets-management)). Include downstream credentials the exposed one could reach.
5. **Preserve evidence.** Keep logs, access records, the offending commit reference, timestamps, and affected identifiers. Do not delete or rewrite them. Note what you did and when — a short timeline written while it is fresh is valuable later.
6. **Remediate the root cause.** Fix the underlying weakness, add a regression test ([§16](#16-testing)), and follow the vulnerability-management workflow ([§18](#18-vulnerability-management)).
7. **Follow up.** Once contained, record what happened and what changed so the same class of issue is prevented. Where notification obligations may apply (client, regulator, affected individuals), that determination belongs to Cognativ leadership and the client — engineers MUST NOT make or communicate that decision unilaterally.

> Cognativ's formal incident-response and breach-notification procedures, where they exist, are maintained outside this handbook as company operational policy. This section defines only the engineer's immediate obligations and does not substitute for that process.

---

## 27. Exceptions

Where a project cannot meet a requirement in this standard:

1. Raise it with the project lead before shipping, not after.
2. Record what cannot be met, why, the resulting risk, and any compensating control — in the pull request, the project's documentation, or a tracked issue, whichever the project uses as its record.
3. Where the gap is a platform limitation on a client-owned system, note it so it can be raised with the client.

An undocumented deviation is a defect. A documented, accepted risk is a decision.

---

## Related documents

- [Cognativ Developers Handbook — index](../README.md)
- [Onboarding and secure-development training](../onboarding/README.md)
- [HECVAT and compliance mapping](compliance-mapping.md)
- [Code review](../standards/code-review.md)
- [Git guidelines](../standards/git.md)
- [General programming best practices](../standards/general-best-practices.md)
- [SQL and PostgreSQL](../standards/sql.md)
- [React and Next.js](../standards/react.md)
- [Developer setup](../onboarding/developer-setup.md)

## Changes to this standard

Proposed changes follow the normal Cognativ workflow: a branch, a pull request against this repository, and review by engineering leadership. Material changes SHOULD update the **Last reviewed** date and version in the metadata table at the top of this document.
