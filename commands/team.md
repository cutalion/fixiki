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
| anything else | Treat as a free-form task. Run **task** flow (Step 4). |

## Step 2: `init` flow

1. Refuse if `.ai_team/charter.md` already exists. Print "already initialized" and `cat .ai_team/charter.md | head -20`.
2. Otherwise:
   - Create `.ai_team/`, `.ai_team/log/`, `.ai_team/specs/`.
   - Auto-detect:
     - **Project name:** from `package.json` `name`, or `Cargo.toml` `name`, or git repo root dirname.
     - **Language:** from manifest (lockfile most authoritative).
     - **Test command:** `npm test` / `bundle exec rspec` / `pytest` / `go test ./...` / `cargo test` (whichever fits language).
     - **Lint command:** project-appropriate.
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

     ## Notes

     (Anything else the team should know — house style, taboo refactors, hot files, weak spots in the test suite.)
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

   - Write `.ai_team/README.md` with this content:

     ```markdown
     # .ai_team/

     This directory is the per-project state of the universal AI engineering team. It travels with the repo.

     ## Layout

     - `charter.md` — **You edit this.** Goals, non-goals, authority, cadence, conventions.
     - `state.yml` — **The lead edits this.** Current focus, in-flight tasks, escalations.
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
4. Print a short summary of what was created and tell the user to review `.ai_team/charter.md` before invoking `/fixiki:team <task>`.
5. **Do not run any other agents.** Init is setup-only.

## Step 3: `status` flow

1. If `.ai_team/charter.md` doesn't exist, print "not initialized — run `/fixiki:team init` first" and stop.
2. Print:
   - First 5 lines of `.ai_team/charter.md` (project name + first goal).
   - `.ai_team/state.yml` (verbatim).
   - Tail (3 most recent) of `.ai_team/log/` filenames + their first 3 lines each.
3. Stop.

## Step 4: `task` flow — become the lead

1. **Load the `fixiki:team-lead` skill** by invoking it. The skill teaches you the orchestration playbook (rules, dispatch heuristic, context-loading, state, logging, synthesis).
   - If for any reason the skill cannot be loaded, refuse to proceed with the task. Tell the user.
2. **Verify charter exists.** If `.ai_team/charter.md` is missing, print "run `/fixiki:team init` first" and stop.
3. **Follow the team-lead skill's playbook** with the task = `$ARGUMENTS`.
4. **Synthesize** results to the user per the skill's synthesis template.
5. **Write the log entry** per the skill's logging spec.

## Step 5: Usage (printed on `/fixiki:team` or `/fixiki:team help`)

```
/fixiki:team init                  Scaffold .ai_team/ in this repo (one-time)
/fixiki:team <free-form task>      Run the team on the task in sync mode
/fixiki:team status                Print current state and recent log
/fixiki:team help                  This message

Phase 3+ (not yet shipped):  /fixiki:team start | /fixiki:team stop | /fixiki:team run
Phase 2 (not yet shipped):   /fixiki:team LIN-<id>
```

## Hard rule

The `/fixiki:team` command **never writes code** to files outside `.ai_team/`. Code edits go through `team-engineer` only. (Init writing template files into `.ai_team/` is allowed; init never touches source code.)
