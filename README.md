# Cognativ Developers Handbook

| Field | Value |
| --- | --- |
| Owner | Engineering |
| Applies to | Cognativ software engineering (employees, contractors, and partner engineers) |
| Status | Active |
| Last reviewed | 2026-08-24 |
| Review cadence | At least annually, and after significant engineering, security, or process changes |
| This version | <https://github.com/cognativ-inc/developers-handbook/> |
| Editor | Diego Martin Lafuente — <diego.lafuente@cognativinc.com> |

Copyright © Cognativ Inc.

---

## About this handbook

This is the engineering handbook for Cognativ's developers. All Cognativ developers are expected to read it, learn it, and use it as their guide while building software at the company — and to keep applying it while implementing and reviewing code.

It sets out the conventions the team works by (branching, commit and pull request practice, code review, language and framework standards) and, most importantly, the **security requirements that apply to every Cognativ project**.

> **New to Cognativ?** Start with [Onboarding and secure-development training](onboarding/README.md).
>
> **Looking for the secure coding standard?** [Secure Software Development Standard](security/secure-software-development-standard.md).

---

## Contents

### Security

| Document | What it covers |
| --- | --- |
| [**Secure Software Development Standard**](security/secure-software-development-standard.md) | Cognativ's mandatory, technology-agnostic secure coding standard: principles, OWASP alignment, authentication, authorization, validation, database, secrets, PII, encryption, error handling, logging, dependencies, source control, code review, testing, CI/CD, vulnerability management, configuration, API, frontend, file handling, cloud, AI-assisted development, third parties, and incident handling. |
| [Compliance mapping](security/compliance-mapping.md) | How this repository maps to security-questionnaire evidence requests (HECVAT and similar), and what it deliberately does not claim. |

### Onboarding

| Document | What it covers |
| --- | --- |
| [Onboarding and secure-development training](onboarding/README.md) | How the handbook is used when an engineer joins, what they must read, and what the handbook does and does not evidence about training. |
| [Developer setup](onboarding/developer-setup.md) | Standard development environment, editor, extensions, and local tooling. |

### Engineering standards

| Document | What it covers |
| --- | --- |
| [Git guidelines](standards/git.md) | Branching model, branch naming, pull requests, merge practice, communication. |
| [Code review](standards/code-review.md) | Review process, review etiquette, and security review expectations. |
| [General programming best practices](standards/general-best-practices.md) | Language-agnostic practice: credentials, input validation, error handling, conditionals, dependency scanning. |
| [JavaScript](standards/javascript.md) | Variables, naming, comments, functions, concurrency, error handling. |
| [TypeScript](standards/typescript.md) | Types, interfaces, enums, access modifiers, strictness. |
| [React and Next.js](standards/react.md) | Component design, hooks, client/server boundaries, type safety. |
| [C#](standards/csharp.md) | Style, formatting, naming conventions, language usage, flow control. |
| [SQL and PostgreSQL](standards/sql.md) | Query practice, indexing, transactions, and SQL security. |

---

## How the documents relate

- The **Secure Software Development Standard** is the security baseline for all Cognativ engineering. It is normative — it uses MUST / SHOULD / MAY deliberately.
- The **engineering standards** cover craft, style, and process. Where they touch security, the Secure Software Development Standard governs.
- Where a client or project imposes **stricter** requirements, those take precedence. Precedence is defined in [standard §1](security/secure-software-development-standard.md#1-scope-and-purpose).
- Project repositories may carry their own additional guidance; see [onboarding §3](onboarding/README.md#3-project-level-and-machine-readable-engineering-guidance).

## Requirement language

Used throughout the security standard, and in the engineering standards where a rule is normative:

- **MUST** — mandatory Cognativ engineering requirement.
- **SHOULD** — expected unless there is a documented reason otherwise.
- **MAY** — optional or context-dependent.

## Language

English is the default language for all code, comments, commit messages, pull requests, documentation, and team communication. See [Git guidelines](standards/git.md#communicate-effectively).

## Contributing to this handbook

The handbook is maintained by the engineers who use it. To propose a change:

1. Create a branch in this repository.
2. Open a pull request describing what is changing and why.
3. Request review from engineering leadership. Changes to the Secure Software Development Standard are reviewed by engineering leadership before merge.
4. Update the **Last reviewed** date and, for the security standard, the version, when the change is material.

Corrections, clarifications, and updates for new tooling are welcome and expected. If something in here is outdated or wrong, fix it rather than working around it.
