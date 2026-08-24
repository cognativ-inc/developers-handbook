# Developer setup

| Field | Value |
| --- | --- |
| Owner | Engineering |
| Applies to | All Cognativ engineers |
| Status | Active |
| Last reviewed | 2026-08-24 |

At Cognativ we work across a wide range of technologies, and collaborating effectively means agreeing on a baseline set of tools. These standards apply on Windows, macOS, and Linux.

Related: [Onboarding and secure-development training](README.md) · [Secure Software Development Standard](../security/secure-software-development-standard.md)

---

## Editor

Our default editor is **Visual Studio Code**. Keep it on a current version — editors and their extensions receive security fixes like anything else.

Engineers may use a different editor or IDE where a stack calls for it (for example JetBrains tooling for .NET or Java), provided the project's formatting and linting configuration is respected. What must not vary is the output: formatting, lint rules, and analyzers come from the repository's configuration (`.editorconfig`, ESLint, Prettier, analyzer settings), not from personal editor preferences.

### Recommended extensions

For JavaScript/TypeScript and React/Next.js work:

1. **ESLint** — surfaces lint errors inline.
2. **Prettier** — code formatting.
3. **Error Lens** — shows diagnostics inline in the editor.
4. **Pretty TypeScript Errors** — makes TypeScript errors readable.
5. **SonarQube for IDE** (previously published as SonarLint) — local static analysis, including common security issues.
6. **Live Share** — pair programming and review sessions.
7. **Console Ninja** — enriched runtime logging while developing.
8. **Remove Unused Imports** — cleanup helper.

Install whatever additional extensions your stack needs — C# Dev Kit for .NET, Python tooling, Docker, Terraform, database clients.

**Extensions run with your user's privileges and can read your workspace.** Install from known publishers, check download counts and repository links, and remove extensions you no longer use. Do not install an extension just because an AI tool or a blog post suggested it.

---

## Git

Use the command line or the platform's desktop app, whichever you prefer — but be comfortable on the command line ([Git guidelines](../standards/git.md#using-the-command-line)).

Baseline setup:

- Configure your name and a Cognativ email address in your Git identity so commits are attributable.
- Enable **MFA** on the Git platform account you use for Cognativ work.
- Use SSH keys or a platform credential helper rather than storing passwords in plain text. Protect private keys with a passphrase.
- Keep your Git client current.

---

## Local security hygiene

These are the parts of the [Secure Software Development Standard](../security/secure-software-development-standard.md) that apply to your machine on day one.

- **Never commit secrets.** `.env` and equivalent files belong in `.gitignore`, always. Commit a `.env.example` with keys and placeholder values so the next engineer knows what to configure. See [standard §8](../security/secure-software-development-standard.md#8-secrets-management).
- **Use non-production credentials locally**, pointing at non-production resources. Production credentials must not exist on a developer machine.
- **Do not keep production data locally.** Use synthetic or anonymized fixtures ([standard §9](../security/secure-software-development-standard.md#9-sensitive-data-and-pii)).
- **Enable the project's pre-commit checks** if it provides them (linting, formatting, secret scanning), and do not bypass them with `--no-verify`.
- **Review your own diff before pushing.** `git diff --staged` catches most accidental secret and debug-code commits.
- **Keep your machine secure:** full-disk encryption enabled, screen lock on, operating system and browser patched, and a password manager instead of a notes file. A developer laptop holds credentials to client systems.
- **Be careful what you paste into AI tools.** No secrets, no customer data, no client-confidential code in unapproved services ([standard §24](../security/secure-software-development-standard.md#24-ai-assisted-software-development)).

---

## Per-project setup

Before writing code on a new project, confirm with the project lead:

- how to obtain configuration and secrets for local development, and which secret mechanism the project uses;
- which environments exist and which of them you have access to;
- how the project runs tests and what must pass before a pull request is opened;
- which AI tooling is approved for that project's code;
- whether the client imposes stricter engineering or security rules.
