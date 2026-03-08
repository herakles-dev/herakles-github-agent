# Human-in-the-Loop

## The Human

**D. Michael Piscitelli** ([herakles-dev](https://github.com/herakles-dev)) supervises every phase of the contribution pipeline. This is not an autonomous AI bot submitting PRs. It's an engineer using a sophisticated agentic toolchain with full oversight at every step.

## What "Human-Supervised" Means

### The human decides:
- **Which issues to pursue** — The scoring system ranks issues, but the human selects which ones are worth pursuing based on context the automation can't capture (maintainer sentiment, project direction, strategic value).
- **Which approach to take** — The human evaluates potential solutions, considers trade-offs, and selects the implementation strategy. Alternatives are documented in the session record.
- **When comprehension is sufficient** — The tiered comprehension system suggests a level, but the human decides when they understand the code well enough to proceed.
- **Whether the implementation is correct** — Every line of code is reviewed by the human before committing.

### The human reviews:
- **Every diff before PR submission** — The pre-submit gate automates checks, but the human reads the actual diff.
- **Every PR description** — The methodology sign-off is a template, but the human verifies accuracy and edits for the specific contribution.
- **Every compliance output** — The compliance matrix is auto-generated, but the human verifies it matches reality for the specific repo.

### The human acts:
- **Claims issues personally** — Comments on issues are written or approved by the human.
- **Responds to review feedback** — All PR review interactions are human-to-human conversations.
- **Makes judgment calls** — When the automation says CAUTION, the human evaluates whether to proceed.
- **Accepts maintainer decisions** — Rejections are accepted gracefully. No arguing with maintainers.

## What the Automation Does

The agentic system handles the tedious but critical work that humans often skip:

- **Pre-flight validation** — 8 automated checks before investing any time on an issue. Catches competing PRs, stale claims, and blocking labels that a human might miss in a quick scan.
- **Convention detection** — Auto-reads CONTRIBUTING.md, CLAUDE.md, pyproject.toml to build a compliance matrix. Catches trailer requirements, DCO sign-off needs, and coverage thresholds.
- **Secrets scanning** — Regex-based scan of every diff for AWS keys, API tokens, private keys, hardcoded passwords. Catches the mistakes humans make under time pressure.
- **Session tracking** — Maintains a methodology record for every issue, creating the audit trail that proves rigor.
- **AI disclosure compliance** — Automatically enforces per-repo AI disclosure policies. If a project forbids AI mentions, the system blocks PRs that contain them.

## Why This Matters

The open source community is right to push back on low-effort AI-generated PRs. The solution isn't to abandon AI tooling — it's to use it to be **more rigorous**.

A human without this toolchain might:
- Miss a competing PR and waste time on a claimed issue
- Forget to add required trailers to commits
- Accidentally include a secret in a diff
- Submit without reading CONTRIBUTING.md thoroughly
- Skip documenting their approach and alternatives

With this toolchain, those failures are caught automatically. The human's judgment is augmented, not replaced. Every contribution goes through more checks than any purely manual workflow would include.

## The Standard

Every PR submitted through this architecture meets a higher bar than most manual contributions:

1. **Pre-validated** — 8-gate check confirms the issue is claimable
2. **Comprehension-backed** — Relevant project docs and code read at the appropriate depth
3. **Compliance-checked** — Per-repo conventions automatically enforced
4. **Security-scanned** — No secrets, no sensitive files
5. **Methodology-documented** — Full audit trail from issue to PR
6. **Alternatives-considered** — Not just the first approach that works
7. **Human-verified** — Every output reviewed before submission

This is what AI-assisted open source contributions should look like.
