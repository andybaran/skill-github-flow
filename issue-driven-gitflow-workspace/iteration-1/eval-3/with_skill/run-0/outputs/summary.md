# Summary

## Request
"what's the git command to list my local branches again?"

## What I did
Answered the question directly. This is a purely informational "how do I" question,
not a request to modify the repository. Per the issue-driven-gitflow skill, the
workflow (issue -> plan -> review -> branch -> PR) triggers on requests that change
a repository's code or operationally manage branches (e.g. "start work", "fix a bug",
"open a PR", "create/delete a branch"). Asking what command lists branches does none
of that, so the correct application of the skill is to apply none of its machinery
and simply answer.

## GitHub artifacts created
NONE. No issue, branch, PR, or project was created.
- Issues after: [] (empty)
- PRs after: [] (empty)

## Note on setup
The `gh repo clone andybaran/gfeval-negative-ws /tmp/gfeval-run/negative-with` step
failed with "Repository not found" (repo not accessible to this account). This did
not affect the answer, since the question is general git knowledge independent of the
repo. The `gh issue list` / `gh pr list` calls still succeeded and returned empty,
confirming no artifacts were created.

## Answer given
`git branch` (variants: `-v`, `-a`, `-r`, `--show-current`). Full text in answer.md.
