# Developer onboarding and secure-development training

| Field | Value |
| --- | --- |
| Document | Developer onboarding and secure-development training path |
| Owner | Engineering |
| Applies to | All engineers joining a Cognativ project (employees, contractors, partners) |
| Status | Active |
| Last reviewed | 2026-08-24 |
| Review cadence | At least annually, and when onboarding practice changes |

This document explains how the [Cognativ Developers Handbook](../README.md) is used when an engineer joins Cognativ or joins a new project, and what is expected of them afterward. It also states clearly what this repository can and cannot evidence about training.

---

## 1. What every Cognativ engineer is expected to do

1. **Receive access to the Developers Handbook.** Access to this repository is part of engineering onboarding. An engineer who has not been given access should ask their project lead.
2. **Read the relevant engineering standards.** At minimum the [handbook index](../README.md), [Git guidelines](../standards/git.md), [code review](../standards/code-review.md), and [general programming best practices](../standards/general-best-practices.md), plus the standards for the technologies on their project.
3. **Read and understand the [Secure Software Development Standard](../security/secure-software-development-standard.md).** This is required, not optional, and it applies regardless of role or stack.
4. **Follow project-specific engineering and security rules.** Where a client or project imposes stricter requirements, those take precedence over the Cognativ baseline ([standard §1](../security/secure-software-development-standard.md#1-scope-and-purpose)).
5. **Ask when requirements are unclear.** If a security requirement seems impossible, ambiguous, or in conflict with a project constraint, raise it with the project lead before shipping. Asking is expected behavior; guessing is not.
6. **Keep applying these rules.** The standards apply continuously — while implementing, while reviewing others' code, and while operating what has shipped. They are not a one-time reading exercise.

---

## 2. Suggested onboarding sequence

A practical order for a new engineer's first days. Project leads may adapt it, but steps 1–4 should not be skipped.

| Step | Activity | Reference |
| --- | --- | --- |
| 1 | Get repository and tooling access; set up the development environment | [Developer setup](developer-setup.md) |
| 2 | Read the handbook index and understand how the standards fit together | [Handbook index](../README.md) |
| 3 | Read the Secure Software Development Standard end to end | [Secure Software Development Standard](../security/secure-software-development-standard.md) |
| 4 | Read the Git and code review standards; understand the pull request workflow | [Git](../standards/git.md), [Code review](../standards/code-review.md) |
| 5 | Read the standards for the project's stack | [`standards/`](../standards/) |
| 6 | Review the project's own documentation, README, and any repository-level engineering rules | Project repository |
| 7 | Confirm with the project lead what applies specifically to this project: secret management mechanism, environments, deployment path, data sensitivity, client restrictions | Project lead |
| 8 | Have the first pull requests reviewed with the standards explicitly in mind | [Code review](../standards/code-review.md) |

### Sections of the security standard to prioritize by role

All engineers should read the whole standard. If time is short on day one, start here:

- **Backend / API:** [§4 Authentication](../security/secure-software-development-standard.md#4-authentication), [§5 Authorization](../security/secure-software-development-standard.md#5-authorization), [§6 Input validation](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding), [§7 Database](../security/secure-software-development-standard.md#7-database-security), [§20 API](../security/secure-software-development-standard.md#20-api-security)
- **Frontend:** [§21 Frontend](../security/secure-software-development-standard.md#21-frontend-security), [§5 Authorization](../security/secure-software-development-standard.md#5-authorization), [§6 Input validation](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding)
- **Everyone:** [§8 Secrets](../security/secure-software-development-standard.md#8-secrets-management), [§9 Sensitive data and PII](../security/secure-software-development-standard.md#9-sensitive-data-and-pii), [§13 Dependencies](../security/secure-software-development-standard.md#13-dependency-and-supply-chain-security), [§24 AI-assisted development](../security/secure-software-development-standard.md#24-ai-assisted-software-development), [§26 Security incidents](../security/secure-software-development-standard.md#26-security-incidents)
- **Infrastructure / DevOps:** [§17 CI/CD](../security/secure-software-development-standard.md#17-cicd-security), [§19 Secure configuration](../security/secure-software-development-standard.md#19-secure-configuration), [§23 Infrastructure and cloud](../security/secure-software-development-standard.md#23-infrastructure-and-cloud-security)

---

## 3. Project-level and machine-readable engineering guidance

The handbook is the company-wide baseline. Individual project repositories often carry their own guidance, and engineers should look for it before starting work:

- a project `README.md` and any `docs/` or `CONTRIBUTING.md`;
- environment and configuration documentation, including which secret mechanism the project uses;
- architecture or decision records where the project keeps them.

Some repositories additionally carry **machine-readable guidance intended for AI coding assistants and agents**, for example a file such as `AGENTS.md` or `CLAUDE.md`, or a rules directory such as `.agents/` or `.rules/`. These are examples of formats in common use rather than a required Cognativ structure — check what the specific repository actually contains.

Where such files exist:

- they SHOULD reflect, not contradict, the [Secure Software Development Standard](../security/secure-software-development-standard.md);
- they are guidance for tooling, not a security control — they do not replace human review ([standard §24](../security/secure-software-development-standard.md#24-ai-assisted-software-development));
- an engineer who notices that repository guidance conflicts with this handbook should raise it and get the repository guidance corrected.

---

## 4. Continuous expectations after onboarding

- Apply the standards in every pull request you author.
- Apply the standards when reviewing others' work — reviewers are part of the control ([standard §15](../security/secure-software-development-standard.md#15-code-review)).
- Re-read the security standard when you start work in an unfamiliar area (first time touching authentication, payments, file uploads, IaC).
- Treat handbook updates as required reading. Material changes to the security standard are announced through normal engineering channels and are visible in this repository's Git history.
- Propose improvements. If the standard is wrong, outdated, or unclear, open a pull request against this repository. Practitioners keep it accurate.

---

## 5. Training evidence — what this handbook does and does not prove

This section exists so that no one over-states what the repository demonstrates.

### What the handbook can legitimately evidence

- **Documented secure-coding guidance.** A specific, maintained, dated secure development standard rather than generic advice.
- **Security training and onboarding material.** Content that is used to introduce engineers to Cognativ's secure-development requirements, with a defined onboarding path (sections 1–2 above).
- **Documented engineering expectations.** That Cognativ has told its engineers, in writing, what they must do — and that these expectations are reinforced through mandatory code review.
- **Maintenance and ownership.** An owner, a status, a last-reviewed date, a review cadence, and a full change history in Git.

### What the handbook cannot evidence

This repository **does not** prove that any specific individual read, understood, completed, or acknowledged this material. It contains no per-person records, and none should be inferred from it.

Individual completion evidence — where an assessor requires person-level proof — is maintained separately through Cognativ's operational processes, which may include onboarding records, written acknowledgement records, a learning-management system, or HR records. This handbook makes no claim about which of those exist; that information must come from the process owner.

Cognativ also makes no claim of formal security certification on the basis of this handbook.

See [security/compliance-mapping.md](../security/compliance-mapping.md) for the full mapping of what is and is not evidenced here.

---

## Related documents

- [Cognativ Developers Handbook — index](../README.md)
- [Secure Software Development Standard](../security/secure-software-development-standard.md)
- [Compliance mapping](../security/compliance-mapping.md)
- [Developer setup](developer-setup.md)
- [Code review](../standards/code-review.md)
- [Git guidelines](../standards/git.md)
