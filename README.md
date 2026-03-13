# Hercules Agentic Architecture

> AI contributions should be **more** rigorous, not less.

A human-supervised, 10-phase orchestrated pipeline for open source contributions. While the OSS community rightly pushes back on low-effort AI-generated PRs, this framework proves that agentic tooling can produce contributions that exceed the rigor of purely manual ones — when the human stays in the loop and the system does the work humans skip.

## What This Is

A pipeline-first contribution engine that enforces strict sequential gates: no phase proceeds unless all prerequisites pass. The system orchestrates 8 pre-flight gates, 12 pre-submission checks, 8 compliance detections, and 5-7 parallel verification agents around a human engineer who makes every final call.

The human selects issues, evaluates approaches, reviews all outputs, and approves every submission via email gate. The system handles what humans skip under time pressure: convention detection, secrets scanning, import tracing, CVE analysis, and adversarial review.

## Why It Exists

The open source community has a problem: a flood of low-effort, AI-generated PRs that waste maintainer time. The backlash is understandable — many projects now ban or suspect AI-assisted contributions.

This project takes the opposite approach: **use AI tooling to be more rigorous, not less**. Every PR goes through more gates, more checks, and more review than any manual contribution would. The thesis: an engineer with a well-designed agentic pipeline should produce contributions that are objectively better than those without one.

## The 10-Phase Orchestrated Pipeline

```
validate (8 gates) -> comprehend (4 tiers) -> comply (8 detections)
  -> scaffold (artifacts) -> [HUMAN: implement] -> pre-submit (12 checks)
  -> verify (5-7 agents) -> approve (email gate) -> submit -> monitor
```

| Phase | Name | Gate Type | What Happens |
|-------|------|-----------|-------------|
| 1 | **Validate** | Auto | 8-gate pre-flight: issue state, assignments, competing PRs, claims, labels, contributing guide, freshness, comprehension data |
| 2 | **Comprehend** | Auto | 4-tier knowledge gathering: inline read, quick scan, deep clone, semantic RAG import |
| 3 | **Comply** | Auto | 8-detection compliance matrix: AI disclosure, DCO, trailers, coverage, commit format, forbidden patterns, CLA, PR template |
| 4 | **Scaffold** | Human pause | Create per-issue artifacts: spec.md, session.md, decisions.md, tasks.json, pr-checklist.md, pipeline.json |
| 5 | **Implement** | Human | Fork, branch, implement. Minimal, focused changes. Follow project conventions exactly. |
| 6 | **Pre-Submit** | Auto | 12-check static gate: forbidden patterns, trailers, DCO, secrets, sensitive files, test evidence, acceptance criteria, session record, alternatives, AI disclosure, checklist, pipeline state |
| 7 | **Verify** | Agent review | 5-7 parallel agents: correctness, security, domain specialist, conventions, devil's advocate, integration. Problem-solving frameworks embedded in each prompt. |
| 8 | **Approve** | Human (email) | Full diff + verification findings sent to human via SMTP. Must reply to proceed. |
| 9 | **Submit** | Human | Comment claim on issue, push branch, create PR with methodology sign-off |
| 10 | **Monitor** | Continuous | Track review comments, respond within 24h, iterate on feedback |

Each phase writes state to `pipeline.json` (version 2). Orchestrator enforces: no phase N+1 unless phase N = passed.

## V11 Agent Verification (Phase 7)

After 12 static checks pass, 5-7 specialist agents review the actual code in parallel:

| Agent | Role | Lens |
|-------|------|------|
| **Correctness** | Does the fix solve the issue? | Polya's Method, Bayesian evaluation, Five Whys |
| **Security** | Does the fix introduce vulnerabilities? | STRIDE threat model, attack surface analysis |
| **Domain Specialist** | Adaptive: supply chain, auth, API, data | Selected based on what the diff touches |
| **Conventions** | Does the fix follow project conventions? | Pareto principle, 30-second maintainer scan |
| **Devil's Advocate** | Why would a maintainer REJECT this? | Dialectical thinking, pre-mortem, Six Hats |
| **Integration** | Does the fix break anything in the ecosystem? | Systems thinking, Theory of Constraints |

Each agent produces a binary PASS/FAIL verdict with file:line citations. Overall: PASS only if ALL agents pass. Any FAIL blocks the pipeline.

## 28 Automated Gates

| Category | Count | Script |
|----------|-------|--------|
| Pre-flight validation | 8 gates | `validate.sh` |
| Compliance detection | 8 fields | `compliance.sh` |
| Pre-submission checks | 12 checks | `pre-submit.sh` |
| Agent verification | 5-7 agents | `orchestrate.sh` (phase 7) |

## Tiered Comprehension

| Tier | Name | Time | When | Output |
|------|------|------|------|--------|
| 0 | Inline | Seconds | Issue names exact files | Read files directly |
| 1 | Quick Scan | ~30s | First time contributing to a repo | scan-summary.md (CONTRIBUTING.md, CI, CLA, conventions) |
| 2 | Deep Dive | ~2min | Bug spans multiple files, unfamiliar subsystem | Source tree, test patterns, dependency map, import traces |
| 3 | Atheneum | ~5min | 3+ contributions to same repo | Docs + code imported into pgvector RAG for semantic search |

## Human-in-the-Loop

**D. Michael Piscitelli** ([herakles-dev](https://github.com/herakles-dev)) supervises every phase. Three explicit pause points enforce human oversight:

1. **Scaffold pause** (Phase 4) — Human reviews issue spec and chooses implementation approach
2. **Verify pause** (Phase 7) — Human reviews agent findings before proceeding to approval
3. **Approve gate** (Phase 8) — Human receives full diff + findings via email, must explicitly approve

See [HUMAN-IN-THE-LOOP.md](HUMAN-IN-THE-LOOP.md) for full details.

## Stack

- **Claude Code** (Anthropic) — Orchestration, agent coordination, code review
- **V11 Protocol** — Spec-driven development with Agent Teams and formation patterns
- **Problem-Solving Protocol** — 23 frameworks (Cynefin, STRIDE, Polya, Dialectical, Systems Thinking) embedded in agent prompts
- **gh CLI** — GitHub operations (fork, branch, PR, issue management, advisory lookup)
- **IONOS SMTP** — Email approval gate (human must approve every submission)
- **Project CI suites** — Per-project testing (never skip the project's own checks)

## Results

See [CONTRIBUTIONS.md](CONTRIBUTIONS.md) for a live tracker of PRs submitted and merged.

## Documentation

| Document | What's In It |
|----------|-------------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | 10-phase pipeline details, verification formation, pipeline state format |
| [METHODOLOGY.md](METHODOLOGY.md) | All 28 gates: 8 validation, 8 compliance, 12 pre-submit. Claiming protocol. |
| [HUMAN-IN-THE-LOOP.md](HUMAN-IN-THE-LOOP.md) | 3 pause points, what the human decides/reviews/acts on |
| [CONTRIBUTIONS.md](CONTRIBUTIONS.md) | Live PR tracker with status |

---

Built by [D. Michael Piscitelli](https://github.com/herakles-dev) | [herakles.dev](https://herakles.dev)
