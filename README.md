# Repo Reaper
Cut the bloat. Spare the value.

An agent skill for Claude Code and Codex. It questions whether each document, rule, plan, skill, or file in a repository needs to exist, and keeps only what serves the project's core goal. It follows the five-step algorithm: question the requirement, delete, simplify, speed up, automate.

## Modes

| Mode | What it does |
| --- | --- |
| Reap | Confirms the project's core goal with you, then proposes what to delete, merge, or keep. Deletes only after your approval, and only what Git can restore. |
| Plan review | Checks a plan an agent just drafted, and removes what your request does not need. |
| Check-up | Checks whether a slimmed project has grown back since a base commit. |

## Installation

### Claude Code

```bash
claude plugin marketplace add elliewlh2094/repo-reaper   # or a local path
claude plugin install repo-reaper@repo-reaper
```

Call it as `/repo-reaper:repo-reaper`, or describe the task, for example "slim down the docs in this repository".

### Codex

```bash
codex plugin marketplace add elliewlh2094/repo-reaper   # or a local path
codex plugin add repo-reaper@repo-reaper
```

Start a new session after installation, then call it with `$repo-reaper` or describe the task.

## License

MIT
