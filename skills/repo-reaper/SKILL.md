---
name: repo-reaper
description: Cuts repository bloat by questioning whether each item needs to exist. Use when a project feels heavy, inconsistent, or hard to read because of excess documents, rules, plans, skills, or files, often produced by AI agents. Also use when reviewing a drafted plan for over-engineering or overly cautious work before implementation, or when checking whether a slimmed project has grown back. Triggers include slim down, prune, remove bloat, 瘦身, 精簡, 過度設計.
---

# Repo Reaper

Keep only what serves the project's core goal. Deleting is the default, and the item that stays carries the burden of proof. This follows the five-step algorithm: question the requirement, delete, simplify, speed up, automate. Run the steps in that order, because simplifying or automating something that should not exist only preserves the waste.

## Choose a mode

| Mode | Use when | Goal source |
| --- | --- | --- |
| Reap | One-time cut of a chosen scope | Confirmed project goal |
| Plan review | Check a drafted plan before implementation | The user's own task request |
| Check-up | Check whether a slimmed project grew back | Confirmed project goal |

Ask if the mode is unclear.

## Hard rules

- **Write only after approval.** Approval must cover the item, the action, and the reference fixes. A request to review is not approval to change. Do not stage, commit, or push unless asked.
- **Delete only what Git can restore.** An existing item must be tracked, committed, and unmodified. Leave uncommitted changes, untracked files, and ignored files alone. Planned items do not exist yet, so it skips them.
- **Honor the protect list.** Its items are not questioned.
- **Reap and Check-up need Git history** for recovery. Without it, stop.

Start with `git status --short`; list anything already changed.

## Question every item

An item stays only if it passes the test for its type. Name the test and evidence.

**Rules, processes, and guards need an owner.** The owner is the one who can be asked why the item exists. Fit with the goal does not replace an owner.

| Owner | Evidence required | Still ask |
| --- | --- | --- |
| Named person | The name and where they asked: issue, PR, commit | Does the request still apply? |
| Real incident | A specific event someone can point to | Does the risk still exist? |
| None | Only "the team", "best practice", or "the agent suggested it" | Propose deletion |

An AI agent is never an owner. Having an owner does not guarantee an item stays; trusted people are questioned least. A Git author is not an owner either: agents write many commits under a human name, so treat the source commit as a lead and let the user confirm. If no owner can be found, the current maintainer may adopt the item during approval.

To trace an item, read it and its source commit, then stop:

```bash
git log --diff-filter=A --format='%h %ad %s' -- <path>
```

If you still cannot say what it prevents, propose deletion and say what is unknown.

**Documents need a purpose.** A document stays if it directly serves the goal or is required to run, build, or reproduce a result.

**Code files need a path to use.** Judge each file as a whole, not its contents. A code file stays if it is reachable from an entry point or the goal requires it.

**Planned items need the removal test:** if this item is removed, can the result the user asked for still be achieved?

- If no, the step is necessary, and the user who asked for the result is its owner.
- If yes, the item is extra. It needs its own owner or it goes. Look hardest at guards, compatibility layers, abstractions, configuration, extra documents, and extra verification.

**These reasons do not keep an item:** consistency, completeness, parity with another tool, "might need it later", and "other documents link to it". For text, a link count measures sync cost, not value.

Judge tracked generated output, such as reports a script rebuilds, as one set: keep only what a current task compares against. A `.md` extension does not prove a human wrote it. Compressing wording is not a substitute for deciding whether the item should exist, and moving content is not deletion. A merge keeps only what has no other home, in an existing file that stays. Never create a file to hold it or to record the deletion.

## Reap

1. **Confirm the goal.** Draft one sentence from the README and recent commits, and ask the user to confirm it. Produce no deletion proposal before that, because an inferred goal lets the agent justify whatever already exists. If the user cannot state a goal, follow [references/goal-interview.md](references/goal-interview.md).
2. **Agree on scope and the protect list.** Suggest rules that guard against loss Git cannot undo. Record the current commit as the base.
3. **Measure, then read only the items you select.**

   ```bash
   wc -c CLAUDE.md AGENTS.md 2>/dev/null  # always-loaded rules
   git ls-files -z | xargs -0 wc -c | sort -rn | head -20  # largest files
   git grep -nF '<path, file name, or directory>'  # references
   ```

4. **Propose one category per batch, about 30 rows at most.** Group rows that share a verdict and evidence. Search every kept file; list each reference fix with its line. Start with what generates bloat: always-loaded rules, skills, process documents, plans, large documents, code files, then the rest.

   | Item | Path | Verdict (delete/merge/keep) | Test and evidence | Owner | Add-back risk | Inbound refs |
   | --- | --- | --- | --- | --- | --- | --- |

   For a category too large to review row by row, propose a keep list instead; everything not on it is proposed for deletion.

5. **Stop and wait for approval.**
6. **Apply only what was approved.** Delete the item, then remove or redirect the references to it. For a merge, state what was removed. Leave no "moved" or "removed" notes; Git keeps the history. If a build or test fails, restore that item and report it.
7. **Leave a map.** Propose the goal and one line per top-level location for the existing README.
8. **Propose the maintenance block** for `AGENTS.md` or `CLAUDE.md`, adapted to the project. If neither exists, ask before creating one.

   ```markdown
   ## Staying lean
   - Core goal: <goal>
   - One in, one out: a change that adds a rule, document, skill, or tool names what it removes or merges. If nothing can go, explain why and get the maintainer's agreement.
   - After drafting a plan and before implementing it, remove anything the requested result does not need, unless a named person or a real incident requires it.
   - When a release ships, delete its plan documents and move unfinished items to the plan index.
   ```

## Plan review

1. **Quote the goal.** Quote the user's own request at the top of the result. Do not take the goal from the plan, because the agent wrote the plan. If you cannot find the request, ask for it.
2. **Apply the removal test** to every planned item. Do not add new items.
3. **Report the result:**

   | Removed item | Why it is not needed | Add back if |
   | --- | --- | --- |

   Then give the trimmed plan. Say so if nothing needed removal.

4. **Handle the plan's location.** If the plan is only in the conversation, the result is the new plan. If it is in a file, get approval before editing the file. A planned deletion of an existing file still follows Reap.

## Check-up

Keep no tracking file and no size budget.

1. Ask for the base commit, such as the last Reap commit.
2. List what was added:

   ```bash
   git log --name-status --format='%h %s' <base>..HEAD -- <scope>
   git log --numstat --format='%h %s' <base>..HEAD -- <scope>
   ```

3. Question only new items, with the Reap table and approval steps.
4. Report where the growth came from, and suggest one change that would stop it at the source.

## Automation

List candidates and the work they save; the user decides. If a guard must always hold, suggest moving it to a test or a hook. Do not add scripts, hooks, or checks during a run.

## Verify and report

- Search for each deleted path and identifier. Expect no dangling references. Report any in files you may not change.
- Confirm with `git status --short` and `git diff --stat` that only approved items changed.
- Confirm that the protect list is intact. Run the build and tests if a code file changed.
- Report before-and-after numbers, unapproved items, and how to recover: `git log --all -- <path>` and `git show <commit>:<path>`.

Unrecoverable loss must be zero. Recoverable add-backs around 10% suggest a deep enough cut. It is a rule of thumb, not a quota.
