# issue-driven-gitflow

A reusable [Claude](https://claude.com/claude-code) skill that enforces a
disciplined, **issue-driven GitHub Flow**: nothing gets coded until there's a
GitHub issue and a *reviewed, approved plan* for it. Every change lands on a
short-lived `type/description` branch via a squash-merged PR with Conventional
Commits — and `main` is never edited directly.

It's meant to be dropped into any project so the whole team gets the same
workflow automatically whenever they ask Claude to build, fix, or ship code.

## What it does

When you ask Claude to do development work ("add a dark-mode toggle", "fix the
NaN on an empty cart", "let's start a new service"), the skill drives this loop:

1. **Ensure an issue exists.** The GitHub issue is the source of truth for the
   task; its body holds the spec. No issue → one is created first.
2. **Plan → Review → Approve.** A *planning agent* comments a concrete plan on
   the issue; a separate *review agent* critiques it with fresh eyes; they
   iterate in the issue comments until the reviewer posts `Verdict: APPROVED`.
3. **Implement.** A separate *implementation agent* follows the approved plan,
   on a properly-named branch, with an automated test, landing as a PR.
4. **Land & clean up.** Squash-merge, delete the branch, sync `main`.

When a repo accumulates **more than 3 open issues**, the work is grouped under a
**GitHub Project** so status stays visible.

## The non-negotiables

- **`main` is never edited directly.** On `main` and asked to change code →
  Claude auto-creates the right branch first.
- **Every change starts from a GitHub issue.**
- **No code before an approved plan** — the plan/review loop runs even for small
  changes (a sound one-line plan is approved in a single round).
- **Automated verification is preferred.** If a repo has no test harness, Claude
  stops and asks whether to set one up rather than silently falling back to
  manual checks.

For purely informational/read-only requests ("what's the git command for…",
"explain this function", "show me the diff"), the skill deliberately stays out
of the way and just answers.

## Install

This is a folder-based skill. Install it wherever your environment loads skills
from — e.g. for Claude Code, copy the folder into `~/.claude/skills/`:

```bash
# from a packaged .skill (a zip):
unzip issue-driven-gitflow.skill -d ~/.claude/skills/

# or copy the folder directly:
cp -r issue-driven-gitflow ~/.claude/skills/
```

Then start (or reload) your Claude session. The skill triggers automatically on
development requests based on its description — you don't invoke it by name.

## Requirements

- **`git`** and the **GitHub CLI (`gh`)**, authenticated: `gh auth login`.
- For the multi-agent planning loop, an environment that can dispatch
  subagents (e.g. Claude Code). Without subagents, Claude plays the roles in
  sequence and still leaves the durable issue/PR trail.

## What's in here

| Path | Purpose |
|------|---------|
| `SKILL.md` | The workflow Claude follows (the skill itself). |
| `references/agent-prompts.md` | Role prompts for the planning / review / implementation agents. |
| `references/projects.md` | How and when to group issues under a GitHub Project. |
| `scripts/gitflow.sh` | Bundled helper for the mechanical git/gh steps. |

## The helper script

`scripts/gitflow.sh` encapsulates the three fiddly, repeated steps so naming,
the commit trailer, and the PR footer can't drift. It validates its inputs
(rejects a malformed branch name or a non-Conventional commit):

```bash
# 1. Branch — syncs the default branch, then cuts type/description off it
scripts/gitflow.sh branch feat/add-csv-export

#    ...make changes, write & run the automated test, then: git add -A

# 2. Commit — Conventional message + "Closes #42." + co-author trailer
scripts/gitflow.sh commit "feat(export): add CSV export for reports" 42

# 3. PR — pushes and opens a squash-ready PR (title becomes the squash commit)
scripts/gitflow.sh pr "feat(export): add CSV export for reports" 42
```

`type` ∈ `feat | fix | chore | docs | refactor | test | perf`. The co-author
trailer defaults to `Claude <noreply@anthropic.com>`; set `CLAUDE_COAUTHOR` to
include the exact model, e.g.:

```bash
export CLAUDE_COAUTHOR="Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

## Customizing

The conventions are intentionally explicit in `SKILL.md` and `gitflow.sh` so you
can adapt them to a team's house style — branch `type`s, the commit trailer, the
PR body template, and the >3-issues Project threshold are all easy to change in
those two files.
