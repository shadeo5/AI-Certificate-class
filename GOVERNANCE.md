# Governance & Autonomy Model

This repository is governed by a **tiered autonomy model**. The guiding principle:

> **An autonomous agent may _prepare_ any change, but a human (the repository owner)
> is on every consequential action. Nothing autonomous reaches `main` without a
> recorded human sign-off.**

This document is the human-readable statement of intent. The machine-enforced
version lives in the branch ruleset, `.github/CODEOWNERS`, and
`.github/workflows/plan-gate.yml`. Where this document and the enforced controls
disagree, that gap is a defect to fix.

---

## Identities

| Identity | Role | Purpose |
|----------|------|---------|
| `shadeo5` | Repository **admin** | The accountable human. Reviews and approves the agent's work. |
| `shadeo5-agent` | **Write** collaborator | The autonomous agent. Opens PRs; cannot merge on its own. |

Keeping these separate is what makes the model enforceable: because the agent is
a distinct identity, the owner can be a valid **reviewer** of its work (GitHub
blocks self-approval, so a single identity could not review itself).

---

## Tiers

Scrutiny scales with blast radius. Tier is determined by the paths a PR touches.

| Tier | Example paths | Required to merge | Who can satisfy |
|------|---------------|-------------------|-----------------|
| **Low** | `docs/`, `*.md` | Plan Gate check + 1 review | Owner (no auto-merge) |
| **Medium** | `src/`, dependency bumps | Plan Gate check + 1 review | Owner (any review) |
| **High** | `infra/`, `.github/**` | Plan Gate check + 1 review + **code-owner review** | **Owner specifically** |
| **Critical** | production deploys, secrets | *(future)* protected Environment with required reviewers | Owner approves; agent cannot execute |

The **High** tier is enforced via path-scoped `.github/CODEOWNERS` +
`require_code_owner_review` in the ruleset — the one native mechanism for
per-path review requirements on a branch-wide ruleset.

`.github/CODEOWNERS` lists itself as an owned path, so the agent cannot rewrite
_who owns what_ without the owner's approval.

---

## Enforced controls (policy as code)

- **Branch ruleset on `main`** (`main-protection`):
  - Pull request required (no direct pushes)
  - ≥ 1 approving review
  - Require review from Code Owners
  - Required status check: `require-plan` (Plan Gate)
  - Block force-push (non-fast-forward) and branch deletion
  - **Bypass:** repository admin (see Accepted Risks)
- **Plan Gate** (`.github/workflows/plan-gate.yml`): every PR must carry the plan
  template. Hardened with least-privilege `permissions:` and a SHA-pinned action.
- **PR template** (`.github/pull_request_template.md`): enforces a written plan,
  evidence, and review checklist on every PR.

## Standards alignment

The model draws on: **NIST SSDF (SP 800-218)** and **SLSA** (documented, auditable
process), **OpenSSF Scorecard** (least-privilege token permissions, SHA-pinned
dependencies, security policy), and **GitHub's Actions hardening guidance**.

---

## Accepted risks

- **Admin bypass.** The repository admin (`shadeo5`) can override any gate via
  `gh pr merge --admin`. This is a *deliberate, accepted* trade-off: as a
  single accountable owner, the admin must retain direct control and self-service.
  It is mitigated by (a) GitHub requiring the override to be **explicit** (the
  passive path still refuses to merge), and (b) every override being logged.
  A stricter posture (SLSA L3+ two-person control) would remove admin from the
  bypass list, at the cost of requiring a second independent reviewer.

## Roadmap

- Move the agent identity from a long-lived PAT to a **GitHub App** (short-lived,
  scoped, signed-commit credentials).
- Require **signed-commit verification** in the ruleset.
- Stand up the **Critical tier** (protected Environment + required reviewers)
  when a real deploy target exists.
