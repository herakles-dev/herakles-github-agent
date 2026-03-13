# Contributions

Everything I've submitted, where it stands, and what I learned.

## The Numbers

**5 PRs submitted. 0 merged. 0 rejected.** All still in review as of March 13, 2026. The framework is a week old — I'll update this as things move.

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

### Kubeflow

**[docs-agent #118 — CORS wildcard to env-var allowlist](https://github.com/kubeflow/docs-agent/pull/118)** (Mar 7)
Fixed a wildcard CORS configuration (`*`) that allowed any origin. Replaced with an environment-variable-driven allowlist. This was the PR that taught me static checks aren't enough — three agents found the missing `ALLOWED_ORIGINS` env var in deployment.yaml and a wildcard bypass vulnerability that 12 bash checks missed. Led to building the V2 agent verification phase. Pipeline v2.

## Pending

**Inspector #873 — Phantom runtime dependencies** (Mar 13)
Five dependencies missing from root `package.json` that break pnpm installations. Traced every import in every bundled file. 6 verification agents ran — supply chain specialist found 2 real CVEs (confirmed across 5 databases), devil's advocate argued to keep scope tight. Decided: ship scoped fix, note CVEs in PR description, follow up with version bump PR. Currently awaiting my email approval. Pipeline v2.

## What I've Learned So Far

**The python-sdk is intensely competitive.** Many issues have multiple people racing to submit fixes. The validate.sh pre-flight check has saved me from wasting time on claimed issues several times.

**The inspector is less contested and more welcoming.** Good first issues actually available. Maintainers are responsive. Node 22.7.5+ requirement means I can't run the full test suite locally (I'm on Node 20), but `npx tsc --noEmit --skipLibCheck` catches type errors.

**Kubeflow uses Prow bot and DCO sign-off.** Miss the `--signoff` flag and your PR is dead on arrival. The compliance matrix catches this automatically now.

**Prettier hooks surprise you.** On the Inspector phantom deps PR, the lint-staged pre-commit hook silently changed a prettier devDependency version during my commit. The conventions reviewer agent caught it. I wouldn't have noticed.

**AI agents hallucinate CVE details.** The supply chain agent said the fix version for express-rate-limit was `^8.3.1`. Four independent verification agents confirmed the CVE was real but the minimum fix was actually `^8.2.2`. Always verify. Never trust a single source.

## Pipeline Evolution

| Version | What Changed | Why |
|---------|-------------|-----|
| **v1** (Mar 7) | 8 pre-flight + 10 pre-submit checks | Starting point. Static analysis only. |
| **v2** (Mar 13) | 12 pre-submit + 5-7 verification agents + email approval | V1's static checks missed real issues on the Kubeflow PR. Added agents that actually reason about code. Added email gate because agents aren't perfect either. |

---

*Updated: 2026-03-13*
