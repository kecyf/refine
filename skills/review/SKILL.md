---
name: review
description: Verify a work session's output by delegating reviews to parallel subagents and synthesizing one prioritized verdict. Generalist — covers code, docs, content, config, or any deliverable, not just diffs. Companion to /plan. Use when the user invokes /review.
disable-model-invocation: true
---

# Review

You are the orchestrator running on an expensive frontier model. This is the companion to `/plan`: where plan specifies the work, review verifies it once the work is done. Your context and your reasoning are the scarce resources. Spend them on framing the review and synthesizing verdicts, never on reading every diff or document yourself.

The deliverable can be anything a session produces: code, documentation, marketing content, configuration, data transformations, migration scripts, a filled spreadsheet. The workflow is the same; only the review angles change.

## Workflow

### 1. Frame the review scope

Restate what was built this session and what "good" looks like. If a `/plan` plan drove the work, treat its tasks as the intent baseline (what *should* have changed) and its done criteria as the first checks to re-run. If the scope is ambiguous on points that change which checks matter, ask the user 1-4 sharp questions now (use the question tool) — never mid-run later.

### 2. Detect the blast radius

Find everything the session actually touched before reviewing. For git repos, a quick `git status --short` + `git diff --stat` across the relevant repos/worktrees (trivial lookups are fine to do yourself). For non-git deliverables (external docs, DB rows, published content), list what was created or modified from the session's own trail. Only areas with real changes get reviewed. Note the dominant change type per area — code, docs, content, config, data — it decides which angles apply.

### 3. Delegate the reviewing (parallel subagents)

Never read full diffs or documents yourself. Launch subagents in parallel, scoped per area with changes. Each prompt must include full standalone context (subagents see nothing of this conversation): the paths or artifacts to review, what the session was trying to do, and the required output — **a clear verdict with findings, evidence (file:line or precise location), and severity**.

Pick angles by change type; use dedicated review subagents when the environment provides them, a general readonly subagent otherwise:

| Angle | When |
|---|---|
| Bugs / correctness | any code change — logic, edge cases, error handling, broken imports |
| Security | anything touching auth, secrets, tokens, external input, DB writes, webhooks, permissions |
| Intent-fit + standards | every reviewed area — does the result match the plan/intent, and respect the project's own conventions (step 4) |
| Accuracy + consistency | docs and content — factual claims vs the code/data they describe, broken links, stale references, tone and terminology drift |
| Deep quality audit | optional — large or structural changes the user wants audited hard |

Code areas get correctness + intent-fit at minimum; add security when the diff touches a sensitive surface. Docs/content areas skip correctness/security — run intent-fit + accuracy instead.

### 4. Check the project's non-negotiables

Fold the project's own conventions into the intent-fit subagent prompts (or verify yourself if quick) — they are the rules generic reviewers miss. Pull them from the repo's `AGENTS.md` / `CONTRIBUTING` / equivalent docs: build & test gates, commit conventions, language and style rules, directory ownership ("right repo for the right work"), and any source-of-truth data that must never be edited directly. If the project documents a verification gate (test suite, build, lint), check it was run — or flag that it wasn't.

### 5. Synthesize one verdict

With all subagent verdicts in, do the synthesis yourself. Deduplicate overlapping findings, resolve contradictions (re-read the cited location yourself when two subagents disagree — targeted reads are verification, not exploration), and produce a single prioritized report — do NOT dump raw subagent output. Group by severity:

- 🔴 **Blocker** — must fix before commit/ship/publish (bugs, security, broken gate, intent mismatch, factually wrong content)
- 🟡 **Should fix** — quality, missing tests, standards violations, stale docs
- 🟢 **Nice to have** — optional polish

End with a clear go / no-go call per area and the shortest path to green. Flag any decision points left to the user. Do not fix anything yourself unless the user asks — review verifies; the user decides what to act on.
