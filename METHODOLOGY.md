# Methodology

## Gate Summary

The pipeline enforces 28+ automated gates across 4 categories:

| Category | Count | Script | When |
|----------|-------|--------|------|
| Pre-flight validation | 8 gates | `validate.sh` | Before any work begins |
| Compliance detection | 8 fields | `compliance.sh` | After comprehension |
| Pre-submission checks | 12 checks | `pre-submit.sh` | After implementation |
| Agent verification | 5-7 agents | `orchestrate.sh` | After pre-submit passes |

## Pre-Flight Validation (8 Gates)

Run before investing any time on an issue. Prevents wasted effort on claimed, blocked, or stale issues.

| Gate | What It Checks | Blocks If |
|------|---------------|-----------|
| 1. Issue state | Is it still OPEN? | Closed or merged |
| 2. Assignments | Is someone officially assigned? | Has assignee |
| 3. Competing PRs | Any open PRs referencing this issue? | Active competing PR |
| 4. Comment claims | Did someone say "I'll take this"? | Claimed < 14 days ago |
| 5. Label signals | `needs confirmation`, `wontfix`, `stale`, `invalid`, `duplicate`? | Blocking labels present |
| 6. Contributing guide | Does CONTRIBUTING.md exist? CLA required? | Advisory only |
| 7. Freshness | How old is the issue? | > 365 days (advisory) |
| 8. Comprehension data | Scan summary and compliance matrix exist? | Advisory only |

Verdicts: **CLEAR** (proceed), **CAUTION** (warnings exist), **BLOCKED** (stop).

### Competing PR Protocol

| Situation | Action |
|-----------|--------|
| Active PR, author engaged | Do NOT submit competing PR. Move on. |
| PR open < 14 days, no review | Wait. Don't compete. |
| PR stale 14-30 days, author silent | Comment: "Is this still active? Happy to help." |
| PR stale 30+ days, no author response | Comment offering to take over. If no reply in 3 days, submit your own PR referencing the stale one. |
| No PR, someone commented "I'll take this" | Treat as claimed for 14 days |
| No PR, no comments, no assignee | CLEAR — claim it |

### Claiming Protocol

1. Always comment before coding: "I'd like to take this. [Brief approach]. Will have a PR up shortly."
2. Deliver within 7 days or unclaim
3. One claim at a time per repo

## Compliance Detection (8 Fields)

Auto-generated from comprehension data. No API calls — reads local scan data and pattern-matches.

| # | Detection | Sources | Example Output |
|---|-----------|---------|----------------|
| 1 | AI disclosure policy | CLAUDE.md, CONTRIBUTING.md | `policy: forbidden` (MCP python-sdk) |
| 2 | DCO requirement | CONTRIBUTING.md ("Signed-off-by") | `dco_required: true` (Kubeflow) |
| 3 | Required trailers | CLAUDE.md, CONTRIBUTING.md | `trailers_required: Github-Issue, Reported-by` |
| 4 | Coverage threshold | pyproject.toml `fail_under`, package.json `coverageThreshold` | `coverage_requirement: 100%` |
| 5 | Commit format | CONTRIBUTING.md convention detection | `commit_format: conventional-commits` |
| 6 | Forbidden patterns | Auto-generated from AI disclosure policy | `forbidden_in_commits: co-authored-by, Claude, GPT` |
| 7 | CLA requirement | CONTRIBUTING.md ("CLA", "Contributor License Agreement") | `cla_required: true` |
| 8 | PR template | `.github/pull_request_template.md` exists? | `pr_template_exists: true` |

Output: `data/comprehension/OWNER-REPO/compliance.md` — used by pre-submit.sh and orchestrate.sh.

## Pre-Submission Gate (12 Checks)

Run after implementation, before agent verification. Fast static checks that catch obvious issues.

### Prerequisites (Check 0)
| # | Check | Blocker? |
|---|-------|----------|
| 0 | Comprehension + compliance data exist | BLOCKER |

### Commits (Checks 1-3)
| # | Check | Blocker? |
|---|-------|----------|
| 1 | Forbidden patterns in commits (AI mentions when project forbids) | BLOCKER |
| 2 | Required trailers present in latest commit | BLOCKER |
| 3 | DCO sign-off present when required | BLOCKER |

### Security (Checks 4-5)
| # | Check | Blocker? |
|---|-------|----------|
| 4 | Secrets in diff (AWS key, API token, private key, password, GitHub PAT, Slack token) | BLOCKER |
| 5 | Sensitive files in diff (.env, .key, .pem, .p12, .pfx, credentials.*, secrets.*, id_rsa) | BLOCKER |

### Quality (Checks 6-7)
| # | Check | Blocker? |
|---|-------|----------|
| 6 | Test evidence documented (not template placeholders) | Warning |
| 7 | Issue spec has acceptance criteria | Warning |

### Methodology (Checks 8-12)
| # | Check | Blocker? |
|---|-------|----------|
| 8 | Session record exists | Warning |
| 9 | Alternatives documented | Warning |
| 10 | AI disclosure compliance (final policy enforcement) | BLOCKER |
| 11 | PR checklist complete (no unchecked items) | Warning |
| 12 | Pipeline state consistent (validate/comprehend/comply all passed) | Warning |

Exit codes: `0` = CLEAR, `1` = BLOCKED (any blocker), `2` = CAUTION (warnings only).

## Agent Verification (5-7 Agents)

After static checks pass, the orchestrator assembles a briefing document and pauses. 5-7 specialist agents review the actual implementation in parallel. Each agent applies domain-specific problem-solving frameworks.

### Mandatory Agents (5)

| # | Role | Agent Type | Frameworks |
|---|------|-----------|-----------|
| 1 | Correctness Reviewer | code-quality-engineer | Polya's Method, Bayesian evaluation, Five Whys, Inversion, Abstraction Check |
| 2 | Security Reviewer | secure-coding-advisor | STRIDE threat model, Cynefin classification, Attack Surface Analysis, Inversion |
| 3 | Conventions Reviewer | code-quality-engineer | Pareto principle, Inversion ("30-second maintainer scan") |
| 4 | Devil's Advocate | code-quality-engineer | Dialectical thinking, Pre-mortem, Six Thinking Hats (Black Hat), Socratic Method |
| 5 | Integration Reviewer | code-quality-engineer | Systems Thinking, Theory of Constraints, Second-order effects |

### Adaptive Agent (Agent 2b)

Selected based on what the diff touches:

| Diff Domain | Agent Type | Focus |
|-------------|-----------|-------|
| Dependencies, packaging | api-security-expert | Supply chain CVEs, typosquatting, version analysis |
| Auth, OAuth, JWT | auth-specialist | OAuth2 flows, session fixation, privilege escalation |
| API routes, CORS, headers | api-security-expert | CORS bypass, header injection, origin validation |
| Data handling, SQL | secure-coding-advisor | Injection, path traversal, deserialization |

### Verification Protocol

1. Orchestrator assembles briefing from: spec.md, compliance.md, session.md, full diff, changed file contents, comprehension context
2. All agents run in parallel as background agents (Sonnet model for efficiency)
3. Each agent reads the briefing, reads source files, produces structured PASS/FAIL verdict with file:line citations
4. Orchestrator synthesizes results into `verify-findings.json`
5. Overall verdict: **PASS only if ALL agents pass**
6. Results recorded in pipeline.json via `--record-verify`

### Problem-Solving Protocol

Each agent prompt embeds relevant frameworks from a master protocol of 23 methods:

- **Cynefin Classification** — Categorize the problem domain (Simple/Complicated/Complex/Chaotic)
- **Polya's Method** — Understand → Plan → Execute → Verify
- **STRIDE Threat Model** — Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- **Dialectical Thinking** — Thesis → Antithesis → Synthesis (devil's advocate)
- **Systems Thinking** — Feedback loops, emergent properties, second-order effects
- **Theory of Constraints** — Find and address the bottleneck
- **Inversion** — "What would make this fail?" Pre-mortem analysis
- **Bayesian Evaluation** — Update confidence based on evidence, not priors

## Session Records

Every issue gets a methodology record (`session.md`) that tracks:
- Validation results and date
- Comprehension tier used and key files read
- Selected approach and rationale
- Alternatives considered with rejection reasons
- Implementation details (branch, files changed, tests)
- Test output evidence
- Pre-submission gate results
- Verification agent findings
- PR URL and submission date
- Review feedback log

This creates a complete audit trail from issue discovery to merged PR.

## V11 Session Integration

Each issue becomes a mini V11 session with 6 tracked tasks:

```
T1: Comprehension   -> Read issue, understand code area, run comprehend.sh
T2: Implement        -> Fork, branch, implement fix
T3: Test             -> Run project CI, add coverage
T4: Pre-Submit       -> 12-check compliance gate + 5-7 agent verification
T5: PR Submit        -> Claim issue, create PR with methodology sign-off
T6: Monitor          -> Respond to review feedback < 24h
```

The `solve.sh` script scaffolds the entire session structure. The `orchestrate.sh` script runs phases 1-4 automatically, pauses for human implementation, then runs phases 6-8 with gates.

## Target Organizations

Prioritized by ecosystem relevance and credibility value:

| Tier | Organizations | Why |
|------|--------------|-----|
| **S** | anthropics, modelcontextprotocol | Claude/MCP ecosystem — highest credibility |
| **A** | NVIDIA, vercel, vllm-project | GPU/AI infrastructure, high visibility |
| **B** | huggingface, langchain-ai, run-llama | ML/LLM tooling |
| **C** | Trending repos | Opportunistic high-visibility contributions |

## Issue Scoring

6-factor weighted scoring (0-100 scale):

| Factor | Weight | Signal |
|--------|--------|--------|
| Repo Stars | 15% | Visibility multiplier |
| Issue Reactions | 15% | Community demand |
| Label Quality | 15% | Maintainer invitation (`good first issue` = 90, `help wanted` = 85) |
| Solvability | 20% | Clear steps, small scope, familiar stack |
| Credibility Value | 20% | S-tier org, non-trivial code change, tests included |
| Freshness | 15% | Recent issues score higher |

Thresholds: pursue >= 55, priority >= 75, must-do >= 90.
