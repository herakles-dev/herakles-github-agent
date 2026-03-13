# Contributions

Live tracker of PRs submitted using the Hercules Agentic Architecture.

## Open PRs

| Date | Repo | PR | Issue | Description | Pipeline |
|------|------|----|-------|-------------|----------|
| 2026-03-07 | modelcontextprotocol/python-sdk | [#2241](https://github.com/modelcontextprotocol/python-sdk/pull/2241) | [#2236](https://github.com/modelcontextprotocol/python-sdk/issues/2236) | pywin32 soft imports for Windows process management | v1 (8+10 gates) |
| 2026-03-07 | modelcontextprotocol/python-sdk | [#2242](https://github.com/modelcontextprotocol/python-sdk/pull/2242) | [#2233](https://github.com/modelcontextprotocol/python-sdk/issues/2233) | ClosedResourceError in _handle_message | v1 (8+10 gates) |
| 2026-03-07 | modelcontextprotocol/inspector | [#1132](https://github.com/modelcontextprotocol/inspector/pull/1132) | [#1131](https://github.com/modelcontextprotocol/inspector/issues/1131) | Handle tools/resources/prompts list_changed notifications | v1 (8+10 gates) |
| 2026-03-09 | modelcontextprotocol/inspector | [#1134](https://github.com/modelcontextprotocol/inspector/pull/1134) | [#1077](https://github.com/modelcontextprotocol/inspector/issues/1077) | Fix task polling blocking the Run button | v1 (8+10 gates) |
| 2026-03-07 | kubeflow/docs-agent | [#118](https://github.com/kubeflow/docs-agent/pull/118) | [#76](https://github.com/kubeflow/docs-agent/issues/76) | CORS wildcard -> env-var allowlist | v2 (8+12+agents) |

## Pending Submission

| Date | Repo | Issue | Description | Pipeline | Status |
|------|------|-------|-------------|----------|--------|
| 2026-03-13 | modelcontextprotocol/inspector | [#873](https://github.com/modelcontextprotocol/inspector/issues/873) | Add 5 phantom runtime dependencies for strict package managers | v2 (8+12+6 agents) | Awaiting human approval |

## Merged PRs

| Date | Repo | PR | Issue |
|------|------|----|-------|
| — | — | — | (tracking) |

## Pipeline Versions

| Version | Gates | Agent Verification | Email Approval | First Used |
|---------|-------|--------------------|----------------|------------|
| **v1** | 8 pre-flight + 10 pre-submit | No | No | 2026-03-07 |
| **v2** | 8 pre-flight + 12 pre-submit | 5-7 parallel agents with problem-solving frameworks | IONOS SMTP gate | 2026-03-13 |

## Stats

- **Total PRs submitted**: 5
- **PRs pending submission**: 1 (inspector #873, awaiting approval)
- **PRs merged**: 0
- **PRs open**: 5
- **PRs rejected**: 0
- **Repos contributed to**: 3 (python-sdk, inspector, docs-agent)
- **Organizations**: 2 (modelcontextprotocol, kubeflow)
- **Issues verified with agent formation**: 2 (docs-agent #76, inspector #873)
- **CVEs independently verified**: 5 (express-rate-limit GHSA-46wh-pxpv-q5gq, minimatch x3, serve-handler transitive)

---

*Updated: 2026-03-13*
