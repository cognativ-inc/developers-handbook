# Code review

| Field | Value |
| --- | --- |
| Owner | Engineering |
| Applies to | All Cognativ engineering work |
| Status | Active |
| Last reviewed | 2026-08-24 |

Code review is how Cognativ keeps software quality and security consistent across projects. **Code review is mandatory**, and every developer is expected to participate — including on projects they are not assigned to. Catching issues in review is far cheaper than catching them in production.

Related: [Git guidelines](git.md) · [Secure Software Development Standard §15](../security/secure-software-development-standard.md#15-code-review)

---

## Contents

- [What a review is for](#what-a-review-is-for)
- [Author responsibilities](#author-responsibilities)
- [Reviewer responsibilities](#reviewer-responsibilities)
- [Security review](#security-review)
- [Requesting a review](#requesting-a-review)
- [Merging](#merging)
- [Review etiquette](#review-etiquette)

---

## What a review is for

A review should answer four questions:

1. **Does it work?** Does the change do what it claims, including on the failure paths?
2. **Is it safe?** Does it meet the [Secure Software Development Standard](../security/secure-software-development-standard.md)?
3. **Is it maintainable?** Will the next engineer understand it? Does it follow the project's conventions and this handbook's standards?
4. **Is it the right size and scope?** Is unrelated refactoring mixed in? Is it small enough to review honestly?

Automated checks (linters, tests, static analysis, dependency and secret scanning) support review. They do not replace it — a green pipeline is not an approval.

---

## Author responsibilities

- **Keep pull requests small and focused.** One concern per branch. Split large features into reviewable increments, and keep refactors in their own branch ([Git guidelines](git.md#when-working-on-features-avoid-refactoring)).
- **Review your own diff first.** Read every line before asking someone else to. Most review comments are things the author would have caught themselves.
- **Write a description a reviewer can act on** ([Git guidelines](git.md#writing-good-pr-descriptions)): what changed, why, how it was tested, what to look at closely, and any dependencies.
- **Flag security-relevant changes explicitly** in the description so they get the right reviewer. See [Security review](#security-review).
- **Confirm no secrets are in the diff.** Check for `.env` files, keys, tokens, connection strings, and real data in fixtures ([standard §8](../security/secure-software-development-standard.md#8-secrets-management)).
- **State the technical context** where it matters — language and runtime version, framework version, target environment — so reviewers can judge the code properly.
- **Respond to every comment**, even if the answer is "good catch, fixed" or "I chose this because…". Do not resolve a reviewer's concern silently.
- **Own AI-generated code.** If a tool wrote it, you still read it, understand it, and are accountable for it ([standard §24](../security/secure-software-development-standard.md#24-ai-assisted-software-development)).

---

## Reviewer responsibilities

- **Review promptly.** Review as soon as you reasonably can; a stale branch drifts and blocks its author. Reviewers in other timezones need a fair window, so plan for it rather than merging around it.
- **Actually read the code.** Approving means you looked. If you only skimmed part of it, say which part you reviewed.
- **Review for security, not just style** — see below.
- **Use automated tooling** to catch mechanical issues so human attention goes to logic, design, and security.
- **Explain your reasoning.** A comment that says what to change *and why* teaches; a comment that just says "change this" does not.
- **Provide references.** Link to the relevant handbook section, official documentation, or an article that explains the concern.
- **Ask about intent** before assuming a mistake: "why `let` here instead of `const`?" often reveals a constraint you did not know about.
- **Distinguish blocking from non-blocking.** Say plainly whether a comment must be addressed before merge or is a suggestion. Prefix optional comments with something like "nit:" or "optional:".
- **Say when you are satisfied.** A "+1" comment or a platform approval is the signal the author is waiting for.

---

## Security review

Reviewers MUST review for security, not only correctness and style.

### Changes that need proportionate scrutiny

The normative list lives in [standard §15](../security/secure-software-development-standard.md#15-code-review) and is repeated here for convenience. The following are **security-sensitive** and SHOULD be reviewed by someone with relevant experience — a project lead or senior engineer where available — and SHOULD NOT be merged on a quick skim:

- authentication, session, and token handling;
- authorization, roles, permissions, and multi-tenancy;
- cryptography, key handling, and password storage;
- payments, billing, and financial calculations;
- personal or otherwise sensitive data — new fields, new exports, new logging;
- infrastructure, IaC, IAM, network, and deployment configuration;
- file upload, download, parsing, and rendering;
- dependency additions and lockfile changes;
- externally exposed APIs, webhooks, and anything reachable without authentication;
- AI-generated code in any of the above areas.

### Reviewer security checklist

On a security-sensitive change, walk through this before approving:

- [ ] Is all input validated server-side, at the trust boundary, against a schema or allow-list?
- [ ] Is authorization checked **per object**, not just per route — including for the caller's tenant?
- [ ] Can a request write a field it should not (role, owner, tenant, price, status)?
- [ ] Are queries parameterized? Any string-built SQL or shell commands?
- [ ] Are there secrets, tokens, keys, or real data in the diff — including tests and fixtures?
- [ ] Do errors fail closed, and do they avoid leaking stack traces, SQL, or internal detail to users?
- [ ] Does anything sensitive end up in logs, traces, analytics, or third-party error reporting?
- [ ] Are new dependencies justified, maintained, and correctly named?
- [ ] Are the failure paths handled — timeouts, unavailable dependency, failed verification?
- [ ] Are there tests for the denial cases, not just the happy path?

A security concern raised in review SHOULD be resolved before merge, or explicitly deferred with a tracked issue and the project lead's agreement. It MUST NOT be silently dismissed.

---

## Requesting a review

- Open the pull request with a clear title using the conventions in [Git guidelines](git.md#writing-good-pr-titles).
- Assign or tag a primary reviewer so someone is responsible for leading the review. For a large refactor or a significant behavior change, name a specific person rather than hoping someone picks it up.
- Share the pull request link in the team's engineering channel and ask for review. Sharing it in Slack (for example the front/back development channel used by the team) is the fastest way to get attention; check with your project lead which channel a given project uses.
- Give reviewers time. Colleagues overseas will not see your request immediately.

---

## Merging

- All changes reach long-lived branches through a pull request. Direct commits to protected branches are prohibited ([Git guidelines](git.md#pull-requests)).
- Do not merge while an unresolved blocking comment or security concern is open.
- Resolve conflicts by updating your branch from the target branch before merging, and re-run the checks.
- **Approval requirements are configured per repository.** Some Cognativ repositories and many client-owned repositories enforce a specific number of approvals and required status checks through platform settings; where they do, those settings are authoritative. Where nothing is enforced by the platform, the project lead sets the expectation.
- **Security-sensitive changes MUST NOT be merged without an explicit review** by another engineer, regardless of how long the pull request has been open. "Nobody responded" is not approval for an authentication, authorization, cryptography, payments, sensitive-data, or infrastructure change.
- For a low-risk change that is not security-sensitive (documentation, a contained bug fix, a small performance improvement) and that has had no response after a reasonable wait — at least one working day — the author may merge it and note in the pull request that it was merged unreviewed. Use this sparingly; it is a fallback, not the normal path.
- **Hotfixes** for production incidents may be merged quickly to stop harm, but MUST still go through a pull request, MUST pass the test suite, and MUST be reviewed after the fact. Follow up on the same day where possible, and see [standard §18](../security/secure-software-development-standard.md#18-vulnerability-management) when the fix is security-related.

---

## Review etiquette

When reviewing others' work, the goal is to improve the code — not to judge the person. Clients sometimes read these threads too.

- Be respectful and professional. No ALL CAPS, no insults, no sarcasm, no emojis expressing disgust or frustration. Save emojis for greetings and congratulations.
- Comment on the code, not the author.
- Prefer questions and suggestions: "may I suggest…", "I tried this and it performed better", "what was the reason for…".
- Assume the author had a reason; ask before criticizing.
- Be helpful rather than terse. If more context is needed, move to a direct conversation and then summarize the outcome in the pull request.

**Do not:**

> What is this? 🤮

> 🤢 var names with only one letter!

> ARROW ANTIPATTERN, SERIOUSLY?

**Do:**

> This is turning into an arrow anti-pattern — the nesting makes the error paths hard to follow. Could we split each check into its own guard clause and return early? This article explains the trade-off well: https://…

> Why `let` here instead of `const`? If the reference never changes, `const` documents that for the next reader.

> If you spread the object here, all key-value pairs get copied at once, so we won't have to update this code when a field is added to the source object.

Explaining the rationale and linking to a reference makes the feedback useful beyond this one pull request.
