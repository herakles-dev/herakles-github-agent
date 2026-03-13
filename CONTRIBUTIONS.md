# Contributions

Everything I've submitted, where it stands, and what I learned.

## The Numbers

**6 PRs submitted. 0 merged. 0 rejected.** All in review as of March 14, 2026. The framework is a week old — I'll update this as things move.

## Open PRs

### MCP Python SDK

**[#2241 — pywin32 soft imports](https://github.com/modelcontextprotocol/python-sdk/pull/2241)** (Mar 7)
Five-line fix for Windows process management. The SDK hard-imported `pywin32`, which isn't available on all platforms. Changed to soft imports with graceful fallbacks. Pipeline v1.

**[#2242 — ClosedResourceError in _handle_message](https://github.com/modelcontextprotocol/python-sdk/pull/2242)** (Mar 7)
Race condition where the message handler could access a closed resource. Required understanding the async lifecycle. CI was green after fixing a ruff formatting issue. Pipeline v1.

### MCP Inspector

**[#1132 — Handle list_changed notifications](https://github.com/modelcontextprotocol/inspector/pull/1132)** (Mar 7)
The inspector wasn't handling `tools/list_changed`, `resources/list_changed`, or `prompts/list_changed` MCP notifications. Added the handlers. Pipeline v1.

**[#1134 — Fix task polling blocking Run button](https://github.com/modelcontextprotocol/inspector/pull/1134)** (Mar 9)
The task polling mechanism was synchronous and blocked the Run button UI. Changed to non-blocking `pollTaskInBackground`. Pipeline v1.

**[#1144 — Add missing runtime dependencies for strict package managers](https://github.com/modelcontextprotocol/inspector/pull/1144)** (Mar 14)
Five phantom dependencies missing from root `package.json` that break pnpm and yarn PnP installations. Traced every import in every bundled file. First PR to go through the full V2 pipeline with voice review — 3 rounds of PR description refinement to avoid sounding like AI slop. Supply chain agent found 2 real CVEs; devil's advocate kept the scope tight. Pipeline v2.

### Kubeflow

**[docs-agent #118 — CORS wildcard to env-var allowlist](https://github.com/kubeflow/docs-agent/pull/118)** (Mar 7)
Fixed a wildcard CORS configuration (`*`) that allowed any origin. Replaced with an environment-variable-driven allowlist. This was the PR that taught me static checks aren't enough — three agents found the missing `ALLOWED_ORIGINS` env var in deployment.yaml and a wildcard bypass vulnerability that 12 bash checks missed. Led to building the V2 agent verification phase. Pipeline v2.

## What I've Learned So Far

**The python-sdk is intensely competitive.** Many issues have multiple people racing to submit fixes. The validate.sh pre-flight check has saved me from wasting time on claimed issues several times.

**The inspector is less contested and more welcoming.** Good first issues actually available. Maintainers are responsive. Node 22.7.5+ requirement means I can't run the full test suite locally (I'm on Node 20), but `npx tsc --noEmit --skipLibCheck` catches type errors.

**Kubeflow uses Prow bot and DCO sign-off.** Miss the `--signoff` flag and your PR is dead on arrival. The compliance matrix catches this automatically now.

**Prettier hooks surprise you.** On the Inspector phantom deps PR, the lint-staged pre-commit hook silently changed a prettier devDependency version during my commit. The conventions reviewer agent caught it. I wouldn't have noticed.

**AI agents hallucinate CVE details.** The supply chain agent said the fix version for express-rate-limit was `^8.3.1`. Four independent verification agents confirmed the CVE was real but the minimum fix was actually `^8.2.2`. Always verify. Never trust a single source.

**Your PR description is part of the contribution.** The first draft of the #1144 description scored 5/10 on voice authenticity — too many tables, preemptive FAQs, security-theater language ("trust boundaries"). Three rounds of voice review got it to 7-8/10. A 5-line diff doesn't need 40 lines of description. Match the weight of the words to the weight of the change.

## Pipeline Evolution

| Version | What Changed | Why |
|---------|-------------|-----|
| **v1** (Mar 7) | 8 pre-flight + 10 pre-submit checks | Starting point. Static analysis only. |
| **v2** (Mar 13) | 12 pre-submit + 5-7 verification agents + email approval | V1's static checks missed real issues on the Kubeflow PR. Added agents that actually reason about code. Added email gate because agents aren't perfect either. |
| **v2.1** (Mar 14) | Voice reviewer agent (Agent 6) | PR #1144's first draft scored 5/10 on authenticity. Added an agent that specifically checks if the PR description reads like a human wrote it. |

---

*Updated: 2026-03-14*
