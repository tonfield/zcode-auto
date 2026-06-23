# [Project Name]

## Project Overview
- [what this repo is, stack, runtime, purpose]

## Project Structure
- `[path]` - [what lives here]
- `jobs/` - job files (one per job, tracks research, design, progress, closeout)
- `memory/` - shared project knowledge base (gotchas, decisions)

## Commands
- Dev: `[command]`
- Test: `[command]`
- Lint: `[command]`

## Conventions
- [repo-specific coding, naming, architectural conventions]

## Verification
- [how to verify changes in this repo]

## Gotchas
- [setup quirks, environment assumptions, easy-to-miss constraints]

## Safe Parallelism
- Read-only delegation lanes: [safe areas personas may inspect in parallel via Explore]
- Safe disjoint edit lanes: [paths/contracts that can be edited independently]
- Keep serialized: job files, shared config, package/lock files, migrations, generated artifacts, and other high-coupling surfaces

## Job System
- Non-trivial work uses job files at `jobs/[slug].md`. Auto creates them via Decompose (Step 0) for complex requests; `/job <slug>` switches between existing jobs.
- Job files use Protocol v2: Baseline with Protocol version, Receipts (every decision traces to a source), optional Delegation Plan for non-trivial delegation lanes, phase-grouped Progress (each phase has a pass condition), optional Subagent Receipts for accepted delegated outputs, and Closeout disclosure. Completed jobs move to `jobs/archive/`; closeout consolidates reusable lessons to `memory/` with provenance checked.
- The Auto orchestrator (see the zcode-auto plugin) follows the verification protocol: clarify, establish baselines, decompose work into safe lanes, delegate read-only lanes in parallel via Explore personas, design bounded edits via the patch-planner persona (parent applies via Edit), verify persona output before accepting it, report deltas, tag claims `[verified]`/`[assumed]`, check impact, invoke the reviewer persona (every code-change turn and job Phase 4 Verify), run adversarial review at verification boundaries for completed durable work, invoke the goal-evaluator persona (every non-urgent turn), close with a disclosure. Depth scales naturally with the work. Urgent production incidents use the compressed incident path documented in the plugin's full protocol (read via `/using-auto` or from the plugin's `assets/auto-protocol.md`).
- Auto also applies two continuous lenses alongside the cycle — critical thinking (question the premise, surface uncertainty and trace claims, steel-man the opposite, pre-mortem non-trivial actions) and simplicity (stop at the first rung that holds, prefer deletion over addition, match the effort to the work). Both run inside the existing steps, not as new gates; both are lazy in their own domain, never about correctness.
