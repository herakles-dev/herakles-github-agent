# Hercules Agentic Architecture

**A human-supervised contribution framework.** Every PR goes through 10 phases, 28+ automated gates, and 5-7 independent code review agents before I approve it from my inbox.

This is not a bot. I pick the issues, write the code, review every diff, and approve every submission. The system handles the parts most contributors skip: checking for competing PRs, detecting project conventions, scanning for secrets, tracing imports, verifying CVEs, and arguing against its own work.

> If you're a **maintainer reviewing a PR from [herakles-dev](https://github.com/herakles-dev)** — this is what happened before you saw it.

---

## What Your PR Went Through

```
validate (8 gates) -> comprehend -> comply (8 detections) -> scaffold
-> [I write the code] -> pre-submit (12 checks) -> verify (5-7 agents)
-> approve (I read the full diff in my email) -> submit -> monitor
```

**Before I started:** 8 automated checks confirmed the issue is open, unassigned, unclaimed, has no competing PRs, no blocking labels, and the contributing guide was read and parsed.

**Before anything touched GitHub:** 12 static checks scanned for secrets, verified commit trailers, enforced DCO sign-off, checked AI disclosure compliance, and validated the methodology documentation.

**Before I approved:** 5-7 specialist agents independently reviewed the code:

| Agent | What It Does |
|-------|-------------|
| Correctness | Verifies the fix actually solves the issue, line by line |
| Security | STRIDE threat model applied to the diff |
| Domain specialist | Adapts to the diff — supply chain, auth, CORS, or data security |
| Conventions | 30-second maintainer scan — would you reject this on style? |
| Devil's advocate | Argues against merging. Finds the reasons to say no. |
| Integration | Traces second-order effects on the broader ecosystem |

Every agent must PASS. One FAIL blocks the pipeline.

**Before I pushed:** The full diff and all agent findings were emailed to me. I read every line. I replied "approved."

---

## A Real Example

[**MCP Inspector #873**](https://github.com/modelcontextprotocol/inspector/issues/873) — 5 phantom dependencies breaking pnpm installations.

I traced every `import` in every bundled file. The **conventions reviewer** caught a prettier version bump that slipped in through a lint-staged hook — I wouldn't have noticed. The **supply chain specialist** found two real CVEs in the dependencies I was adding. The **devil's advocate** said: don't fix the CVEs in this PR, it's scope creep.

I wasn't sure about the CVE call. So I spawned 4 more agents to cross-verify against GitHub Advisory DB, npm audit, NVD, Snyk, and GitLab Advisory. Real CVEs — but pre-existing in the workspace packages, not introduced by my change.

Decision: ship the scoped fix, note the CVEs for the maintainers, submit a separate version bump later.

That's the kind of decision this framework is designed to support. Not replace — support.

---

## Why This Exists

Open source has a spam problem. AI tools made it easy to generate a fix and fire off a PR without reading the contributing guide, checking for competing work, or understanding the codebase. Maintainers are drowning in it.

I think the answer isn't banning AI — it's using it to be **more rigorous, not less**. Make it impossible to skip the hard parts. Every contribution through this pipeline goes through more checks than any purely manual workflow would include.

**Honest status:** 5 PRs submitted, 0 merged yet, 0 rejected. The framework is a week old. [Live tracker →](CONTRIBUTIONS.md)

---

## Who I Am

**D. Michael Piscitelli** ([herakles-dev](https://github.com/herakles-dev)). I select the issues, choose the approaches, write or review every line, approve every submission from my inbox, and respond to every review comment personally. [More on the human oversight model →](HUMAN-IN-THE-LOOP.md)

## Go Deeper

- [**Architecture**](ARCHITECTURE.md) — Full 10-phase pipeline walkthrough, with real examples and the messy parts
- [**Methodology**](METHODOLOGY.md) — Every gate and check explained, with the failure that inspired each one
- [**Human-in-the-Loop**](HUMAN-IN-THE-LOOP.md) — Where I decide, where the system decides, and where agents disagree
- [**Contributions**](CONTRIBUTIONS.md) — Live PR tracker with lessons learned

## Stack

Claude Code · bash · gh CLI · IONOS SMTP · pgvector RAG

---

Built by [D. Michael Piscitelli](https://github.com/herakles-dev) | [herakles.dev](https://herakles.dev)
