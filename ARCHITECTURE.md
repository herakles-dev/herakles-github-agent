# Architecture

## Overview

The Hercules Agentic Architecture is a 10-phase pipeline that transforms an open GitHub issue into a submitted PR, with automated validation at every stage. The human engineer supervises each phase and makes all critical decisions.

## Pipeline Flow

```
Issue Discovery → Score → Validate (8 gates) → Comprehend (Tier 0-3)
    → Claim → Fork/Branch → Implement → Test
    → Pre-Submit (10 checks) → PR Submit → Monitor Reviews
```

## Phase Details

### Phase 1: GitHub Fundamentals

Before contributing to any project, understand its norms:
- Read CONTRIBUTING.md for coding standards, commit format, PR process
- Check for CLA (Contributor License Agreement) requirements
- Understand CI pipeline and required checks
- Note any special workflows (Prow bot, auto-labeling, etc.)

### Phase 2: Issue Discovery

Automated search across target organizations:
- **S-Tier**: anthropics, modelcontextprotocol (Claude/MCP ecosystem)
- **A-Tier**: NVIDIA, vercel, nextjs (high-visibility infrastructure)
- **B-Tier**: huggingface, langchain-ai (ML ecosystem)
- **C-Tier**: Trending repos (opportunistic)

Filter by: `good first issue`, `help wanted`, `bug`, reaction count, recent activity.

### Phase 3: Target Cataloging

Score issues on weighted criteria:
- Repo Stars (20%) — visibility multiplier
- Issue Reactions (15%) — community demand signal
- Label Quality (15%) — maintainer invitation signals
- Solvability (20%) — can we fix this well?
- Credibility Value (15%) — will this PR demonstrate competence?
- Freshness (15%) — recent issues, no competing PRs

### Phase 4: Issue Management

Each active issue gets a dedicated directory:
```
data/issues/OWNER-REPO-NUMBER/
├── spec.md         # Acceptance criteria, constraints, approach
├── session.md      # Methodology record (filled as work progresses)
├── decisions.md    # Approach selection + alternatives considered
└── tasks.json      # V11 task definitions (6 tasks per issue)
```

### Phase 5: Spec-Driven Execution

Each issue becomes a mini V11 session with 6 tracked tasks:
1. **Comprehension** — Read issue, understand code area
2. **Implement** — Fork, branch, write the fix
3. **Test** — Run project CI, add coverage
4. **Pre-Submit** — 10-check compliance gate
5. **PR Submit** — Create PR with proper description
6. **Monitor** — Respond to review feedback within 24h

### Phase 6: Tiered Comprehension

Four levels of codebase understanding, used as needed:

**Tier 0 (Inline)**: Issue names exact files. Read them directly. Seconds.

**Tier 1 (Quick Scan)**: First contribution to a repo. Fetches key files without cloning: CONTRIBUTING.md, README, CI config, PR template, pyproject.toml/package.json. Produces a scan summary with CLA detection, CI pipeline info, commit conventions. ~30 seconds.

**Tier 2 (Deep Dive)**: Bug spans multiple files or unfamiliar subsystem. Clones the repo. Maps source tree, test patterns, dependencies. Traces import chains for issue-referenced files. ~2 minutes.

**Tier 3 (Atheneum)**: 3+ contributions planned to same repo. Imports docs and key source into a pgvector-backed RAG system for semantic search. ~5 minutes upfront, pays off across multiple issues.

### Phase 7-8: Implementation & Testing

- Fork, create feature branch
- Implement minimal, focused changes
- Follow project coding standards exactly
- Run the project's full CI suite locally
- Add/update tests for the fix

### Phase 8.5: Pre-Submission Compliance

10-check automated gate. Categories:
- **Commits**: Forbidden patterns, required trailers, DCO sign-off
- **Security**: Secrets scan, sensitive file detection
- **Quality**: Test evidence, acceptance criteria
- **Methodology**: Session record, alternatives documented, AI disclosure

### Phase 9: PR Submission

- Clear title referencing issue number
- Detailed description following project PR template
- Issue linked (Fixes #N)
- Methodology sign-off (when project allows AI disclosure)

### Phase 10: Community Building

- Respond to review feedback within 24 hours
- Accept maintainer decisions gracefully
- Build relationships through quality contributions
- Never argue about rejections

## Compliance Matrix

Auto-generated per repo from comprehension data. Detects:

| Field | How Detected |
|-------|-------------|
| AI disclosure policy | Pattern-match CLAUDE.md / CONTRIBUTING.md for forbidden language |
| DCO requirement | Look for "DCO", "sign-off", "Signed-off-by" in contributing guide |
| Required trailers | Extract `Trailer-Name:` patterns from CLAUDE.md / CONTRIBUTING.md |
| Coverage threshold | Parse `fail_under` from pyproject.toml or coverageThreshold from package.json |
| Commit format | Detect "conventional commit", "angular", or "feat:/fix:" patterns |
| CLA requirement | Look for "CLA", "Contributor License Agreement" in contributing guide |

## Infrastructure

```
scripts/
├── hunt.sh         # Issue discovery across target orgs
├── score.sh        # Weighted issue scoring
├── validate.sh     # 8-gate pre-flight validation
├── comprehend.sh   # 4-tier comprehension framework
├── compliance.sh   # Per-repo compliance matrix generation
├── solve.sh        # V11 session scaffolding per issue
├── pre-submit.sh   # 10-check pre-submission gate
├── analyze-repo.sh # Deep repo analysis
├── track.sh        # Contribution tracking
└── gsoc.sh         # Google Summer of Code module
```

## Key Principles

1. **Quality over quantity** — One excellent PR beats ten mediocre ones
2. **Minimum viable comprehension** — Don't over-research. Use the right tier.
3. **Claim before coding** — Always comment on the issue first
4. **Atomic contributions** — One issue = one branch = one PR
5. **Respect the project** — Follow their conventions, not yours
