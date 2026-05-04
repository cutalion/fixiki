---
description: Universal AI engineering team. `/fixiki:team init` to scaffold .ai_team/. `/fixiki:team <task>` to run the team in sync mode against a free-form task. `/fixiki:team status` to print state.
---

# /fixiki:team

You have been invoked as the **lead** of the universal AI engineering team.

**Arguments received:** `$ARGUMENTS`

## Step 1: Parse the subcommand

Look at `$ARGUMENTS`:

| Arg pattern | Subcommand |
|---|---|
| empty / `help` | Print usage (see Step 5) and stop. |
| exactly `init` | Run **init** flow (Step 2). |
| `init <anything else>` | Print "init takes no arguments" and stop. |
| exactly `status` | Run **status** flow (Step 3). |
| `status <anything else>` | Print "status takes no arguments" and stop. |
| exactly `start` / `stop` / `run` | Print "Phase 3 — not yet shipped. See `docs/superpowers/specs/2026-05-01-ai-team-design.md` §8." and stop. |
| matches `^LIN-\d+$` | Print "Phase 2 — Linear integration not yet shipped. Pass the issue body inline as `/fixiki:team <body>` for now." and stop. |
| starts with `LIN-` but doesn't match `^LIN-\d+$` (e.g., empty or non-numeric) | Print "Pass a Linear issue id, e.g. `/fixiki:team LIN-123`" and stop. |
| starts with `--async ` | Treat the rest as a free-form task; pass `mode=async` to the lead. Run **task** flow (Step 4). |
| exactly `--async` | Print "usage: /fixiki:team --async <task>" and stop. |
| exactly `curate` | Run **curate** flow (Step 3.5). |
| `curate <anything else>` | Print "curate takes no arguments" and stop. |
| anything else | Treat as a free-form task; pass `mode` per Step 4 detection. Run **task** flow (Step 4). |

## Step 2: `init` flow

1. Refuse if `.ai_team/charter.md` already exists. Print "already initialized" and `cat .ai_team/charter.md | head -20`.
2. Otherwise:
   - Create `.ai_team/`, `.ai_team/log/`, `.ai_team/specs/`.
   - Auto-detect:
     - **Project name:** from `package.json` `name`, or `Cargo.toml` `name`, or git repo root dirname.
     - **Language:** from manifest (lockfile most authoritative).
     - **Test command:** `npm test` / `bundle exec rspec` / `pytest` / `go test ./...` / `cargo test` (whichever fits language).
     - **Lint command:** project-appropriate.
     - **Project shape** (used to seed the Definition of Done):
       - **Web service** → presence of `Procfile`, OR `package.json` `scripts.start`, OR a Gemfile referencing `sinatra`/`rails`/`rack`/`hanami`, OR `pyproject.toml`/`requirements.txt` referencing `flask`/`fastapi`/`django`, OR `go.mod` with a `main.go` that imports `net/http`/`gin`/`echo`. Smoke command default: the detected start command + a `curl http://localhost:<port>/` probe expecting HTTP 2xx/3xx.
       - **CLI** → presence of a `bin/` directory with executables, OR `package.json` `bin` field, OR `pyproject.toml` `[project.scripts]`, OR `Cargo.toml` `[[bin]]`. Smoke command default: invoke the binary with `--help` (or `--version`) in a clean subshell, assert exit 0.
       - **Library / module** (default fallback) → smoke command default: a one-liner that requires/imports the package's public entry point in a fresh process and exits cleanly.
       - **Background job / worker** → presence of `Sidekiq`, `Resque`, `Celery`, `Bull`, etc. Smoke command default: boot the worker process, assert it stays up for N seconds without crashing.
     - **Docs root:** first existing of `docs/`, `doc/`, `documentation/`. Fallback: `.ai_team/docs/`.
     - **ADR path:** first existing of `**/adr/`, `**/decisions/`, `**/ADRs/` (anywhere in repo, but prefer one under the docs root). Fallback: `<docs-root>/adr/`.
     - **Design path:** `<docs-root>/design/`.
     - **Contracts path:** if `openapi.yaml`, `openapi.json`, `schema.graphql`, `proto/`, `*.proto`, or `*.thrift` is present anywhere in repo, set to `<docs-root>/contracts/`. Else `none`.
     - **Usage path:** existing `<docs-root>/usage/` if present, else `none` (only `README.md` is the user-facing surface).
     - **Changelog:** existing `CHANGELOG.md` or `CHANGELOG.rst` at repo root, else `none`.
   - Determine `{{SMOKE_GATE_AUTODETECTED}}` from the detected project shape:
     - Web service: `App starts via \`<start-command>\` and \`curl http://localhost:<port>/\` returns HTTP 200.`
     - CLI: `Binary \`bin/<name> --help\` runs in a clean shell and exits 0 with usage output.`
     - Library: `\`<interpreter> -e "require '<package>'" \` succeeds in a fresh process.`
     - Worker: `Worker starts via \`<start-command>\` and stays up for 5s without crashing.`
   - Determine `{{DOCS_ROOT_AUTODETECTED}}`, `{{ADR_PATH_AUTODETECTED}}`, `{{DESIGN_PATH_AUTODETECTED}}`, `{{CONTRACTS_PATH_AUTODETECTED}}`, `{{CHANGELOG_PATH_AUTODETECTED}}`, `{{USAGE_PATH_SUFFIX}}` (e.g., ` plus <docs-root>/usage/` if usage path detected, else empty), and `{{USAGE_PATH_HUMAN}}` (e.g., `<docs-root>/usage/` if detected, else `(none)`) from the auto-detection above.
   - Write `.ai_team/charter.md` with this content (substituting the placeholders):

     ```markdown
     # Team Charter — {{PROJECT_NAME}}

     > Edit this file. It is read on every `/fixiki:team` invocation. Stale charter = stale team behavior.

     ## Goals

     1. {{GOAL_PLACEHOLDER}}
     2. (add more — order matters; earlier = higher priority)

     ## Non-goals

     - (explicit non-goals; the team will refuse work that conflicts with these)

     ## Authority

     - **Default scope:** branch-only
     - **May open PRs:** yes
     - **May merge PRs:** no
     - **May modify CI config:** no
     - **May modify .ai_team/:** yes (it's the team's own state)

     To grant broader authority, change `branch-only` → `broad` and list auto-mergeable classes:

     ```
     broad: docs, dep-bumps-patch, comment-only-changes
     ```

     ## Cadence (Phase 3+ autonomous mode — ignored in Phase 1)

     - Default: hourly during 09:00-19:00 weekdays, project-local timezone
     - Override per-task with Linear label `cadence:N` (minutes)

     ## Escalation

     - **Default channel:** Linear (set issue status to `Waiting for Input`) — ignored in Phase 1; escalations surface to terminal.
     - **P0 channel:** unset (Phase 4 will add Gmail).

     ## Project conventions

     - **Language:** {{LANG_AUTODETECTED}}
     - **Test command:** {{TEST_CMD_AUTODETECTED}}
     - **Lint command:** {{LINT_CMD_AUTODETECTED}}
     - **Branch prefix:** `ai-team/`

     (Override any of these if auto-detection got them wrong.)

     ## Definition of Done

     > Every change must satisfy these gates before being declared done. Edit the smoke command — auto-detection is a starting point, not the final answer.

     - **Tests pass:** `{{TEST_CMD_AUTODETECTED}}`
     - **Linter clean:** `{{LINT_CMD_AUTODETECTED}}`
     - **Smoke-invocation:** {{SMOKE_GATE_AUTODETECTED}}

     The smoke-invocation gate boots the artifact in a fresh subprocess (clean shell — not in-process via the test runner) and exercises its user-invocation surface. This catches manifest gaps, boot-path regressions, and misconfiguration that in-process tests cannot see.

     ## Notes

     (Anything else the team should know — house style, taboo refactors, hot files, weak spots in the test suite.)

     ## Documentation conventions

     > Where this project keeps each doc type. Auto-detected at init.
     > If a path is `none`, the team treats that doc type as "not used in this project"
     > and skips agents/ceremony that would produce it.

     - **Docs root:** {{DOCS_ROOT_AUTODETECTED}}
     - **ADRs:** {{ADR_PATH_AUTODETECTED}}
     - **Design docs:** {{DESIGN_PATH_AUTODETECTED}}
     - **API contracts:** {{CONTRACTS_PATH_AUTODETECTED}}
     - **Domain rules:** none
     - **User-facing docs:** README.md{{USAGE_PATH_SUFFIX}}
     - **Changelog:** {{CHANGELOG_PATH_AUTODETECTED}}
     - **Specs (team-internal):** .ai_team/specs/

     ## Documentation governance

     > Defaults — override any line below.

     - ADRs are immutable + supersedable
     - Specs are mutable while open, frozen at PR merge
     - Design docs are versioned per feature
     - API contracts are versioned (semver-style for breaking changes)
     - Domain rules are living docs
     - Code comments follow code
     - READMEs/changelogs follow shipped behavior

     ## Session mode

     > Controls how the team runs by default. Values: sync (lead runs in this session, asks the user) | async (lead records decisions per the team-lead skill's rules).

     - **Default mode:** sync
     ```

   - Write `.ai_team/state.yml` with this content:

     ```yaml
     # Team state — managed by the lead. Edit only if you know what you're doing.
     schema_version: 1
     last_session: null
     current_focus: null
     in_flight: []
     escalations: []
     counters:
       sessions: 0
       dispatches: 0
     ```

   - Write `.ai_team/doc-conventions.md` with this content (substituting placeholders the same way as the charter):

     ````markdown
     # Documentation conventions

     This file codifies the team's doc taxonomy. Agents read it to know which doc types this project uses, where they live, and how they evolve. Edit `.ai_team/charter.md` to override paths or governance — this file is generated from the charter and the universal defaults.

     ## Taxonomy

     | Doc type | Location (this project) | Lifecycle | Source-of-truth rule | Owner |
     |---|---|---|---|---|
     | ADR | {{ADR_PATH_AUTODETECTED}} | Immutable + supersedable. Body never edited after acceptance; supersession adds a new ADR with `supersedes:` and the old one gets `superseded-by:`. | Body is truth as-of write date. Code contradicting an unsuperseded ADR raises a concern from code-critic. | architect drafts during design; writer finalizes after merge |
     | Spec | .ai_team/specs/<slug>.md | Mutable while work is open; frozen at PR merge. | While open: spec is truth, deviations require updating it in same PR. After merge: historical record. | analyst writes; engineer updates if scope shifts mid-task |
     | Design doc | {{DESIGN_PATH_AUTODETECTED}} | Versioned per feature (v1, v2); old versions retained. | Latest version is truth for that feature. | architect |
     | API contract | {{CONTRACTS_PATH_AUTODETECTED}} | Versioned semver-style; breaking changes bump version + add migration note. | Contract is truth for both producers and consumers. | architect drafts; engineer must bump version on breaking changes |
     | Domain / business rules | (charter `Domain rules` path) | Living, edited continuously. | Domain doc and code must agree; either may lead but divergence is a concern. | domain-modeler maintains; engineer flags discoveries |
     | Code comments / docstrings | inline | Follow code. | Code is truth; comment that disagrees is a bug to fix in same PR. | engineer writes; code-critic checks |
     | README / user guide / CHANGELOG | README.md / {{USAGE_PATH_HUMAN}} / {{CHANGELOG_PATH_AUTODETECTED}} | Follow shipped behavior, may lag merge by one writer pass. | What the user can actually do is truth. | writer |

     ## Frontmatter status

     ADRs, specs, design docs, and API contracts carry a `status` line in YAML frontmatter:

     ```yaml
     status: current
     # or
     status: superseded-by: <id>
     # or
     status: stale-pending-review
     ```

     Examples:

     ```yaml
     ---
     title: Use PostgreSQL for primary store
     status: current
     date: 2026-05-04
     ---
     ```

     ```yaml
     ---
     title: Use MySQL for primary store
     status: superseded-by: ADR-0017
     superseded-on: 2026-05-04
     ---
     ```

     ## Skipped doc types

     If a doc type's charter path is `none`, the team treats it as "not used in this project":
     - The corresponding agent is not dispatched (e.g., `none` for `Domain rules` → domain-modeler skipped).
     - Architect folds contract content into the design doc when contracts path is `none`.
     - Curator does not scan that doc type.

     ## When to update vs. when to follow

     - **Engineer follows the open spec.** Deviations require updating the spec in the same PR. If the engineer believes the spec is wrong, they raise a `spec-stale` concern; the lead routes it to the analyst.
     - **Architect and writer follow accepted ADRs.** Contradicting an unsuperseded ADR is a code-critic finding; the resolution is to write a new ADR superseding the old one.
     - **All agents flag stale docs they encounter.** A doc that references files/symbols that no longer exist is a `doc-conflict` concern raised to the lead.
     - **Curator marks rather than fixes.** It writes `status: stale-pending-review` and emits concerns; the lead routes the actual content fix to writer or architect.
     ````

   - Write `.ai_team/README.md` with this content:

     ```markdown
     # .ai_team/

     This directory is the per-project state of the universal AI engineering team. It travels with the repo.

     ## Layout

     - `charter.md` — **You edit this.** Goals, non-goals, authority, cadence, conventions.
     - `state.yml` — **The lead edits this.** Current focus, in-flight tasks, escalations.
     - `doc-conventions.md` — Generated reference for the team's doc taxonomy. Edit `charter.md` to override paths or governance; this file is regenerated on `init`.
     - `log/` — One markdown file per `/fixiki:team` session. Append-only history.
     - `specs/` — Mini-specs the analyst writes for non-trivial tasks.

     ## How to invoke the team

     In a Claude Code session (with the fixiki plugin installed):

     ```
     /fixiki:team init                    # one-time: scaffold this directory (already done if you're reading this)
     /fixiki:team <free-form task>        # sync mode: lead runs in this session, reports back
     /fixiki:team status                  # print state
     ```

     ## How the team operates (Phase 1)

     1. You invoke `/fixiki:team <task>`.
     2. The current Claude Code session becomes the **lead**.
     3. The lead reads this directory + project context.
     4. The lead dispatches some subset of: analyst → planner → plan-critic → engineer → code-critic → qa → writer.
     5. The lead synthesizes results into one report and writes a log entry here.

     ## Phase 2+ (not yet shipped)

     - Linear integration (`/fixiki:team LIN-123`)
     - Autonomous cron-driven mode (`/fixiki:team start`)
     - PR-only output, escalation via Linear comments
     ```

3. Ask the user one question: "What's the team's first goal? I'll write it into `charter.md`. (Type 'skip' to leave the placeholder.)" — incorporate the answer (or skip).
4. Print a short summary of what was created and tell the user to review `.ai_team/charter.md` before invoking `/fixiki:team <task>`. Specifically call out the `Definition of Done` smoke command — auto-detection is a guess; the user should refine it to match their actual user-invocation surface before the team runs its first task.
5. **Do not run any other agents.** Init is setup-only.

## Step 3: `status` flow

1. If `.ai_team/charter.md` doesn't exist, print "not initialized — run `/fixiki:team init` first" and stop.
2. Print:
   - First 5 lines of `.ai_team/charter.md` (project name + first goal).
   - `.ai_team/state.yml` (verbatim).
   - Tail (3 most recent) of `.ai_team/log/` filenames + their first 3 lines each.
   - **Drift summary** — dispatch `team-doc-curator` in `summary-only` mode (one-line counts, no per-doc detail). Print the returned line. If charter declares no doc paths beyond `.ai_team/specs/`, print `docs: (no doc paths declared in charter)` instead of dispatching.
3. Stop.

## Step 3.5: `curate` flow

1. If `.ai_team/charter.md` doesn't exist, print "not initialized — run `/fixiki:team init` first" and stop.
2. Load the `fixiki:team-lead` skill.
3. Dispatch `team-doc-curator` only (no other agents). Pass it the charter's `## Documentation conventions` section.
4. Print the curator's drift report verbatim.
5. Write a one-line entry to `.ai_team/log/<YYYY-MM-DD>-curate.md` recording the dispatch and the headline drift counts.
6. Stop. Do not dispatch any other agent.

## Step 4: `task` flow — become the lead

1. **Load the `fixiki:team-lead` skill** by invoking it. The skill teaches you the orchestration playbook (rules, dispatch heuristic, context-loading, state, logging, synthesis).
   - If for any reason the skill cannot be loaded, refuse to proceed with the task. Tell the user.
1.5. **Mode determination.** Before dispatching anything, the team-lead skill computes `mode` (sync | async) from: (a) `--async` flag in `$ARGUMENTS`, (b) env var `AI_TEAM_MODE`, (c) charter `## Session mode` setting (`Default mode:`), (d) inference from the user's task text. The lead announces the chosen mode. Pass `mode` into every subsequent dispatch.
2. **Verify charter exists.** If `.ai_team/charter.md` is missing, print "run `/fixiki:team init` first" and stop.
3. **Follow the team-lead skill's playbook** with the task = `$ARGUMENTS`.
4. **Synthesize** results to the user per the skill's synthesis template.
5. **Write the log entry** per the skill's logging spec.

## Step 5: Usage (printed on `/fixiki:team` or `/fixiki:team help`)

```
/fixiki:team init                  Scaffold .ai_team/ in this repo (one-time)
/fixiki:team <free-form task>      Run the team on the task in sync mode
/fixiki:team --async <task>        Run the team on the task in async mode (no human in loop)
/fixiki:team status                Print current state, recent log, and a doc drift summary
/fixiki:team curate                Run the doc-curator only and print a drift report
/fixiki:team help                  This message

Phase 3+ (not yet shipped):  /fixiki:team start | /fixiki:team stop | /fixiki:team run
Phase 2 (not yet shipped):   /fixiki:team LIN-<id>
```

## Hard rule

The `/fixiki:team` command **never writes code** to files outside `.ai_team/`. Code edits go through `team-engineer` only. (Init writing template files into `.ai_team/` is allowed; init never touches source code.)
