# GitHub Projects: grouping issues when work scales

When a repo has more than 3 open issues, scattered issues lose their shared
thread — it's hard to see what's in flight, blocked, or done. A GitHub Project
(the newer "Projects v2", board view) gives the workstream a single status
surface.

## Create a project (once per workstream)

```bash
gh project create --owner "@me" --title "<workstream name>"
# → returns a project number, e.g. 7, and a URL. Tell the user the URL.
```

For an org-owned repo, use `--owner <org>` instead of `@me`.

## Add issues to it

```bash
gh project item-add <project-number> --owner "@me" --url <issue-url>
```

Add every issue in the workstream — both pre-existing ones (which is why you
create the project the moment you cross the >3 threshold) and new ones as they're
filed.

## Move items across status columns

Projects v2 status is a custom single-select field. To set it you need the
field and option IDs:

```bash
# Discover the Status field id and its option ids (Todo / In Progress / Done)
gh project field-list <project-number> --owner "@me" --format json

# Then update an item's status
gh project item-edit \
  --id <item-id> \
  --project-id <project-id> \
  --field-id <status-field-id> \
  --single-select-option-id <option-id>
```

Move an item to **In Progress** when the implementation agent starts its branch,
and to **Done** when the PR merges. This keeps the board honest without manual
bookkeeping by the user.

## Linking the project to the repo

So the project shows up under the repo's Projects tab:

```bash
gh project link <project-number> --owner "@me" --repo <owner>/<repo>
```

## Keep it lightweight

The point of the project is *visibility*, not ceremony. Don't invent elaborate
custom fields or automation the user didn't ask for — Todo / In Progress / Done
plus the linked issues is plenty unless they want more.
