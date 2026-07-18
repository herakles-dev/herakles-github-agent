# Architecture

This is the full technical walkthrough of the 10-phase pipeline. If you just want the overview, see the [README](README.md).

## The Shape of a Contribution

Every issue goes through the same pipeline. No shortcuts.

```
validate -> comprehend -> comply -> scaffold -> [implement] ->
pre-submit -> verify -> approve -> submit -> monitor
```

The pipeline is a state machine. Each phase writes its result to `pipeline.json`. The orchestrator won't run phase N+1 unless phase N has passed. If you interrupt a session and come back later, it picks up where you left off.

Three phases pause and wait for me: **scaffold** (choose the approach), **verify** (review agent findings), and **approve** (read the diff in my email and say yes). Everything else runs automatically.

## Phase 1: Validate

Before I spend any time on an issue, the system checks 8 things:

1. **Is it still open?** — Issues get closed. Check first.
2. **Is anyone assigned?** — Respect official assignments.
3. **Are there competing PRs?** — Search for open PRs that reference this issue. If someone's working on it, move on.
4. **Has anyone claimed it in comments?** — "I'll take this" counts as a claim for 14 days.
5. **Are there blocking labels?** — `wontfix`, `stale`, `needs confirmation`, `duplicate` — these mean don't bother.
6. **Does a contributing guide exist?** — If yes, I'll need to read it. If not, proceed with caution.
7. **How old is it?** — Issues older than a year might be stale for a reason.
8. **Do I have comprehension data for this repo?** — If I've scanned this repo before, that data speeds things up.

The output is simple: CLEAR, CAUTION, or BLOCKED. I don't start issues that come back BLOCKED.

## Phase 2: Comprehend

Not every issue needs the same depth of understanding. A typo fix doesn't need a full codebase analysis. A multi-file bug does. The system has 4 tiers:

**Tier 0 — Just read the files.** The issue says "line 42 of server.py has a typo." Read server.py. Done. Seconds.

**Tier 1 — Quick scan.** First time contributing to a repo. The system fetches CONTRIBUTING.md, README, CI config, PR template, package.json/pyproject.toml — about 20 key files — without cloning. Produces a summary: here's how they want commits formatted, here's whether they need DCO sign-off, here's their CI pipeline. Takes ~30 seconds.

**Tier 2 — Deep dive.** The bug spans multiple files or I don't understand the subsystem. Clone the repo. Map the source tree. Trace import chains from the files mentioned in the issue. Find test patterns. ~2 minutes.

**Tier 3 — Semantic search.** I'm planning 3+ contributions to the same repo and need to understand its architecture. Import the docs and key source files into a pgvector-backed RAG system. Now I can ask natural language questions across the whole codebase. ~5 minutes upfront, but it pays off across multiple issues.

The rule: use the minimum tier that gives you enough context. Don't over-research.

## Phase 3: Comply

Every open source project has its own rules. Some require DCO sign-off. Some require specific commit trailers. Some forbid any mention of AI tools. Some demand 100% test coverage. Some have CLAs.

The compliance script reads the comprehension data and auto-detects 8 things:
- AI disclosure policy (forbidden / allowed / required)
- DCO requirement
- Required commit trailers
- Coverage thresholds
- Commit format (conventional commits, angular, free-form)
- Forbidden patterns
- CLA requirement
- PR template existence

This becomes a `compliance.md` file that the rest of the pipeline reads. No manual configuration — it's all detected from the project's own docs.

**Why this matters:** The MCP python-sdk forbids any mention of AI tools in commits or PRs. If I accidentally include a `Co-Authored-By: Claude` trailer, the pre-submit gate catches it. Kubeflow requires DCO sign-off on every commit — if I forget `--signoff`, the gate catches that too. These are the kind of mistakes that get your PR immediately closed.

## Phase 4: Scaffold

The system creates a directory for this issue with everything I'll need: an issue spec with acceptance criteria, a session record to fill in as I work, a decisions log for approach selection, a task breakdown, and a pre-submission checklist auto-populated from the compliance matrix.

Then it **stops**. This is where I decide if the issue is actually worth doing, choose my approach, and document what alternatives I considered. The pipeline doesn't resume until I start writing code.

## Phase 5: Implement

I write the code. Fork, branch, implement, commit. The system checks for new commits in the fork directory to know when I'm done.

The rules I hold myself to:
- One issue = one PR. No scope creep.
- Follow the project's conventions exactly, even if I'd do it differently.
- Run the project's test suite locally.
- Keep changes minimal and focused.

## Phase 6: Pre-Submit

12 automated checks before anything goes near the verification agents:

**The blockers** (if any of these fail, full stop):
- No forbidden patterns in commits (AI mentions when the project forbids them)
- Required trailers present
- DCO sign-off present when required
- No secrets in the diff (AWS keys, API tokens, private keys, passwords — the regex patterns catch the common ones)
- No sensitive files (.env, .key, .pem, credentials.*)
- AI disclosure policy compliance

**The warnings** (proceed with caution):
- Test evidence is documented, not just template placeholders
- Acceptance criteria are defined
- Session record exists
- Alternatives are documented
- PR checklist is complete
- Pipeline state is consistent

I built this gate because it's fast (~2 seconds) and catches the stuff that's embarrassing to miss. The agent review in Phase 7 is slower and more thorough, but there's no point running 6 agents if I left an AWS key in my diff.

## Phase 7: Verify

This is the heart of the v2.1 pipeline. After 12 static checks pass, the system assembles a briefing document — issue spec, compliance rules, session record, full diff, complete contents of every changed file, and comprehension context — and spawns 6-8 specialist agents to review the code simultaneously.

### The Agents

**Correctness Reviewer** — Does this fix actually solve the issue? The prompt tells it to work through the problem using Polya's Method (Understand → Plan → Execute → Verify) and check every acceptance criterion against the code. It cites specific file:line references.

**Security Reviewer** — Does this fix introduce vulnerabilities? The prompt includes STRIDE threat modeling (Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation of Privilege) applied to the specific diff. Not "check for security issues" — that produces generic output. "Apply STRIDE to this diff" produces specific findings.

**Domain Specialist** — This one adapts. The system looks at what the diff touches and picks the right specialist. Dependency changes get a supply chain expert who verifies CVEs against real databases (not hallucinated ones). Auth changes get an auth specialist. CORS changes get an API security expert. This agent exists because a dependency PR needs different expertise than an auth PR.

**Conventions Reviewer** — Would a maintainer reject this on style alone? The prompt tells it to do a "30-second maintainer scan" — the kind of quick pattern-matching that experienced maintainers do when they open a PR. Wrong commit format? Scope creep? Formatting inconsistency? This agent catches it.

**Devil's Advocate** — My favorite. Its entire job is to argue against merging the PR. It uses dialectical reasoning (thesis → antithesis → synthesis) and pre-mortem analysis ("imagine this PR was rejected — why?"). It looks for unstated maintainer preferences, scope creep, architectural concerns. If it can't find a good reason to reject, the PR is probably solid.

**Integration Reviewer** — Does this break anything in the broader ecosystem? Package manager compatibility? Version conflicts? CI pipeline impacts? It uses systems thinking to trace second-order effects.

**Voice Reviewer** — added after a PR description came back from a first draft scoring 5/10. It doesn't touch the code at all — it reads the drafted PR description and asks one question: does this read like a human engineer wrote it, or like AI slop? It scores authenticity and flags the tells: needless tables, preemptive FAQs nobody asked, security-theater phrasing, over-explaining things the maintainers already know. A great fix with an AI-sounding writeup still gets you pattern-matched as spam, so this one matters as much as the code reviewers.

Each agent gives a binary PASS/FAIL with citations. If ANY agent fails, the pipeline stops. I fix the issue and re-run.

### Why This Exists

I added the agent review after the docs-agent CORS PR. The 12 static checks found nothing wrong with that diff. But when I ran 3 agents manually, they caught two real issues: a missing `ALLOWED_ORIGINS` environment variable in the deployment config and a wildcard CORS bypass. Static analysis can't catch those. You need something that actually reasons about the code.

## Phase 8: Approve

The full diff, all agent findings, and the compliance summary are emailed to me. I read every line. I reply "approved" — or I don't, and nothing gets pushed.

This gate exists because the agents are good but not perfect, and a clean pass doesn't mean I get to skip reading the diff myself. Case in point: reviving Inspector PR #1134, all six verify agents passed the fix — but the integration reviewer had done real work to get there, tracing `callTool`'s return value two hops out and catching a regression the other five missed: an app-resource tool run as a task opened its embedded view on a placeholder instead of the final result. I read that finding line by line, wrote a regression test that failed on the pre-fix code to prove it was real, and only then approved. The email forces me to slow down and actually read what I'm about to put my name on.

## Phase 9: Submit

After I approve: comment on the issue to claim it, push the branch, create the PR using the project's template. Include the methodology sign-off if the project allows AI disclosure.

## Phase 10: Monitor

Watch for review comments. Respond within 24 hours. If the maintainer asks for changes, make them and re-run the pre-submit + verify cycle. If they reject it, accept the decision gracefully and learn from it.

## The State Machine

Everything persists in `pipeline.json` (version 2.1). Here's a real one from the Inspector phantom dependencies fix:

```json
{
  "version": 2,
  "issue": "modelcontextprotocol/inspector#873",
  "current_phase": "approve",
  "phases": {
    "validate":   { "status": "passed", "result": "CLEAR" },
    "comprehend": { "status": "passed", "result": "reused-tier1" },
    "comply":     { "status": "passed", "result": "generated" },
    "scaffold":   { "status": "passed", "result": "created" },
    "implement":  { "status": "passed", "result": "3 commits" },
    "pre-submit": { "status": "passed", "result": "CLEAR" },
    "verify":     { "status": "passed", "result": "PASS", "findings": {...} },
    "approve":    { "status": "waiting" },
    "submit":     { "status": "pending" },
    "monitor":    { "status": "pending" }
  }
}
```

Interrupt the session, come back a week later, run `--resume` — it picks up exactly where it left off.

## Infrastructure

12 bash scripts, each doing one thing:

| Script | Job |
|--------|-----|
| `orchestrate.sh` | The master pipeline. 10 phases, state machine, agent coordination. |
| `validate.sh` | 8-gate pre-flight. Is this issue worth pursuing? |
| `comprehend.sh` | 4-tier comprehension. How deeply do I need to understand this repo? |
| `compliance.sh` | 8-detection compliance matrix. What are this project's rules? |
| `solve.sh` | Session scaffolding. Create the issue directory and artifacts. |
| `pre-submit.sh` | 12-check static gate. Did I make any obvious mistakes? |
| `notify.sh` | IONOS SMTP email. The approval gate + notifications. |
| `hunt.sh` | Issue discovery across target orgs. |
| `score.sh` | Weighted scoring. Which issues are worth the most? |
| `analyze-repo.sh` | Deep repo analysis. |
| `track.sh` | Contribution stats. |
| `gsoc.sh` | Google Summer of Code module. |
