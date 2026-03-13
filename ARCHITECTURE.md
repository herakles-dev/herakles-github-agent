# Architecture

## Overview

The Hercules Agentic Architecture is a 10-phase orchestrated pipeline that transforms a GitHub issue into a submitted PR. A single orchestrator script (`orchestrate.sh`) enforces strict sequential gates — no phase proceeds unless all prerequisites pass. State persists in `pipeline.json` (version 2) so sessions can be interrupted and resumed.

## Pipeline Flow

```
validate ──> comprehend ──> comply ──> scaffold ──> [HUMAN: implement]
   (auto)       (auto)       (auto)   (pause)         (manual)

──> pre-submit ──> verify ──> approve ──> submit ──> monitor
      (auto)      (agents)   (email)    (manual)   (continuous)
```

Three explicit human pause points: scaffold (exit 2), verify (exit 2), approve (exit 2).

## Phase Details

### Phase 1: Validate (8 Gates)

Pre-flight check before committing any time to an issue.

| Gate | What It Checks | Blocks If |
|------|---------------|-----------|
| 1. Issue state | Is it still OPEN? | Closed or merged |
| 2. Assignments | Is someone officially assigned? | Has assignee |
| 3. Competing PRs | Any open PRs referencing this issue? | Active competing PR |
| 4. Comment claims | Did someone say "I'll take this"? | Claimed < 14 days ago |
| 5. Label signals | `needs confirmation`, `wontfix`, `stale`? | Blocking labels present |
| 6. Contributing guide | Does CONTRIBUTING.md exist? CLA required? | Advisory only |
| 7. Freshness | How old is the issue? | > 365 days (advisory) |
| 8. Comprehension data | Scan summary and compliance matrix exist? | Advisory only |

Exit codes: `0` = CLEAR, `1` = BLOCKED, `2` = CAUTION.

### Phase 2: Comprehend (4 Tiers)

Tiered knowledge gathering — use the minimum depth needed, escalate only when context is insufficient.

| Tier | Name | Time | What It Does | Output |
|------|------|------|-------------|--------|
| 0 | Inline | Seconds | Read issue-referenced files directly | Embedded in context |
| 1 | Quick Scan | ~30s | Fetch 20 key project files without cloning | scan-summary.md, CONTRIBUTING.md, CI config, PR template |
| 2 | Deep Dive | ~2min | Clone repo, map sources, trace imports, detect tests | source-tree.txt, test-files.txt, dependencies.txt, issue-traces.txt |
| 3 | Atheneum | ~5min | Import docs + code into pgvector RAG system | Semantic search via API (port 8131) and MCP server |

Auto-escalation triggers:
- Issue body mentions 1 specific file → stay at Tier 0
- Issue body is vague ("X doesn't work") → Tier 1 minimum
- Fix requires understanding 3+ files → Tier 2
- 3+ contributions planned to same repo → consider Tier 3

### Phase 3: Comply (8 Detections)

Auto-generates a per-repo compliance matrix from comprehension data. No API calls — reads local scan data and pattern-matches.

| Detection | Output Field | What It Finds |
|-----------|-------------|---------------|
| AI disclosure policy | `policy: forbidden\|allowed\|required` | Scans CLAUDE.md + CONTRIBUTING.md for forbidden AI language |
| DCO requirement | `dco_required: true\|false` | Detects "DCO", "sign-off", "Signed-off-by" |
| Required trailers | `trailers_required: list\|none` | Extracts `Trailer-Name:` patterns |
| Coverage threshold | `coverage_requirement: %\|none` | Parses `fail_under` from pyproject.toml, `coverageThreshold` from package.json |
| Commit format | `commit_format: conventional\|angular\|free-form` | Pattern detection in contributing guide |
| Forbidden patterns | `forbidden_in_commits: list` | Auto-generated from AI disclosure policy |
| CLA requirement | `cla_required: true\|false` | Detects "CLA", "Contributor License Agreement" |
| PR template | `pr_template_exists: true\|false` | Checks `.github/pull_request_template.md` |

### Phase 4: Scaffold (Human Pause)

Creates per-issue artifact directory. Pipeline pauses (exit 2) for human to review spec and choose approach.

```
data/issues/OWNER-REPO-NUMBER/
├── spec.md           # Issue spec with acceptance criteria
├── session.md        # Methodology record (filled during implementation)
├── decisions.md      # Approach selection + alternatives
├── tasks.json        # V11 task definitions (6 tasks: T1-T6)
├── pr-checklist.md   # Pre-submission checklist (auto-filled from compliance.md)
└── pipeline.json     # Pipeline state (10 phases, version 2)
```

### Phase 5: Implement (Human)

Human implements the fix: fork, branch, commit. The system detects commits in the fork directory to mark this phase complete.

Rules:
- Minimal, focused changes (one issue = one PR)
- Follow project coding standards exactly
- Run project CI suite locally
- No scope creep

### Phase 6: Pre-Submit (12 Static Checks)

Fast static gate after implementation, before agent verification. Catches obvious issues.

| # | Category | Check | Blocks If |
|---|----------|-------|-----------|
| 0 | Prereqs | Comprehension + compliance data exist | Missing scan-summary.md or compliance.md |
| 1 | Commits | Forbidden patterns in commits | AI mention when project forbids |
| 2 | Commits | Required trailers present | Missing required trailer |
| 3 | Commits | DCO sign-off present | DCO required but missing |
| 4 | Security | Secrets in diff | AWS key, API token, private key, password pattern |
| 5 | Security | Sensitive files in diff | .env, .key, .pem, credentials.* |
| 6 | Quality | Test evidence documented | Template placeholders in session.md |
| 7 | Quality | Acceptance criteria defined | Missing from spec.md |
| 8 | Methodology | Session record exists | No session.md |
| 9 | Methodology | Alternatives documented | No alternatives in session/spec |
| 10 | Methodology | AI disclosure compliance | Policy mismatch |
| 11 | Methodology | PR checklist complete | Unchecked items |
| 12 | Methodology | Pipeline state consistent | validate/comprehend/comply not all passed |

Exit codes: `0` = CLEAR, `1` = BLOCKED (checks 0-5, 10), `2` = CAUTION (warnings only).

### Phase 7: Verify (5-7 Agent Review)

After static checks pass, the orchestrator pauses and assembles a briefing document containing the issue spec, compliance matrix, session record, full diff, changed file contents, and comprehension context. Then 5-7 specialist agents review the actual code in parallel.

#### Verification Formation

| Agent | Type | Lens | Problem-Solving Frameworks |
|-------|------|------|---------------------------|
| **1. Correctness** | code-quality-engineer | Does fix solve the issue? | Polya's Method, Bayesian evaluation, Five Whys, Inversion |
| **2. Security** | secure-coding-advisor | Does fix introduce vulnerabilities? | STRIDE threat model, Cynefin classification, attack surface analysis |
| **2b. Domain Specialist** | Adaptive (see below) | Domain-specific risk | Selected based on diff content |
| **3. Conventions** | code-quality-engineer | Follows project conventions? | Pareto principle, Inversion ("30-second maintainer scan") |
| **4. Devil's Advocate** | code-quality-engineer | Why would maintainer REJECT? | Dialectical thinking, pre-mortem, Six Hats Black Hat, Socratic method |
| **5. Integration** | code-quality-engineer | Breaks anything in ecosystem? | Systems thinking, Theory of Constraints, second-order effects |

#### Domain Specialist Selection (Agent 2b)

| Diff Touches | Agent Type | Focus |
|-------------|-----------|-------|
| Dependencies, packaging, npm audit | api-security-expert | Supply chain, CVE verification, typosquatting |
| Auth, OAuth, JWT, sessions | auth-specialist | OAuth2 flows, session fixation, privilege escalation |
| API routes, CORS, headers | api-security-expert | CORS bypass, header injection, origin validation |
| Data handling, SQL, deserialization | secure-coding-advisor | Injection, path traversal, deserialization |

#### Verification Output

Each agent produces a structured finding:

```json
{
  "verdict": "PASS|FAIL",
  "timestamp": "ISO-8601",
  "reviewers": [
    {
      "role": "correctness|security|supply-chain|conventions|devils-advocate|integration",
      "verdict": "PASS|FAIL",
      "summary": "One paragraph assessment",
      "findings": [
        { "file": "path", "line": 42, "status": "PASS|FAIL", "detail": "description" }
      ]
    }
  ],
  "summary": "Overall synthesis"
}
```

Overall verdict: **PASS only if ALL agents pass.** Any FAIL blocks the pipeline.

### Phase 8: Approve (Email Gate)

Full diff, verification findings, compliance status, and pipeline state are sent to the human via IONOS SMTP. The human must reply to approve. This is the final gate before any code is pushed or PR is created.

Email contents:
- Pipeline status (all phase results)
- Compliance matrix summary
- Verification findings (all agent verdicts)
- Full git diff
- Action required: reply "approved"

### Phase 9: Submit

After human approval:
1. Comment to claim the issue (if not already claimed)
2. Push branch to fork
3. Create PR using project's PR template
4. Include methodology sign-off (when AI disclosure is allowed)

### Phase 10: Monitor

Continuous tracking of PR review comments. Respond to feedback within 24 hours. Iterate on changes if requested. Accept maintainer decisions gracefully.

## Pipeline State (pipeline.json v2)

```json
{
  "version": 2,
  "issue": "modelcontextprotocol/inspector#873",
  "slug": "modelcontextprotocol-inspector-873",
  "created": "2026-03-13T05:30:04Z",
  "current_phase": "approve",
  "phases": {
    "validate": { "status": "passed", "result": "CLEAR", "timestamp": "..." },
    "comprehend": { "status": "passed", "result": "reused-tier1", "timestamp": "..." },
    "comply": { "status": "passed", "result": "generated", "timestamp": "..." },
    "scaffold": { "status": "passed", "result": "created", "timestamp": "..." },
    "implement": { "status": "passed", "result": "3 commits", "timestamp": "..." },
    "pre-submit": { "status": "passed", "result": "CLEAR", "timestamp": "..." },
    "verify": { "status": "passed", "result": "PASS", "timestamp": "...",
      "findings": { "verdict": "PASS", "reviewers": [...] }
    },
    "approve": { "status": "waiting", "result": null, "timestamp": "..." },
    "submit": { "status": "pending", "result": null, "timestamp": null },
    "monitor": { "status": "pending", "result": null, "timestamp": null }
  }
}
```

## Infrastructure

```
scripts/
├── orchestrate.sh   # Master pipeline (10 phases, state machine, agent verification)
├── validate.sh      # 8-gate pre-flight validation
├── comprehend.sh    # 4-tier comprehension framework
├── compliance.sh    # 8-detection compliance matrix generation
├── solve.sh         # V11 session scaffolding per issue
├── pre-submit.sh    # 12-check pre-submission gate
├── notify.sh        # IONOS SMTP email (approval gate + notifications)
├── hunt.sh          # Issue discovery across target orgs
├── score.sh         # Weighted issue scoring (6 factors)
├── analyze-repo.sh  # Deep repo analysis
├── track.sh         # Contribution tracking & stats
└── gsoc.sh          # Google Summer of Code 2026 module
```

## Key Principles

1. **Pipeline-first** — Single orchestrator enforces gate sequence with state persistence
2. **Quality over quantity** — One excellent PR beats ten mediocre ones
3. **Minimum viable comprehension** — Use the right tier, don't over-research
4. **Claim before coding** — Always comment on the issue first
5. **Atomic contributions** — One issue = one branch = one PR
6. **Respect the project** — Follow their conventions, not yours
7. **Adversarial verification** — The devil's advocate agent exists to find reasons to REJECT
