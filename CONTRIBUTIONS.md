# Contributions

Everything I've submitted, where it stands, and what I learned.

## The Numbers

**First external merge landed.** The `opensource-pipeline` (3 agents + skill) was merged into [affaan-m/ECC](https://github.com/affaan-m/ECC) — a 230k-star agent-harness project — by the repo owner on March 31, 2026. Months later the files are still in `main`, and the distinctively-named agents now show up across ~50 downstream repos in the ecosystem's fork network.

That same skill went out to four community collections. ECC merged it; two are still open; one maintainer asked me to come back once the project matures. Separately, four MCP-ecosystem and infrastructure PRs are in review. The full ledger is below — closed ones included, because the ones that didn't land say as much as the ones that did.

## Merged

### ECC (agent harness optimization system) — 230k+ stars

**[#1036 — add opensource-pipeline: 3-agent workflow for safe public releases](https://github.com/affaan-m/ECC/pull/1036)** (merged Mar 31, 2026)
Added the `opensource-pipeline` skill and its three agents (`opensource-forker`, `opensource-sanitizer`, `opensource-packager`) — the workflow that strips secrets, sanitizes internal references, and packages a repo for safe public release. Merged by the repo owner after 8 comments and 8 reviews. Months later the files are still in `main` and actively maintained by the team (last touched July 9). The agents now appear across ~50 downstream repositories in the ecosystem's fork/adapt network.

## Open PRs

### MCP Inspector

**[#1132 — Handle list_changed notifications](https://github.com/modelcontextprotocol/inspector/pull/1132)** (Mar 7)
The inspector wasn't handling `tools/list_changed`, `resources/list_changed`, or `prompts/list_changed` MCP notifications. Added the handlers. Pipeline v1.

**[#1134 — Fix task polling blocking Run button](https://github.com/modelcontextprotocol/inspector/pull/1134)** (Mar 9, revived Jul 18)
The task polling loop ran synchronously and disabled the Run button until the task finished. Extracted it into a non-blocking `pollTaskInBackground` so the button stays live for concurrent calls. The PR sat unreviewed for four months and rotted into a merge conflict, so I rebased it onto v1.0.0 — where the conflict was a trap: v1.0.0 had added `input_required` elicitation handling *inside* the exact loop I was deleting. A blind resolution would have silently dropped that feature; I ported it into the background poller instead. A 6-agent QC swarm then caught a real second-order regression (running an app-resource tool as a task showed the embedded app a placeholder instead of the final result). Fixed it, added a regression test that provably fails on the pre-fix code, and shipped with 541 tests green. Pipeline v2.1.

**[#1144 — fix: declare bundled runtime dependencies for strict package managers](https://github.com/modelcontextprotocol/inspector/pull/1144)** (Mar 14, revived Jul 18)
Five phantom dependencies missing from root `package.json` that break pnpm and yarn PnP installs. Traced every import in every bundled file. When the inspector shipped v1.0.0 (July 18) without fixing it, I re-verified everything against the new release: corrected `express` (now 5.x, not 4.x), confirmed the dependency set is exactly five (the browser-bundled `ajv`/`ajv-formats` are *not* phantom deps — a trap the first pass could have fallen into), and dropped the old CVE note because a fresh audit showed zero vulnerabilities. Regenerated the lockfile in a Node 22 container to match CI, then rebased and re-ran a 6-agent verify swarm — which caught an uncommitted lockfile before it could red-X CI. Pipeline v2.1.

### Kubeflow

**[docs-agent #118 — CORS wildcard to env-var allowlist](https://github.com/kubeflow/docs-agent/pull/118)** (Mar 7)
Fixed a wildcard CORS configuration (`*`) that allowed any origin. Replaced with an environment-variable-driven allowlist. This was the PR that taught me static checks aren't enough — three agents found the missing `ALLOWED_ORIGINS` env var in deployment.yaml and a wildcard bypass vulnerability that 12 bash checks missed. Led to building the V2 agent verification phase. Pipeline v2.

### Skill collections

The `opensource-pipeline` skill that merged into ECC also went out to the community skill collections, so people can find it where they already look.

**[anthropics/skills #817 — Add opensource-pipeline skill](https://github.com/anthropics/skills/pull/817)** (Mar 31)
Submitted the same three-agent workflow to Anthropic's official skills collection.

**[ComposioHQ/awesome-claude-skills #546 — Add Open-Source Pipeline skill](https://github.com/ComposioHQ/awesome-claude-skills/pull/546)** (Mar 31)
Added it to the community-curated awesome-claude-skills list.

## Closed / Didn't Land

The ones that didn't make it — kept here on purpose. What you do with a rejection is part of the record.

**[VoltAgent/awesome-claude-code-subagents #157 — Add opensource pipeline agents](https://github.com/VoltAgent/awesome-claude-code-subagents/pull/157)** (Mar 31)
Same agents, submitted to another collection. The maintainer's call: *"the project looks quite new at the moment — please feel free to open a new PR once it has matured."* Fair. Noted, and I'll come back when there's a track record to point to.

**[modelcontextprotocol/python-sdk #2241 — pywin32 soft imports](https://github.com/modelcontextprotocol/python-sdk/pull/2241)** (Mar 7)
A five-line fix for a real cross-platform bug — the SDK hard-imported `pywin32`, which isn't installable everywhere. Closed by a maintainer because the underlying issue hadn't been triaged yet: per their CONTRIBUTING.md, wait for maintainer feedback before opening a PR. A process lesson, not a code one — respect the triage gate.

**[modelcontextprotocol/python-sdk #2242 — ClosedResourceError in _handle_message](https://github.com/modelcontextprotocol/python-sdk/pull/2242)** (Mar 7)
A race where the message handler could touch a closed resource. A contributor pointed out an existing PR (#2072) already covered it; I closed mine gracefully. The right call is not to compete on ground that's already taken.

## What I've Learned So Far

**The python-sdk is intensely competitive.** Many issues have multiple people racing to submit fixes. The validate.sh pre-flight check has saved me from wasting time on claimed issues several times.

**The inspector is less contested and more welcoming.** Good first issues actually available. Maintainers are responsive. The Node 22.7.5+ requirement blocked the full test suite locally (I'm on Node 20) until I moved verification into a `node:22` Docker container — now I run the real `tsc`, ESLint, the full Jest suite, and `vite build` exactly as CI does. On #1134 that meant 541 tests instead of a type-check and a hope.

**Kubeflow uses Prow bot and DCO sign-off.** Miss the `--signoff` flag and your PR is dead on arrival. The compliance matrix catches this automatically now.

**Prettier hooks surprise you.** On the Inspector phantom deps PR, the lint-staged pre-commit hook silently changed a prettier devDependency version during my commit. The conventions reviewer agent caught it. I wouldn't have noticed.

**AI agents hallucinate CVE details.** The supply chain agent said the fix version for express-rate-limit was `^8.3.1`. Four independent verification agents confirmed the CVE was real but the minimum fix was actually `^8.2.2`. Always verify. Never trust a single source.

**Your PR description is part of the contribution.** The first draft of the #1144 description scored 5/10 on voice authenticity — too many tables, preemptive FAQs, security-theater language ("trust boundaries"). Three rounds of voice review got it to 7-8/10. A 5-line diff doesn't need 40 lines of description. Match the weight of the words to the weight of the change.

**A rebase can silently delete someone else's feature.** Reviving #1134 after four months meant rebasing onto a release that had rewritten the exact code I was changing. The merge conflict wasn't "pick a side" — upstream had added `input_required` handling inside the loop I was deleting. Taking my side would have reverted their feature with not one failing test to warn me. Read what the other half of a conflict actually adds before you resolve it.

**The swarm earns its keep on the second-order stuff.** On #1134 all six reviewers passed the core fix, but the integration reviewer followed `callTool`'s return value two hops out and found a real regression the static gates and the other five agents missed — an app-resource tool run as a task opened its embedded view on a placeholder. A regression test, proven to fail without the fix, locked it. That's the whole point of agents that reason instead of run checklists.

## Pipeline Evolution

| Version | What Changed | Why |
|---------|-------------|-----|
| **v1** (Mar 7) | 8 pre-flight + 10 pre-submit checks | Starting point. Static analysis only. |
| **v2** (Mar 13) | 12 pre-submit + 5-7 verification agents + email approval | V1's static checks missed real issues on the Kubeflow PR. Added agents that actually reason about code. Added email gate because agents aren't perfect either. |
| **v2.1** (Mar 14) | Voice reviewer agent (Agent 6) | PR #1144's first draft scored 5/10 on authenticity. Added an agent that specifically checks if the PR description reads like a human wrote it. |

---

*Updated: 2026-07-18*
