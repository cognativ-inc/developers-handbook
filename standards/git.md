# Git guidelines

| Field | Value |
| --- | --- |
| Owner | Engineering |
| Applies to | All Cognativ engineering work |
| Status | Active |
| Last reviewed | 2026-08-24 |

Git is an essential skill for every Cognativ developer, whether you work through GitHub, GitLab, Bitbucket, or the terminal. As the company grows, keeping our codebases stable depends on a predictable, shared workflow. This document defines that workflow.

Some projects are managed in a client's Git organization. In those cases the client's rules take precedence, and parts of this document may not apply. The guidance here reflects widely used industry practice, so it will usually align with a client's expectations anyway.

Related: [Code review](code-review.md) · [Secure Software Development Standard §14 Source-control security](../security/secure-software-development-standard.md#14-source-control-security)

---

## Contents

- [Communicate effectively](#communicate-effectively)
- [Repository and branch structure](#repository-and-branch-structure)
- [Pull requests](#pull-requests)
  - [Naming branches](#naming-branches)
  - [Creating pull requests](#creating-pull-requests)
  - [Merge often](#merge-often)
  - [Size of a branch](#size-of-a-branch)
  - [When working on features, avoid refactoring](#when-working-on-features-avoid-refactoring)
- [Code review](#code-review)
- [Security rules for source control](#security-rules-for-source-control)
- [Using the command line](#using-the-command-line)

---

## Communicate effectively

Good communication makes collaboration seamless. Let your team know when you start something, what you are working on, and when you are done, and respond to questions and review comments promptly.

English is the **default** language for all communication — chat channels, issue trackers, commit messages, pull requests, and code comments. As with the rest of this handbook, avoid using your local language; it excludes colleagues and clients.

Remember:

1. Always communicate in English.
2. Use neutral, factual language.
3. Be short and concise — no walls of text.

---

## Repository and branch structure

Many branching models exist, and the right one depends on the project. Cognativ's default is a **GitFlow-style workflow**, unless the repository belongs to a client or the project imposes a different architecture.

Two long-lived branches:

- **`main`** (called `master` in older repositories) — production-ready code. Only release and hotfix branches merge here.
- **`dev`** (also seen as `develop`) — the integration branch where features land and are reviewed.

> Naming note: newer repositories use `main` and `dev`; older ones use `master` and `develop`. Follow whatever the repository already uses rather than renaming branches mid-project, and check the repository's default branch before opening a pull request.

Short-lived branches:

- **`feature/*`** — new work. Branched from the integration branch, merged back after review.
- **`release/*`** — release preparation. Branched from the integration branch, merged into both the production and integration branches.
- **`hotfix/*`** — urgent production fixes. Branched from the production branch, merged into both the production and integration branches.

This separation keeps development and production code clearly divided and gives every change a consistent path to release.

Trunk-based development is an acceptable alternative where a project's delivery model calls for it; that is a project-level decision made with the project lead, not an individual one.

---

## Pull requests

> **Pull requests are mandatory. Never commit or push directly to long-lived branches** (`main`/`master`, `dev`/`develop`, release branches).

Pull requests let several people review and discuss a change before it lands, which keeps quality high and surfaces conflicts with other work early. Committing straight to an integration branch bypasses that, breaks the team's workflow, and causes conflicts for everyone with an open branch off it.

To keep merges clean:

- work on a feature branch, never directly on a shared branch;
- pull the latest changes from the target branch regularly;
- resolve conflicts in your branch, not in the shared one.

Where the platform allows it, long-lived branches should be protected so this is enforced rather than remembered — see [Security rules for source control](#security-rules-for-source-control).

### Naming branches

Meaningful branch names make a repository navigable.

Bad:

> `branch1`
>
> `feature/fixes`
>
> `refactor-old-code`

Good:

> `feature/new-login-ui`
>
> `feature/refactor-css-new-ui`
>
> `bug/fixed-margin-login-page`

Conventions:

- Feature branches: `feature/` — for example `feature/new-login-system`, `feature/voting-on-posts`.
- Bug fixes: `bug/` — for example `bug/login-form-validation`.
- Release branches: `release/` — for example `release/v1.2`. Merged into both the production and integration branches once released.
- Hotfix branches: `hotfix/` — for example `hotfix/login-issue`.

Where a project uses an issue tracker, including the ticket key is helpful: `feature/PROJ-142-new-login-ui`.

Following these conventions protects both the company's work and your colleagues' time. Skipping them puts both at risk.

### Creating pull requests

#### Writing good PR titles

Prefixes in the title tell the team what a pull request is and how urgently it needs attention.

- **`[WIP]` — Work in Progress.** Not ready for review or merge. Use it while a feature is still in flight so nobody reviews or merges it prematurely.
  > Ex. `[WIP] feature/Group management for group administrators`
- **`[RFC]` — Request for Comments.** A proposal for a new feature or change, opened to gather feedback before the work is finished.
- **`[BUG]` — Bug fix.** Resolves a specific defect.
- **`[HOTFIX]` — Hotfix.** A critical production fix that needs attention immediately.
- **`[IMPROVEMENT]`** — An enhancement to an existing feature rather than a new one.
- **`[PERFORMANCE]`** — Performance work.
- **`[DOCUMENTATION]`** — Documentation changes.
- **`[SECURITY]`** — A change that fixes a vulnerability or hardens a security control. Flag these so they get the right reviewer. For an unannounced vulnerability, keep exploitation details out of the public description and follow [standard §18](../security/secure-software-development-standard.md#18-vulnerability-management).

#### Writing good PR descriptions

A clear description is what makes a review fast. Include:

1. **A summary of the changes** — what a reviewer should expect to see.
2. **Context** — the background and reasoning behind the change.
3. **Screenshots or recordings** for visual changes. Make sure they contain no real customer data or credentials.
4. **Dependencies** — other changes or deployments this relies on.
5. **A link to the issue or ticket** the work belongs to.
6. **How you tested it**, and what testing is still outstanding.
7. **Technical context** — language, runtime, and framework versions where it affects how the code should be judged. This prevents avoidable review round-trips.
8. **A note on anything security-relevant** — authentication, authorization, data handling, dependencies, infrastructure — so the right reviewer picks it up. See [Code review](code-review.md#security-review).

Keep it clear and concise, and avoid jargon a reviewer outside the project would not follow.

### Merge often

Regularly bring the integration branch into your feature branch (merge or rebase, per the project's convention) so branches stay in sync and conflicts are resolved in small pieces rather than all at once at the end.

Do not rewrite history on a branch other people are working from ([standard §14](../security/secure-software-development-standard.md#14-source-control-security)).

### Size of a branch

Large branches produce long pull requests that nobody can review honestly. Think about the size of the feature before you start.

A registration feature might touch UI, interactions, copy, translations, state management, and error handling. Rather than one branch containing all of it, break it into smaller chunks that can each be reviewed properly. There is no fixed line count or commit count that says "split this" — judge by complexity and by whether a reviewer could realistically hold the whole change in their head.

### When working on features, avoid refactoring

Avoid significant refactors while building a new feature. Large-scale renaming or introducing new libraries mid-feature creates noise, hides the actual change, and risks breaking the application if not every call site is updated.

For example, while implementing a voting feature, do not simultaneously rename CSS classes across the application. Put the refactor on its own branch, get it reviewed separately, then rebase or merge it into the feature branch.

If you discover mid-feature that a refactor is needed: pause the feature, branch for the refactor, land that, then come back.

---

## Code review

Code review is mandatory at Cognativ. The process, etiquette, security checklist, and merge expectations are documented separately in **[Code review](code-review.md)**.

---

## Security rules for source control

Summarized from [standard §14](../security/secure-software-development-standard.md#14-source-control-security):

- **No secrets in Git.** No API keys, passwords, tokens, private keys, certificates, or connection strings — in any file, on any branch, in any commit. Review your diff before pushing. If a secret is committed, rotate it immediately and follow [standard §8](../security/secure-software-development-standard.md#8-secrets-management).
- **Maintain `.gitignore`.** Keep `.env` files, key and certificate files, local database dumps, and build output out of the repository. Commit a `.env.example` with placeholder values instead.
- **Enable MFA** on the Git platform account you use for Cognativ work.
- **Branch protection** should be enabled on long-lived branches where the platform and the repository owner allow it: no direct pushes, pull request required, required status checks where CI exists.
- **Least-privilege access.** Repository permissions match the role. Access — including personal access tokens, deploy keys, and SSH keys — is removed when someone leaves a project or the company.
- **Never push Cognativ or client code** to a personal repository, public fork, gist, or paste site.
- **Do not bypass local checks.** If a pre-commit hook or secret scanner blocks a commit, fix the problem rather than using `--no-verify`.

---

## Using the command line

Git UI applications are convenient, but working with Git from the terminal is an important skill — it is what you will have on a server, in a container, or in CI. GitLab's Git tutorial is thorough and worth reading: <https://docs.gitlab.com/topics/git/>
