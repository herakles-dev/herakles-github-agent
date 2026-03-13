# Human-in-the-Loop

## Let Me Be Direct

I'm D. Michael Piscitelli ([herakles-dev](https://github.com/herakles-dev)). I built this system, and I make every decision that matters. This is not a bot. There is no "submit PRs while I sleep" mode. The pipeline literally cannot push code without my explicit email approval.

I'm writing this page because AI-assisted open source contributions have a trust problem, and I think being transparent about exactly where the human is in the loop is how you earn that trust back.

## Where The Pipeline Forces Me To Stop

The orchestrator has three hard stops. Not "the human can optionally review" — the script exits and nothing happens until I act.

### Stop 1: Before I Start Coding

The system has checked the issue, gathered context, detected the project's rules, and scaffolded the artifacts. Now it stops and waits for me.

This is where I decide: Is this issue actually worth pursuing? The scoring system said yes, but scoring can't capture everything — maybe the maintainer has been hostile to similar PRs, maybe the project is mid-rewrite and this code is getting deleted next week, maybe the issue is technically valid but politically charged.

I choose the approach here. I document what alternatives I considered and why I rejected them. Then I write the code.

### Stop 2: After The Agents Review

The code is written. 12 static checks passed. 5-7 agents reviewed the code and produced their verdicts. Now the pipeline stops again.

This is where I evaluate the agent findings. Are they right? Agents are useful but not infallible. They can produce false positives ("this introduces a vulnerability" when it doesn't) and false negatives (missing something obvious). On the Inspector PR, the supply chain agent said the fix version for a CVE was 8.3.1 — I sent 4 independent agents to verify and the actual minimum fix was 8.2.2. The agent was close but wrong on the specific version.

I also resolve disagreements here. On that same PR, the supply chain specialist said "bump the versions to fix CVEs" and the devil's advocate said "don't touch files you weren't asked to change." Both had valid points. I decided: keep the scope tight, note the CVEs in the PR description, submit a separate version bump later. Two contributions instead of one messy one.

### Stop 3: Before Anything Gets Pushed

The full diff, every agent finding, and the compliance summary arrive in my email. I read every line of code. I read every agent verdict. I reply "approved" — and only then does anything get pushed to GitHub.

This exists because I don't trust the process to be perfect. I trust it to catch most things. But my name is on the PR, and I want to know exactly what I'm submitting.

## What I Decide

**Which issues to work on.** The scoring system ranks them, but I pick based on strategic value, personal interest, and gut feeling about whether the maintainer will be receptive.

**How to fix them.** I consider multiple approaches, document the alternatives, and choose one. Sometimes the agents suggest a different approach during verification — I evaluate that and decide.

**Scope.** This is the hardest one. The supply chain agent on Inspector #873 found real CVEs. Should I fix them in the same PR? Expand the scope? The devil's advocate argued no. I agreed — but it was my call, informed by the agents' analysis.

**Whether agent findings are correct.** I override when I have domain knowledge they lack. I investigate when I'm not sure. I don't blindly trust PASS or blindly act on FAIL.

## What The System Handles

The system does the work I'd skip if I were in a hurry:

**Checking for competing PRs.** I'd probably glance at the issue and start coding. The system searches for open PRs, cross-references, comment claims, and assignees. It saved me from wasting time on already-claimed issues more than once.

**Detecting project conventions.** I'd probably skim CONTRIBUTING.md. The system parses it programmatically and catches things I'd miss — like specific trailer requirements or the fact that a project has a 100% coverage requirement.

**Scanning for secrets.** I'd like to think I'd never accidentally commit a key. But people do, all the time, and the regex patterns catch the common ones before they hit GitHub.

**Tracing imports.** On the phantom dependencies PR, the system traced every `import` in every bundled file to verify exactly which packages were missing. I could have done that manually — it would have taken 20 minutes and I might have missed one.

**Verifying CVEs are real.** AI agents sometimes hallucinate vulnerability IDs. The system cross-references against GitHub Advisory Database, npm audit, NVD, and Snyk. On the Inspector PR, it confirmed 5 CVEs across 5 independent sources. That's not something I'd do manually for every PR.

**Adversarial review.** I don't naturally think "why would a maintainer reject this?" The devil's advocate agent does, every time. It caught the prettier version bump I didn't notice. It flagged scope creep risk on the CVE fix. It's the most valuable agent in the formation.

## What This Is Not

This is not a system that generates PRs and hopes they stick. It's not a spray-and-pray approach. It's not autonomous.

It's a toolchain that makes the tedious parts of rigorous contribution automatic, so I can focus on the parts that require actual judgment: understanding the problem, choosing the right fix, and making good decisions about scope and approach.

The open source community is right to be skeptical of AI-assisted contributions. The way to earn that trust is not to hide the AI — it's to show that the human is making the decisions and the AI is doing the verification work that most contributors skip entirely.
