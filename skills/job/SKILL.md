---
name: job
description: Use to switch the active job file or create one manually — reads/creates jobs/[slug].md, refreshes TodoWrite from Progress, announces the active job. Auto's Decompose step creates jobs automatically for complex requests; this is the manual switch.
---

# job

Switch the active job file. Auto creates populated jobs automatically via the Decompose step (Step 0) when a request is complex — including optional delegation lanes for orchestrated work. This skill is for switching to an existing job or creating one manually.

Behavior:
1. Parse the arguments for a slug. If none is provided, ask for one.
2. Check whether `jobs/[slug].md` already exists.
3. If it exists:
   - Read it.
   - Report current state: Summary, Progress, open Issues.
   - Refresh TodoWrite from `## Progress`.
   - Announce active job.
4. If it does not exist:
   - Ensure `jobs/` directory exists.
   - Create `jobs/[slug].md` from the job template (resolve `<plugin-root>/assets/job-template.md` — the plugin root is derivable from this skill's own file path or stated in the always-on session injection; `jobs/` itself lives in the user's project cwd).
   - Replace the placeholder slug in `# job-slug`.
   - Seed `## Summary` from any context the user provided.
   - Note: Auto's Decompose step creates populated jobs during complex requests — a blank template is only a manual fallback. If you're mid-request, let Auto decompose instead.
   - Refresh TodoWrite from `## Progress` — Phase 1 becomes active.
   - Leave `## Delegation Plan` and `## Subagent Receipts` empty unless the current request needs non-trivial delegation; existing job files without those optional sections remain valid.
   - Announce new job created.
5. Do not run an optimization preflight. Jobs start light and grow.
6. Do not create branches, worktrees, or workspace metadata. Jobs are file-only.

Finish with a compact status block:
- Job: `jobs/[slug].md`
- Status: new / existing
- Progress summary
- Next recommended action
