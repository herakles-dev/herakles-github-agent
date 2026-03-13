# Hercules Agentic Architecture

> AI contributions should be **more** rigorous, not less.

## The Problem

Open source has a spam problem. AI tools made it trivially easy to generate a "fix" and fire off a PR — so people do, constantly, without reading the contributing guide, without checking if someone's already working on it, without understanding the codebase. Maintainers are drowning in low-effort PRs that waste their time.

The backlash is understandable. Some projects now ban AI-assisted contributions entirely.

I think that's the wrong response. The problem isn't AI tooling — it's that people use it to skip the hard parts. Reading the contributing guide is boring. Checking for competing PRs is tedious. Documenting your approach and alternatives takes effort. Scanning your diff for accidentally committed secrets is easy to forget.

So I built a system that makes it impossible to skip those parts.

## What This Actually Is

It's a pipeline. A strict, sequential, gate-enforced pipeline that takes a GitHub issue and walks it through 10 phases before anything gets pushed. The system won't let me submit a PR unless I've:

- Verified nobody else is working on it (8 pre-flight checks)
- Read the project's contributing guide and detected its conventions automatically (4-tier comprehension + 8 compliance detections)
- Passed 12 static checks on my diff (secrets, trailers, DCO, AI disclosure policy)
- Had 5-7 AI agents independently review the code for correctness, security, convention compliance, integration risk, and — my favorite — an agent whose entire job is to find reasons a maintainer would reject it
- Received the full diff in my email and explicitly approved it

If any gate fails, the pipeline stops. No exceptions, no overrides. Fix the issue and re-run.

## How It Started vs. How It's Going

**V1** (March 7, 2026): 8 pre-flight gates + 10 pre-submit checks. Simple bash scripts. No agent review, no email gate. Submitted 4 PRs this way.

**V2** (March 13, 2026): Added 12 pre-submit checks, 5-7 parallel verification agents with problem-solving frameworks, and an email approval gate. Built this after a PR review caught issues that the static checks missed — a missing environment variable in a deployment config and a CORS wildcard bypass. Those were the kind of bugs that need someone actually thinking about the code, not just regex-matching for secrets. So I added agents that think.

**Current stats**: 5 PRs submitted, 0 merged (yet), 0 rejected. All still in review. The framework is a week old. I'll update this when the numbers change.

## The Pipeline

```
validate -> comprehend -> comply -> scaffold -> [implement] ->
pre-submit -> verify -> approve -> submit -> monitor
```

**Phases 1-3** run automatically: check if the issue is claimable, gather project context at the right depth, detect the project's conventions.

**Phase 4** pauses. I review the issue spec, choose an approach, document alternatives.

**Phase 5** is me writing code.

**Phase 6** runs 12 static checks on the diff. Catches the obvious stuff — secrets, missing trailers, DCO sign-off, AI disclosure policy violations.

**Phase 7** is where it gets interesting. The pipeline assembles a briefing document with the issue spec, compliance rules, full diff, and changed file contents. Then it spawns 5-7 specialist agents that review the code simultaneously:

- **Correctness** — Does this actually fix the issue? Did I miss edge cases?
- **Security** — Did I introduce a vulnerability? OWASP Top 10 relevance?
- **Domain specialist** — Picked based on what the diff touches. Dependency changes get a supply chain expert. Auth changes get an auth specialist. CORS changes get an API security expert.
- **Conventions** — Would a maintainer reject this on style alone? Does it match the project's patterns?
- **Devil's advocate** — What's the strongest argument for rejecting this PR? What are the maintainer's unstated preferences?
- **Integration** — Does this break anything downstream? Version conflicts? Package manager compatibility?

Every agent gives a PASS or FAIL with specific file:line citations. If any agent fails, the pipeline blocks.

**Phase 8** sends the complete diff and all agent findings to my email. I read every line. I reply "approved" or I don't.

**Phase 9**: push and submit the PR.

**Phase 10**: monitor reviews, respond within 24 hours.

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full technical walkthrough.

## A Real Example

Here's what this looks like in practice. [MCP Inspector #873](https://github.com/modelcontextprotocol/inspector/issues/873) — phantom dependencies that break pnpm installations.

The pipeline validated the issue (open since October 2025, zero comments, nobody working on it). Comprehension pulled the project's conventions. I traced every import in the bundled files to find exactly 5 missing dependencies. The pre-submit gate passed all 12 checks.

Then the verification agents ran. The **correctness reviewer** confirmed all 5 dependencies by tracing imports line-by-line. The **conventions reviewer** caught something I missed — the prettier hook had silently bumped a devDependency version during my commit. Fixed it. The **supply chain specialist** flagged two real CVEs in the dependencies I was adding (express-rate-limit IPv6 bypass, serve-handler minimatch ReDoS). The **devil's advocate** said: keep the scope tight, don't try to fix CVEs in the same PR — it's scope creep for a first-time contributor.

I wasn't sure about the CVE decision, so I spawned 4 more agents to independently verify the CVEs were real and not hallucinated. They confirmed across 5 sources (GitHub Advisory DB, npm audit, NVD, Snyk, GitLab Advisory). Real CVEs. But pre-existing in the workspace packages — my PR doesn't introduce them.

Final decision: ship the scoped fix, note the CVEs in the PR description, submit a separate version bump PR later. Two clean contributions instead of one messy one.

That's the kind of decision-making this framework is designed to support. Not replace — support.

## Who I Am

**D. Michael Piscitelli** ([herakles-dev](https://github.com/herakles-dev)). I select the issues, choose the approaches, review every line of code, approve every submission, and respond to every review comment. The system proposes. I dispose.

More on the human oversight model: [HUMAN-IN-THE-LOOP.md](HUMAN-IN-THE-LOOP.md)

## Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) — Full pipeline walkthrough, phase by phase
- [METHODOLOGY.md](METHODOLOGY.md) — Every gate and check, with the "why" behind each one
- [HUMAN-IN-THE-LOOP.md](HUMAN-IN-THE-LOOP.md) — What I decide, what the system decides, and where the line is
- [CONTRIBUTIONS.md](CONTRIBUTIONS.md) — Live PR tracker

## Stack

Claude Code, bash scripts, `gh` CLI, IONOS SMTP for the email gate, and a pgvector RAG system for deep codebase comprehension. The verification agents use problem-solving frameworks adapted from Cynefin, STRIDE, Polya, and dialectical reasoning — not because it sounds impressive, but because telling an agent "check for security issues" produces vague results, while telling it "apply STRIDE threat modeling to this diff" produces specific, actionable findings.

---

Built by [D. Michael Piscitelli](https://github.com/herakles-dev) | [herakles.dev](https://herakles.dev)
