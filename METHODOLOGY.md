# Methodology

## Validation Pipeline

Every issue goes through a structured validation pipeline before any code is written.

### Pre-Flight Validation

The `validate.sh` script checks 8 gates and returns a verdict:

| Gate | What It Checks | Blocks If |
|------|---------------|-----------|
| Issue state | Is it still OPEN? | Closed or merged |
| Assignments | Is someone officially assigned? | Has assignee |
| Competing PRs | Any open PRs referencing this issue? | Active competing PR |
| Comment claims | Did someone say "I'll take this"? | Claimed < 14 days ago |
| Label signals | `needs confirmation`, `wontfix`, `stale`? | Blocking labels present |
| Contributing guide | Does CONTRIBUTING.md exist? CLA required? | Advisory only |
| Freshness | How old is the issue? | > 365 days (advisory) |
| Comprehension data | Scan summary and compliance matrix exist? | Advisory only |

**Verdicts**: CLEAR (proceed), CAUTION (warnings), BLOCKED (stop).

### Competing PR Protocol

| Situation | Action |
|-----------|--------|
| Active PR, author engaged | Do NOT submit competing PR |
| PR open < 14 days, no review | Wait |
| PR stale 14-30 days, author silent | Comment offering to help |
| PR stale 30+ days, no response | Comment offering to take over |
| No PR, someone commented "I'll take this" | Treat as claimed for 14 days |
| No PR, no comments, no assignee | CLEAR — claim it |

### Claiming Protocol

1. Always comment before coding: "I'd like to take this. [Brief approach]. Will have a PR up shortly."
2. Deliver within 7 days or unclaim
3. One claim at a time per repo

## Compliance Automation

The `compliance.sh` script generates a per-repo compliance matrix from existing comprehension data. It never hits the GitHub API — it reads local scan data and pattern-matches for:

- **AI disclosure policy**: Scans CLAUDE.md and CONTRIBUTING.md for forbidden AI language. Default: `allowed`.
- **DCO/sign-off**: Detects "DCO", "sign-off", "Signed-off-by" references.
- **Required trailers**: Extracts `Trailer-Name:` patterns (e.g., `Github-Issue:`, `Reported-by:`).
- **Coverage threshold**: Parses `fail_under` from pyproject.toml.
- **Commit format**: Detects conventional-commits, angular, or free-form.
- **CLA requirement**: Looks for CLA references in contributing guide.

## Pre-Submission Gate

The `pre-submit.sh` script runs 10 checks after implementation, before PR creation:

### Commits (3 checks)
1. **Forbidden patterns** — If the compliance matrix says AI disclosure is forbidden, verify no AI-related strings appear in commit messages (co-authored-by, Claude, GPT, etc.)
2. **Required trailers** — Verify all project-required trailers are present in the latest commit
3. **DCO sign-off** — If DCO is required, verify `Signed-off-by:` is present

### Security (2 checks)
4. **Secrets scan** — Regex scan of the diff for AWS keys, API tokens, private keys, hardcoded passwords
5. **Sensitive files** — Check that no `.env`, `*.key`, `*.pem`, or `credentials.*` files are in the diff

### Quality (2 checks)
6. **Test evidence** — Session record documents test output
7. **Acceptance criteria** — Issue spec has defined acceptance criteria

### Methodology (3 checks)
8. **Session record** — Methodology record exists
9. **Alternatives documented** — At least one alternative approach considered and documented
10. **AI disclosure compliance** — Final check that AI mentions match the project's policy

## Session Records

Every issue gets a methodology record (`session.md`) that tracks:
- Validation results and date
- Comprehension tier used and files read
- Selected approach and rationale
- Alternatives considered with rejection reasons
- Implementation details (branch, files, tests)
- Test output evidence
- Pre-submission gate results
- PR URL and submission date
- Review feedback log

This creates a complete audit trail from issue discovery to merged PR.

## V11 Session Integration

Each issue becomes a mini V11 session with 6 tracked tasks:

```
T1: Comprehension   → Read issue, understand code area
T2: Implement        → Fork, branch, implement fix
T3: Test             → Run project CI, add coverage
T4: Pre-Submit       → 10-check compliance gate
T5: PR Submit        → Create PR with methodology sign-off
T6: Monitor          → Respond to review feedback < 24h
```

The `solve.sh` script scaffolds the entire session structure automatically.
