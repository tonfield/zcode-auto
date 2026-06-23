# zcode-auto

Orchestration-first **Auto** protocol for [ZCode](https://z.ai) — phased work records, critical-thinking + simplicity lenses, a verification protocol, and independent-context personas invoked via the Explore subagent. A faithful port of [opencode-auto](https://github.com/tonfield/opencode-auto) to ZCode's plugin + skill + hook model.

## What this is

A plugin that turns ZCode into an orchestrator with a protocol instead of a one-shot assistant. On every request Auto runs an 8-step cycle — Decompose, Clarify, Baseline, Produce, Verify, Review, Check impact, Goal check, Disclosure — under two continuous lenses (**critical thinking**: question the premise, trace claims, steel-man, pre-mortem; **simplicity**: stop at the first rung that holds, prefer deletion over addition, match effort to work).

For non-trivial work, complex requests decompose into **job files** with phased progress and a verification boundary. The eight personas — adapted from opencode-auto's subagents — are invoked as independent Explore calls, so each runs in fresh context (the property that breaks self-review's ceiling) and can re-read files itself rather than trusting a summary.

## How it differs from opencode-auto

| Primitive | opencode-auto | zcode-auto |
|---|---|---|
| Always-on protocol | `mode: primary` agent | `sessionStart` hook injects the `using-auto` skill |
| Subagents | First-class `mode: subagent` with scoped `permission` blocks | Explore invocations with persona prompts (one fresh context per call) |
| Slash commands | `commands/*.md` | ZCode skills (`/job`, `/review`, `/init`) |
| Write subagent | `patch-implementer` executes bounded edits | `patch-planner` *designs* edits; main agent applies via Edit (exact-match = free verification) |
| `morph_edit` / ast-grep | Native MCP | Edit/Write; ast-grep degrades to structural Grep hints inside personas |

The protocol, lenses, claim tags, job-file system, baseline/delta, receipts, disclosure, and the cross-session memory architecture are unchanged — those are harness-agnostic.

## Install

### Dev load (recommended for now)

```bash
zcode --plugin-dir ~/projects/zcode-auto
```

Plugin discovery via the marketplace is documented but the live `enabledPlugins` config key isn't yet populated in the running build; `--plugin-dir` is the confirmed load mechanism.

### What gets activated

- **`sessionStart` hook** injects the compact `using-auto` protocol summary at every session start (Startup / Clear / Compact).
- **Skills** `/job`, `/review`, `/init` become available as slash commands.
- **Personas** and **assets** are loaded on demand (progressive disclosure).

## Layout

```
zcode-auto/
├── .zcode-plugin/plugin.json     manifest
├── hooks/                        sessionStart → inject using-auto
├── skills/                       using-auto, job, review, init (slash commands)
├── personas/                     8 roles, invoked via Explore
├── assets/                       auto-protocol, delegation-guide, job-template, agents-policy
└── project-template/             cp -r project-template/* . for downstream projects
```

## Using it in a project

```bash
cp -r ~/projects/zcode-auto/project-template/* /path/to/your/project/
```

That drops in a project `AGENTS.md`, a `memory/` skeleton (README/decisions/gotchas), a `jobs/` directory, and a fully-worked `SAMPLE-JOB.md` demonstrating the Protocol v2 structure.

## The personas

| Persona | Role | Fires |
|---|---|---|
| `repo-search` | Local repo discovery, compact evidence | On demand |
| `docs-research` | External docs / conflict resolution | On demand |
| `impact-mapper` | Change impact, old contracts, safe lanes | Before design / closeout |
| `test-strategist` | Recommends verification gates | When gates are unclear |
| `patch-planner` | Designs bounded edits (parent applies) | On demand |
| `reviewer` | Code review with 11 lenses | Every code-change turn + Phase 4 |
| `adversarial-reviewer` | Ground-truth scrutiny, tries to break it | Phase 4 + durable-change closeout |
| `goal-evaluator` | Pure-model completeness check | Every non-urgent turn |

See [`assets/delegation-guide.md`](assets/delegation-guide.md) for the Explore-invocation recipe per persona.

## Status

**v0.1.0** — initial port. Two honest unknowns:

1. **Hook output format under ZCode** — ZCode's hook env vars are undocumented in the cache; the hook follows superpowers' branching logic (Cursor → Claude Code → SDK-standard `additionalContext`). If the hook doesn't auto-inject, `using-auto` is manually invocable as `/using-auto`.
2. **`enabledPlugins` config key** — READMEs document it but the live config doesn't use it yet; `--plugin-dir` is the dev-load path.

## Credits

Protocol, lenses, and job system adapted from [opencode-auto](https://github.com/tonfield/opencode-auto). The simplicity lens adapts ideas from [ponytail](https://github.com/DietrichGebert/ponytail) (MIT).

## License

MIT — see [LICENSE](LICENSE).
