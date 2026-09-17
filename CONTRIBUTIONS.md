# Contributions

Everything I've submitted, where it stands, and what I learned. The ledger below is generated straight from live GitHub state, so it never drifts out of sync with what's actually open, closed, or merged — the story around it is the part I write by hand.

## The Story So Far

The `opensource-pipeline` skill — three agents: forker, sanitizer, packager — was merged into [affaan-m/ECC](https://github.com/affaan-m/ECC), a 230k-star agent-harness project, by the repo owner on March 31, 2026. Months later the files are still in `main`, maintainer-edited, and the distinctively-named agents now show up across roughly 50 downstream repos in the ecosystem's fork network. That's the strongest signal so far that this actually works: not a PR getting merged, but a stranger's codebase quietly absorbing it.

The same pipeline went out to a few other places too, and a couple of other lines of work have opened up since — some landed, some didn't, some are still waiting on review. Rather than restate a snapshot here that goes stale the next time something moves, the ledger below reflects live state every time it's regenerated.

## Merged

| Repo | PR | Title | Date | Note |
|---|---|---|---|---|
| affaan-m/ECC | [#1036](https://github.com/affaan-m/ECC/pull/1036) | feat(agents,skills): add opensource-pipeline — 3-agent workflow for safe public releases | 2026-03-31 | Merged by the repo owner after 8 reviews. Months later the files are still in main and actively maintained; the agents now show up in roughly 50 downstream repos. |

## Open

| Repo | PR | Title | Date | Note |
|---|---|---|---|---|
| affaan-m/ECC | [#2966](https://github.com/affaan-m/ECC/pull/2966) | fix(agents): list files the sanitizer cannot read | 2026-09-05 | Binary-blindspot fix for the sanitizer's UNSCANNED gate, about ten rounds of review-bot back-and-forth. Latest revision closes the hole with an additive allowlist rather than a broader rewrite; awaiting CI and a maintainer re-review. |
| anthropics/skills | [#817](https://github.com/anthropics/skills/pull/817) | Add opensource-pipeline skill | 2026-03-31 | The same opensource-pipeline skill that merged into ECC, submitted to Anthropic's own skills collection. |
| google-deepmind/formal-conjectures | [#5425](https://github.com/google-deepmind/formal-conjectures/pull/5425) | feat(ErdosProblems/399): prove erdos_399.variants.cambie | 2026-09-10 | Filled a sorry with an elementary mod-8 Lean proof of the Erdos #399 Cambie variant, machine-verified with no sorryAx. |
| google-deepmind/formal-conjectures | [#5481](https://github.com/google-deepmind/formal-conjectures/pull/5481) | feat(ErdosProblems/672): link an external Lean proof of erdos_672.variants.euler | 2026-09-10 | A one-line formal_proof linking a public Lean repo that proves the Erdos #672 Euler four-squares variant. |

## Closed (unmerged)

| Repo | PR | Title | Date | Note |
|---|---|---|---|---|
| ComposioHQ/awesome-claude-skills | [#546](https://github.com/ComposioHQ/awesome-claude-skills/pull/546) | Add Open-Source Pipeline skill | 2026-07-26 | The same opensource-pipeline skill, submitted to another community collection. |
| VoltAgent/awesome-claude-code-subagents | [#157](https://github.com/VoltAgent/awesome-claude-code-subagents/pull/157) | Add opensource pipeline agents (forker, sanitizer, packager) | 2026-04-01 | The same opensource-pipeline agents. Maintainer asked me to resubmit once the source project has more of a track record. |
| kubeflow/docs-agent | [#118](https://github.com/kubeflow/docs-agent/pull/118) | security(server): replace CORS wildcard with env-var allowlist | 2026-09-17 | Replaced a wildcard CORS config with an env-var-driven allowlist. Superseded upstream by #218. |
| modelcontextprotocol/inspector | [#1132](https://github.com/modelcontextprotocol/inspector/pull/1132) | feat: handle tools/resources/prompts list_changed notifications | 2026-07-31 | tools/resources/prompts list_changed notification handling. Closed unmerged when Inspector v2 deprecated the entire v1 PR queue. |
| modelcontextprotocol/inspector | [#1134](https://github.com/modelcontextprotocol/inspector/pull/1134) | fix: task polling blocking the Run button | 2026-07-31 | Made task polling non-blocking so the Run button stays live during a call. Closed unmerged when Inspector v2 deprecated the entire v1 PR queue. |
| modelcontextprotocol/inspector | [#1144](https://github.com/modelcontextprotocol/inspector/pull/1144) | fix: declare bundled runtime dependencies for strict package managers | 2026-07-31 | Declared five phantom bundled runtime dependencies missing from the root package.json. Closed unmerged when Inspector v2 deprecated the entire v1 PR queue. |
| modelcontextprotocol/python-sdk | [#2241](https://github.com/modelcontextprotocol/python-sdk/pull/2241) | fix: make pywin32 imports soft to support server-only Windows installs | 2026-03-24 | pywin32 soft imports so server-only Windows installs don't hard-fail. Closed for landing before the issue had been triaged. |
| modelcontextprotocol/python-sdk | [#2242](https://github.com/modelcontextprotocol/python-sdk/pull/2242) | fix: catch ClosedResourceError in _handle_message error recovery path | 2026-03-10 | A ClosedResourceError fix in the message-handler recovery path. Closed as a duplicate of #2072. |

## In progress

| Repo | Slug | Date | Note |
|---|---|---|---|
| leanprover-community/mathlib4 | `mathlib4-fermat-family` | 2026-09-17 | Two Lean theorems for the Fermat right-triangle family, kernel-clean against Mathlib. Paused on a Zulip sounding of #mathlib4 before a PR goes up. |

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
| **v2.2** (Jul 18) | Tree hygiene, commit voice, and scope checks (12 → 15 pre-submit) | Reviving PR #1134 I found a stray script and a lockfile I hadn't meant to touch sitting in my own tree. Added checks for the things I kept catching by eye. |
| **v2.3** (Jul 29) | Pipeline check blocks instead of warning; verify verdicts pin to a commit; `track.sh` reconciles against live GitHub | An audit of my own repo found every recent issue had skipped the pipeline entirely — the one check positioned to catch that only ever printed a warning. It also found a finished fix I'd never submitted and three PRs missing from my notes, because PR state lived in prose instead of being derived. |

---

_Generated 2026-09-17 by scripts/export-contributions.sh_
