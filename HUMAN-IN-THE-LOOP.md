# Human-in-the-Loop

## The Human

**D. Michael Piscitelli** ([herakles-dev](https://github.com/herakles-dev)) supervises every phase of the contribution pipeline. This is not an autonomous AI bot submitting PRs. It's an engineer using a pipeline-first agentic toolchain with enforced oversight at every critical decision point.

## Three Enforced Pause Points

The orchestrator pipeline (`orchestrate.sh`) enforces three hard stops where the system halts and waits for human action:

### Pause 1: Scaffold (Phase 4)

The system has validated the issue, gathered comprehension data, and generated a compliance matrix. It scaffolds the issue artifacts (spec, session record, decisions log) and **stops**.

**The human decides:**
- Is this issue worth pursuing? (scoring suggests yes, but context matters)
- Which implementation approach? (alternatives are documented)
- Is the comprehension tier sufficient? (escalate if needed)

The pipeline resumes only when the human begins implementation.

### Pause 2: Verify (Phase 7)

After implementation and 12 static checks pass, the system assembles a briefing and **stops**. 5-7 specialist agents review the code. The human sees all agent findings.

**The human decides:**
- Are the agent findings accurate? (agents can have false positives/negatives)
- Should any FAIL verdicts be overridden? (domain knowledge the agents lack)
- Is additional work needed before approval?

### Pause 3: Approve (Phase 8)

The full diff, verification findings, compliance status, and pipeline state are sent to the human's email via IONOS SMTP. The pipeline **stops and waits for an explicit approval reply**.

**The human reviews:**
- The complete diff (every line of code)
- All agent verification findings
- The PR description and methodology sign-off
- Compliance output accuracy

Nothing is pushed or submitted until the human replies "approved."

## What the Human Decides

- **Which issues to pursue** — The scoring system ranks issues, but the human selects based on context the automation can't capture: maintainer sentiment, project direction, strategic value, ecosystem dynamics.
- **Which approach to take** — The human evaluates potential solutions, considers trade-offs, and selects the implementation strategy. Alternatives are documented in the session record.
- **When comprehension is sufficient** — The tiered system suggests a level, but the human decides when they understand the codebase well enough.
- **Whether the implementation is correct** — Every line of code is reviewed by the human before committing.
- **Scope decisions** — When agents disagree (e.g., supply chain specialist says "bump CVE versions" but devil's advocate says "keep scope tight"), the human resolves the tension.

## What the Human Reviews

- **Every diff before submission** — The pre-submit gate automates 12 checks, but the human reads the actual diff.
- **Every PR description** — The methodology sign-off is a template, but the human verifies accuracy and edits for the specific contribution.
- **Every compliance output** — The compliance matrix is auto-generated, but the human verifies it matches reality.
- **Every agent finding** — The 5-7 verification agents produce structured verdicts, but the human evaluates whether they're correct.

## What the Human Acts On

- **Claims issues personally** — Comments on issues are written or approved by the human.
- **Responds to review feedback** — All PR review interactions are human-to-human conversations.
- **Makes judgment calls** — When automation says CAUTION, the human evaluates whether to proceed.
- **Resolves agent disagreements** — When the devil's advocate and the security reviewer disagree on scope, the human decides.
- **Accepts maintainer decisions** — Rejections are accepted gracefully. No arguing with maintainers.

## What the Automation Handles

The system does the work that humans skip under time pressure:

- **Pre-flight validation** — 8 automated gates before investing any time on an issue. Catches competing PRs, stale claims, and blocking labels.
- **Convention detection** — Auto-reads CONTRIBUTING.md, CLAUDE.md, pyproject.toml to build a compliance matrix. Catches trailer requirements, DCO sign-off needs, coverage thresholds.
- **Secrets scanning** — Regex scan of every diff for AWS keys, API tokens, private keys, hardcoded passwords.
- **Import tracing** — Traces actual import statements in source files to verify dependency declarations are complete and correct.
- **CVE verification** — Cross-references dependencies against GitHub Advisory Database, npm audit, NVD, and Snyk to verify vulnerabilities are real (not hallucinated).
- **Adversarial review** — The devil's advocate agent specifically looks for reasons a maintainer would reject the PR: scope creep, unstated preferences, style mismatches.
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
- Miss a CVE in a dependency they're adding
- Not consider why a maintainer might reject the PR

With this pipeline, those failures are caught automatically. The human's judgment is augmented, not replaced. Every contribution goes through more checks than any purely manual workflow would include.

## The Standard

Every PR submitted through this architecture:

1. **Pre-validated** — 8-gate check confirms the issue is claimable and worth pursuing
2. **Comprehension-backed** — Project docs and code read at the appropriate depth (Tier 0-3)
3. **Compliance-checked** — 8 per-repo conventions auto-detected and enforced
4. **Security-scanned** — 12 static checks for secrets, sensitive files, and policy violations
5. **Agent-verified** — 5-7 specialist agents review correctness, security, conventions, integration, and adversarially challenge the PR
6. **Methodology-documented** — Full audit trail from issue discovery to PR submission
7. **Alternatives-considered** — Not just the first approach that works
8. **Email-approved** — Human explicitly approves the full diff before any code is pushed

This is what AI-assisted open source contributions should look like.
