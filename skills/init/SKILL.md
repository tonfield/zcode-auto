---
name: init
description: Use when bootstrapping or refreshing a project's AGENTS.md and memory/ knowledge base — scaffolds project structure docs, conventions, verification, gotchas, and the memory skeleton for cross-session knowledge.
---

# init

Bootstrap or refresh the current project's `AGENTS.md`, `memory/` knowledge base, and `.gitignore`.

Process:
1. Find project root. Check for existing:
   - `AGENTS.md`
   - `memory/README.md`
   - `memory/decisions.md`
   - `memory/gotchas.md`
   - `.gitignore`
2. Scan the repo lightly: package manifests, CI config, README, existing docs.
3. Ask only the smallest targeted questions needed to fill gaps the codebase can't answer.
4. Create or update `AGENTS.md` at project root. Use this shape:

```markdown
# [Project Name]

## Project Overview
- [what this repo is, stack, runtime]

## Project Structure
- `[path]` - [what lives here]

## Commands
- Dev: `[command]`
- Test: `[command]`
- Lint: `[command]`
- Build: `[command]`

## Conventions
- [repo-specific conventions]

## Verification
- [how to verify changes]

## Gotchas
- [pitfalls, invariants, constraints]

## Safe Parallelism
- Read-only delegation lanes: [safe areas personas may inspect in parallel via Explore]
- Safe disjoint edit lanes: [paths/contracts that can be edited independently]
- Keep serialized: job files, shared config, package/lock files, migrations, generated artifacts, and other high-coupling surfaces
```

5. Create or update `memory/` skeleton:
   - `memory/README.md` — doc map, reading order, authority
   - `memory/decisions.md` — decision log (edit in place; commit history preserves prior versions)
   - `memory/gotchas.md` — recurring pitfalls and invariants
   - Preserve existing project docs; make `memory/README.md` map to them.
6. Optionally: add managed `.gitignore` block for `jobs/` surfaces:

```gitignore
# BEGIN zcode-auto job surfaces
jobs/*
!jobs/
!jobs/*.md
!jobs/archive/
!jobs/archive/*.md
# END zcode-auto job surfaces
```

7. Summarize what was created or updated.

Rules:
- Write project-specific guidance first. Do not copy the global protocol wholesale; include only the concise job-system usage notes that help agents work in that project.
- Preserve user-authored content when updating.
- Keep files concise and practical.
- Document current truth, not speculative design.
- Include orchestration guidance only where useful: read-only/background delegation (Explore personas) can run in parallel; bounded edits are designed by `patch-planner` and applied by the main agent via Edit, with parent-owned verification.
- When documenting job files, note that `## Delegation Plan` and `## Subagent Receipts` are optional/backfilled sections for non-trivial delegated work.
