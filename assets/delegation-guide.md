# Delegation Guide

How Auto invokes each persona in zcode-auto. ZCode has no first-class subagent types, so each persona is an **Explore call in a fresh context**, with the persona body + delegation packet as the prompt. That fresh context is the independence property that breaks self-review's ceiling; Explore's independent Read/Grep/Glob is what lets adversarial-reviewer re-verify against ground truth rather than trusting a summary.

## Canonical invocation

1. **Read** `personas/<name>.md` to get the current persona body (keeps the prompt version-faithful and lets the persona evolve without rewriting the protocol).
2. Build the delegation packet (below).
3. Call `Agent(subagent_type: "Explore", prompt: <persona body>\n\n---\n\n# Delegation packet\n\n<packet>)`.
4. Inspect the return: `Status`, `Scope covered`, `Summary`, `Recommended next action` (for `patch-planner`, also `Actions taken` and `Verification run`). Reject out-of-scope results. Record accepted non-trivial results in `## Subagent Receipts`.

### Standard delegation packet

```text
id: <stable lane id>
objective: <single outcome>
scope: <files/surfaces/question boundaries>
allowed_paths: <boundaries — patch-planner designs within these>
forbidden_paths: <boundaries>
dependencies: <lane ids or none>
gate: <observable check or evidence needed>
return: Status / Scope covered / Summary / Evidence / Open questions / Recommended next action (+ Actions taken / Verification run for patch-planner)
```

### Parallelism

Read-only lanes may run in parallel — issue **multiple Agent calls in a single message** (ZCode runs them concurrently). Serialize write-back work by default; the only "write" persona is `patch-planner`, which designs but does not apply — Auto applies every Edit itself, so concurrent patch-planner lanes are safe as long as their `allowed_paths` are disjoint (record the reasoning in `## Delegation Plan`).

## Per-persona recipes

### repo-search
- **When:** on demand — explore codebase, find patterns, locate definitions/callers.
- **Provide:** specific question + search boundaries (paths, languages, what counts as an answer).
- **Return:** Status / Scope covered / Summary / Evidence (file:line + snippets) / Open questions / Recommended next action.

### docs-research
- **When:** on demand — look up docs/APIs, resolve conflicting or version-sensitive claims.
- **Provide:** exact question + version/date constraints + any conflicting claims to adjudicate.
- **Return:** Status / Scope covered / Summary / Evidence (URLs + decisive quotes) / Open questions / Recommended next action.
- **Note:** this is the only persona with WebSearch/WebFetch access.

### impact-mapper
- **When:** on demand before design/closeout for non-trivial interface/protocol/config changes; or when Step 6 (Check impact) needs an independent sweep.
- **Provide:** target change + suspected surfaces (callers, caches, persisted state, docs, configs).
- **Return:** Status / Scope covered / Summary / Impact map / Safe parallelism / Evidence / Open questions / Recommended next action.

### test-strategist
- **When:** on demand when verification gates are unclear, or before risky build work; or when Baseline needs a gate recommendation.
- **Provide:** job/slice goal + repo commands + changed surfaces.
- **Return:** Status / Scope covered / Summary / Recommended gates / Evidence / Coverage gaps / Open questions / Recommended next action.
- **Note:** it recommends; Auto runs the chosen gate.

### patch-planner
- **When:** on demand when one bounded code/file change is worth designing in a fresh context (e.g., a tricky refactor, a change where you'd benefit from independent eyes on the exact diff).
- **Provide:** exact change spec + `allowed_paths` + `forbidden_paths`.
- **Return:** Status (ready-to-apply | needs-context | blocked) / Scope covered / Summary / Patches (ordered, each with file/operation/old_string/new_string/rationale) / Actions taken / Verification run / Open questions / Recommended next action.
- **Apply:** for each patch, call Edit (`old_string` → `new_string`) or Write (for `operation: write`). Edit's exact-unique-match is a free correctness check — a failed match means the persona's `old_string` drifted from the file; re-Read and re-delegate rather than forcing it. After applying, run the slice gate yourself.

### reviewer
- **When:** **every code-change turn**; and job Phase 4 Verify before adversarial review.
- **Provide:** mode context ("job" | "explicit") + resolved target (files, diff, topic) + supporting context (job `## Design`, `## Research` when relevant) + whether to write findings to `## Issues` + risk profile (LOW/MEDIUM/HIGH, default MEDIUM).
- **Return:** Status / Scope covered / Lenses used / Executive summary / Severity summary / Findings (table) / Job issue updates / Recommended next action.
- **Act:** fix accepted findings, re-verify after each fix.

### adversarial-reviewer
- **When:** mandatory on the default non-urgent path at job Phase 4 Verify and at no-job durable-change closeout; on demand earlier when the standard reviewer may have missed blind spots.
- **Provide:** target files/diff + job `## Design`, `## Research`, `## Receipts` + any verification output already run.
- **Return:** Status / Scope covered / Summary / Findings (numbered: trap · proof · severity · fix) / Job issue updates / Recommended next action.
- **Act:** fix accepted findings, re-verify. If `Status: passed`, proceed.

### goal-evaluator
- **When:** **every non-urgent turn** (Step 7).
- **Provide:** Request (original user request or job `## Goal` condition) + evaluation mode (`active_item` | `phase_pass` | `job_goal`) + what was done + supporting context/evidence.
- **Return:** Status (FULFILLED | NOT FULFILLED | BLOCKED) / Requirements met / Requirements unmet / Scope creep / Next action.
- **Act:** `NOT FULFILLED` for active item/phase/no-job → restart at step 2. Overall job-goal `NOT FULFILLED` before Phase 5 Close → continue to Step 8 with the verdict, do not write `## Closeout`. `FULFILLED`/`BLOCKED` → continue.
- **Note:** pure-model — it declines to use Read/Grep/Glob even though Explore offers them; its independence is judgment-only.

## Verifying persona output before trusting it

A persona's `Status: passed` or `Status: ready-to-apply` is a claim, not a result. Before accepting:

- **reviewer / adversarial-reviewer:** you don't need to re-run their read, but you *do* own the fix-and-re-verify loop for any finding you accept.
- **patch-planner:** apply the patch via Edit, then run the slice gate yourself. A failed Edit match is information — re-Read and re-delegate.
- **goal-evaluator:** its verdict drives the cycle; act on it as specified above.
- **repo-search / docs-research / impact-mapper / test-strategist:** they return evidence; you turn it into plans, receipts, and gates. Spot-check load-bearing claims if a finding seems surprising.

Record accepted non-trivial results in `## Subagent Receipts` with the parent-run verification.
