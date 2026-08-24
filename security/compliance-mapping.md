# Compliance mapping — security questionnaire evidence

| Field | Value |
| --- | --- |
| Document | Compliance mapping for the Cognativ Developers Handbook |
| Owner | Engineering |
| Purpose | Map this repository to common security-questionnaire evidence requests (HECVAT and similar) |
| Status | Active |
| Last reviewed | 2026-08-24 |
| Review cadence | At least annually, and whenever the handbook structure changes |

This document exists so that engineering and whoever completes a security questionnaire (HECVAT, vendor security review, client due diligence) can point precisely at what this repository does and does not evidence.

It is deliberately conservative. Where this repository cannot substantiate a claim, it says so.

---

## 1. What this repository is

The [Cognativ Developers Handbook](../README.md) is the documented engineering standard that Cognativ developers are expected to read and apply. It contains:

- the [Secure Software Development Standard](secure-software-development-standard.md) — Cognativ's secure coding standard;
- the [onboarding and secure-development training path](../onboarding/README.md) — how that standard is introduced to engineers;
- language, framework, and process standards under [`standards/`](../standards/).

It is version-controlled in Git, so every change has an author, a timestamp, and a review record.

---

## 2. HECVAT: secure coding policy / standard

**Request (typical wording):** *"Do you have a documented secure coding policy or standard? Provide it."*

**Cognativ's answer:** Yes. Secure development requirements are documented, version-controlled, and mandatory for Cognativ engineers.

**Evidence in this repository:**

| Evidence | Location | What it demonstrates |
| --- | --- | --- |
| Secure Software Development Standard | [`security/secure-software-development-standard.md`](secure-software-development-standard.md) | A documented, technology-agnostic secure coding standard with defined scope, mandatory (MUST) requirements, an owner, a status, a last-reviewed date, and a review cadence. Covers OWASP alignment, authentication, authorization, input validation, database security, secrets, PII, encryption, error handling, logging, dependencies, source control, code review, testing, CI/CD, vulnerability management, configuration, API, frontend, file handling, cloud, AI-assisted development, third parties, and incident handling. |
| Developers Handbook index | [`README.md`](../README.md) | Establishes that the handbook applies to all Cognativ developers, that they are expected to read and apply it, and how the security standard relates to the rest of the engineering guidance. |
| Code review standard | [`standards/code-review.md`](../standards/code-review.md) | Documents that peer review is mandatory and that security-sensitive changes receive proportionate scrutiny. |
| Git and branching standard | [`standards/git.md`](../standards/git.md) | Documents change control: pull requests required, no direct commits to protected branches, branch naming and review workflow. |
| Language and platform standards | [`standards/`](../standards/) | Practical secure-coding guidance in the technologies Cognativ uses (general practices, SQL/PostgreSQL, JavaScript, TypeScript, React/Next.js, C#). |
| Version control history | Git history of this repository | Shows authorship, change dates, and review of the standard itself. |

**Supplemental evidence outside this repository:** project- or client-specific development and security rules where a project imposes stricter requirements ([standard §1](secure-software-development-standard.md#1-scope-and-purpose) defines the precedence order).

---

## 3. HECVAT: developer secure-coding training evidence

**Request (typical wording):** *"Are developers trained in secure coding practices? Provide evidence."*

**Cognativ's answer:** Cognativ maintains documented secure-development guidance that engineers are required to read and apply, and it is part of engineering onboarding. Evidence that a **specific individual** completed it is maintained separately from this repository (see [section 4](#4-what-this-repository-does-not-evidence)).

**Evidence in this repository:**

| Evidence | Location | What it demonstrates |
| --- | --- | --- |
| Onboarding and training path | [`onboarding/README.md`](../onboarding/README.md) | Documents that the handbook is provided to engineers during onboarding, that reading the Secure Software Development Standard is a required step, and what engineers are expected to do with it during implementation and review. |
| Secure Software Development Standard | [`security/secure-software-development-standard.md`](secure-software-development-standard.md) | The substantive secure-coding training and reference content itself. |
| Developer setup | [`onboarding/developer-setup.md`](../onboarding/developer-setup.md) | Documents the standard developer environment, including local code-quality and security tooling. |
| Ongoing reinforcement through review | [`standards/code-review.md`](../standards/code-review.md), [standard §15](secure-software-development-standard.md#15-code-review) | Demonstrates that secure-coding expectations are applied continuously in peer review, not only at hire. |

**What this legitimately supports:** that secure-coding training/reference material exists, is specific rather than generic, is maintained and dated, is mandatory for engineers, and is delivered as part of onboarding.

**What must be supplied alongside it if the assessor requires person-level proof:** employee-level completion or acknowledgement records. See [section 4](#4-what-this-repository-does-not-evidence).

---

## 4. What this repository does *not* evidence

This is stated explicitly so no one over-claims in a questionnaire response.

This repository **cannot** be used as evidence of:

| Not evidenced here | Where it would come from |
| --- | --- |
| That a named individual read, completed, or acknowledged the standard | Onboarding records, acknowledgement records, an LMS or training platform, HR records, or another company process — whichever Cognativ actually maintains |
| Training completion dates, scores, or certificates per employee | The same operational or HR process |
| Formal security certification (ISO 27001, SOC 2, or similar) | None is claimed. Cognativ does not represent this handbook as a certification |
| That a specific scanning tool (for example Dependabot, CodeQL, SonarQube, Snyk) is enabled on every repository | Per-project CI/CD configuration and screenshots or exports from the relevant platform |
| Actual scan results, penetration test reports, or vulnerability reports | The project's security tooling, security testing engagements, or vulnerability tracker |
| Branch protection, required approvals, or MFA enforcement as configured settings | Source-control platform configuration for the relevant organization or repository |
| Production access logs, deployment approvals, or IAM configuration | Cloud provider and CI/CD platform records |
| A formal incident-response plan with defined roles and notification timelines | Cognativ company operational policy, maintained outside this repository |
| Numeric remediation SLAs | Client contracts or a separate Cognativ policy, where one exists |

The Secure Software Development Standard defines the **required capability** in these areas and, where relevant, notes that the specific implementation is a per-project decision. It does not assert that any given tool or platform setting is universally in place.

---

## 5. How to use this in a questionnaire response

A defensible response pattern:

> Cognativ maintains a documented Secure Software Development Standard as part of its Developers Handbook, version-controlled at
> `https://github.com/cognativ-inc/developers-handbook/`. It defines mandatory secure-coding requirements for all Cognativ
> engineers, is aligned with OWASP guidance, is owned by Engineering, and is reviewed at least annually. The handbook is
> provided to engineers during onboarding, and reading the Secure Software Development Standard is a required onboarding step.
> Secure-coding expectations are reinforced continuously through mandatory peer code review. Individual completion records are
> maintained separately through [Cognativ's onboarding/HR process] and can be provided on request.

Complete the bracketed portion only with a process Cognativ actually operates. If no such record currently exists, say what is true — that the material is required reading and that formal per-person attestation is being implemented — rather than implying records exist.

---

## 6. Other questions this repository helps answer

These are not the two immediate requirements, but assessors commonly ask them and the standard already addresses them:

| Question area | Reference |
| --- | --- |
| Change management / code review | [standard §15](secure-software-development-standard.md#15-code-review), [`standards/code-review.md`](../standards/code-review.md), [`standards/git.md`](../standards/git.md) |
| Secrets and credential management | [standard §8](secure-software-development-standard.md#8-secrets-management) |
| Encryption in transit and at rest | [standard §10](secure-software-development-standard.md#10-encryption) |
| Authentication and MFA expectations | [standard §4](secure-software-development-standard.md#4-authentication) |
| Access control and least privilege | [standard §5](secure-software-development-standard.md#5-authorization), [standard §23](secure-software-development-standard.md#23-infrastructure-and-cloud-security) |
| Vulnerability management process | [standard §18](secure-software-development-standard.md#18-vulnerability-management) |
| Third-party / subprocessor data handling | [standard §25](secure-software-development-standard.md#25-third-party-services-and-apis) |
| Handling of personal / regulated data by developers | [standard §9](secure-software-development-standard.md#9-sensitive-data-and-pii) |
| Logging and auditability | [standard §12](secure-software-development-standard.md#12-security-logging-and-monitoring) |
| Secure use of AI coding tools | [standard §24](secure-software-development-standard.md#24-ai-assisted-software-development) |
| Developer obligations on suspected incident | [standard §26](secure-software-development-standard.md#26-security-incidents) |

---

## Related documents

- [Cognativ Developers Handbook — index](../README.md)
- [Secure Software Development Standard](secure-software-development-standard.md)
- [Onboarding and secure-development training](../onboarding/README.md)
