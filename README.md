# Hercules Agentic Architecture

> AI contributions should be **more** rigorous, not less.

A human-supervised, 10-phase contribution framework for rigorous open source contributions. While the OSS community rightly pushes back on low-effort AI-generated PRs, this project proves that agentic tooling can produce contributions that exceed the rigor of purely manual ones — when the human stays in the loop.

## What This Is

An engineering framework that orchestrates automated validation, comprehension, and compliance tooling around a human engineer. The human selects issues, evaluates approaches, reviews all outputs, and makes every final call. The agentic system handles the tedious but critical work: pre-flight validation, convention detection, secrets scanning, and methodology tracking.

## Why It Exists

The open source community has a problem: a flood of low-effort, AI-generated PRs that waste maintainer time. The backlash is understandable — many projects now ban or suspect AI-assisted contributions.

This project takes the opposite approach: **use AI tooling to be more rigorous, not less**. Every PR goes through more gates, more checks, and more documentation than any manual contribution would.

The thesis: an engineer with a well-designed agentic toolchain should produce contributions that are objectively better than those without one.

## The 10-Phase Pipeline

| Phase | Name | What Happens |
|-------|------|-------------|
| 1 | **GitHub Fundamentals** | Read CONTRIBUTING.md, CoC, understand project norms |
| 2 | **Issue Discovery** | Search across target orgs, filter by visibility and solvability |
| 3 | **Target Cataloging** | Score issues using weighted criteria (stars, activity, difficulty) |
| 4 | **Issue Management** | Per-issue directory with spec, session record, decision log |
| 5 | **Spec-Driven Execution** | V11 mini-session with task tracking per issue |
| 6 | **Tiered Comprehension** | 4-level system: inline read, quick scan, deep clone, semantic RAG |
| 7 | **Implementation** | Fork, branch, implement with minimal focused changes |
| 8 | **Integration Testing** | Run project CI locally, add/update tests |
| 8.5 | **Pre-Submission Compliance** | 10-check gate: secrets, conventions, trailers, AI disclosure |
| 9 | **PR Submission** | Clear description, linked issue, methodology sign-off |
| 10 | **Community Building** | Respond to feedback promptly, build maintainer trust |

## 17 Automated Gates

### Pre-Flight Validation (8 gates)

Run before investing any time on an issue:

1. Issue state (still OPEN?)
2. Assignments (is someone assigned?)
3. Competing PRs (any open PRs for this issue?)
4. Comment claims ("I'll take this" within 14 days?)
5. Label signals (needs-confirmation, wontfix, stale?)
6. Contributing guide (exists? CLA required?)
7. Freshness (issue age and recent activity)
8. Comprehension data (scan summary and compliance matrix exist?)

### Pre-Submission Gate (10 checks)

Run after implementation, before PR creation:

1. Forbidden patterns in commits (co-authored-by when project forbids)
2. Required trailers present (Github-Issue, Reported-by, etc.)
3. DCO sign-off present when required
4. Secrets patterns in diff (AWS keys, API tokens, private keys)
5. No sensitive files in diff (.env, *.key, *.pem, credentials.*)
6. Test evidence documented
7. Issue spec has acceptance criteria
8. Session record exists with methodology
9. Alternatives documented
10. AI disclosure compliance (forbidden = no mentions; allowed/required = appropriate)

## Tiered Comprehension System

Not every issue needs the same depth of understanding. The framework auto-selects the right level:

| Tier | Name | When | What |
|------|------|------|------|
| 0 | Inline | Issue names exact files, fix is obvious | Read referenced files directly |
| 1 | Quick Scan | First time contributing to a repo | Fetch CONTRIBUTING.md, CI config, CLA, conventions |
| 2 | Deep Dive | Bug spans multiple files, unfamiliar subsystem | Clone repo, trace call chains, map dependencies |
| 3 | Atheneum | 3+ contributions to same repo, architecture changes | Import into semantic search (pgvector RAG) |

## Per-Repo Compliance Matrix

Auto-generated from comprehension data. Detects:
- AI disclosure policy (forbidden / allowed / required)
- DCO / sign-off requirements
- Required commit trailers
- Coverage thresholds
- Commit format conventions (conventional-commits, angular, free-form)
- CLA requirements
- PR template presence

## Human-in-the-Loop

See [HUMAN-IN-THE-LOOP.md](HUMAN-IN-THE-LOOP.md) for full details.

**The human (D. Michael Piscitelli) supervises every phase.** This is not an autonomous bot submitting PRs. It's an engineer using a sophisticated toolchain with full oversight:

- Human selects which issues to pursue
- Human evaluates and chooses approaches
- Human reviews every line of generated code
- Human reviews diff, PR description, and compliance output before submission
- Human responds to PR review feedback
- Human makes all final decisions

## Stack

- **Claude Code** (Anthropic) — AI-assisted development with human oversight
- **V11 Protocol** — Spec-driven development with task tracking
- **gh CLI** — GitHub operations (fork, branch, PR, issue management)
- **Project CI suites** — Per-project testing (never skip the project's own checks)

## Results

See [CONTRIBUTIONS.md](CONTRIBUTIONS.md) for a live tracker of PRs submitted and merged.

## Architecture Deep Dive

- [ARCHITECTURE.md](ARCHITECTURE.md) — Full 10-phase framework details
- [METHODOLOGY.md](METHODOLOGY.md) — Validation pipeline and compliance automation
- [HUMAN-IN-THE-LOOP.md](HUMAN-IN-THE-LOOP.md) — Human oversight at every stage
- [CONTRIBUTIONS.md](CONTRIBUTIONS.md) — Contribution tracker

---

Built by [D. Michael Piscitelli](https://github.com/herakles-dev) | [herakles.dev](https://herakles.dev)
