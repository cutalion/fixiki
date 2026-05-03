# fixiki

A universal multi-agent engineering team for [Claude Code](https://claude.ai/code). Drop it into any repo and invoke `/team <task>` — seven specialist agents coordinate to ship a feature branch from a fuzzy ask.

Named after the [Fixies](https://en.wikipedia.org/wiki/Fixies_(TV_series)) — small creatures who fix things.

## What it does

```
/fixiki:team init                  Scaffold .ai_team/ in your repo (one-time setup)
/fixiki:team <task>                Run the team on a free-form task
/fixiki:team status                Print current state and recent log
```

The current Claude Code session becomes the **lead**. It reads your charter, decides which agents to dispatch, and synthesizes one coherent report back to you.

## The team

| Agent | Role |
|---|---|
| `team-analyst` | Turns a fuzzy ask into a clear spec with acceptance criteria |
| `team-planner` | Turns a spec into a numbered, TDD-shaped implementation plan |
| `team-plan-critic` | Reviews the plan before any code is written |
| `team-engineer` | Implements on a feature branch using TDD |
| `team-code-critic` | Reviews the implementation for bugs, style, and consistency |
| `team-qa` | Runs project-appropriate test + lint gates |
| `team-writer` | Updates docs and drafts PR descriptions |

The lead orchestrates them in sequence, runs review loops (up to 2 rounds each), and escalates to you when stuck.

## Installation

In a Claude Code session, install the plugin from GitHub:

```
/plugin install https://github.com/cutalion/fixiki
```

Then scaffold the team state in your project:

```
/fixiki:team init
```

This creates `.ai_team/charter.md` with auto-detected project name, language, test command, and lint command. Edit the charter to set your goals and authority level before running the team on a task.

## Charter

`.ai_team/charter.md` is read on every invocation. Key settings:

```markdown
## Goals
1. Ship feature X by date Y

## Authority
- Default scope: branch-only   # engineer works on feature branches only
# Change to `broad` to allow auto-merging low-risk classes

## Project conventions
- Language: JavaScript (Node.js)   # auto-detected; override if wrong
- Test command: npm test
- Lint command: npm run lint
```

## How a session works

1. You run `/fixiki:team add dark mode to the settings page`.
2. The lead reads `.ai_team/charter.md` and the project structure.
3. It dispatches: analyst → planner → plan-critic → engineer → code-critic → qa → writer.
4. Each agent that returns `request-changes` triggers one re-dispatch of the upstream agent (2-round limit).
5. You get one synthesis: branch name, test results, QA verdict, and what to do next.

## Project state

`.ai_team/` travels with the repo:

```
.ai_team/
├── charter.md   — you edit this
├── state.yml    — the lead edits this
├── log/         — one file per session
└── specs/       — analyst-authored mini-specs
```

## Roadmap

- **Phase 1 (current):** sync local mode — you wait at the terminal.
- **Phase 2:** Linear integration — `/team LIN-123` pulls issue body automatically.
- **Phase 3:** Autonomous cron mode — `/team start` runs the team on a schedule via `/schedule`.
- **Phase 4:** Escalation via Gmail for P0 issues.

## License

MIT.
