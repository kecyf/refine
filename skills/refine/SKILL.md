---
name: refine
description: Critique and tighten an existing plan so a cheaper executor model can run it without gaps — quality-bar audit plus a fresh-context cold read by a subagent. Companion to /plan. Use when the user invokes /refine on a plan draft, plan file, or the current Cursor plan.
disable-model-invocation: true
---

# Refine

You are the orchestrator running on an expensive frontier model. This is the companion to `/plan`: where plan produces a spec, refine hardens one that already exists — written by you earlier, by another agent, or by a human. Do not re-plan from scratch and do not execute anything; the deliverable is a tighter version of the same plan.

## Workflow

### 1. Locate the plan and its intent

Identify what you're refining: the current Cursor plan, a plan file, or a pasted draft. Restate in one or two sentences what the plan is trying to achieve — if that's not extractable from the plan itself, that is already finding #1 (an executor won't guess it either). If the user's goal and the plan visibly diverge, ask before tightening.

### 2. Audit against the quality bar

Read the plan yourself and judge every task against the bar an executor needs. The executor is the **weakest plausible model**: it has seen nothing of any prior session and is weak at filling gaps, recovering from ambiguity, or knowing when to stop.

- **Self-contained**: could a model that never saw the planning session execute each task with only the plan and the repo? Any "as discussed", implicit context, or missing file path fails.
- **Atomic**: one self-contained step per task, explicit file paths and symbol names, no judgment calls or re-discovery required.
- **Verification gates**: every mutating task ends with a command and its expected output; done criteria are machine-checkable, not "works correctly".
- **Hard boundaries**: explicit out-of-scope list, with one line on why each item must not be touched.
- **STOP conditions**: specific to this plan's actual risks, not boilerplate.
- **Drift stamp**: the plan records the commit it was written against; if the codebase moved since, check whether the plan's excerpts still match reality.
- **No secret values** anywhere in the plan.

Spot-check load-bearing claims against the codebase yourself (targeted reads are verification, not exploration): wrong file paths, stale excerpts, and guessed commands are the defects that kill executors.

### 3. Cold read by a fresh subagent

Self-critique misses gaps you mentally fill from context the executor won't have — especially if you authored the plan. Launch one fresh-context subagent on a cheap model, give it the plan text and nothing else, and ask it to report: every ambiguity, every task it could not execute without asking a question, every term or reference it cannot resolve, and every place it would have to guess. Its confusion is your defect list.

### 4. Tighten

Merge your audit and the cold read, then rewrite the plan yourself — same intent, same scope, harder spec. Fix by inlining missing context, splitting non-atomic tasks, adding gates and STOP conditions, and correcting drifted excerpts from your own reads. Do not silently change what the plan does; if you believe the approach itself is wrong, say so to the user instead of rewriting the intent.

End with a short changelog of what was tightened and why, and flag anything that still needs a decision from the user.
