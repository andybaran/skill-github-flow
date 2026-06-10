# Summary — Greenfield notes-app (with skill)

Repo: `andybaran/gfeval-greenfield-ws` (started empty — no commits, no default branch).
Applied the issue-driven-gitflow skill end to end for a brand-new note-taking app.

## Ordered narrative

1. **Orient (Step 0).** Cloned the repo; found it completely empty (`isEmpty: true`,
   no `main`, 0 open issues). The request was three distinct pieces of work
   (auth, notes CRUD, sync) — a multi-service greenfield app.

2. **Filed issues (Step 1).** Created three issues, each with a full self-contained
   body (context, scope, out-of-scope, acceptance criteria, dependencies):
   - #1 — User authentication service (the foundation everything else depends on).
   - #2 — Notes CRUD API (depends on #1).
   - #3 — Notes sync service (depends on #1 and #2).

3. **Created a GitHub Project.** One coherent workstream for a new app with more
   issues certain to follow (the issues list follow-ups: OAuth, password reset,
   sharing, search, real-time). Created **Project #10 "Notes App"**
   (https://github.com/users/andybaran/projects/10), added all three issues, and
   linked it to the repo.

4. **Plan -> Review -> Approve on #1 (Step 2), in the issue comments.**
   - Planning agent: concrete plan — Node + TypeScript + Express + Jest stack
     (justified since repo was empty), UserRepository storage seam, bcrypt +
     stateless JWT, minimal reusable requireAuth/verifyToken surface for #2/#3,
     file-by-file change list, risks, test strategy.
   - Review agent (fresh eyes): 5 actionable constraints (set JWT expiry; identical
     generic login error to prevent enumeration; async bcrypt + sane cost;
     case-insensitive email/duplicate handling; never serialize the password hash)
     and **Verdict: APPROVED** with those as required constraints.

5. **Implement (Step 3).** Moved #1 to In Progress on the board. Branched off `main`,
   implemented the auth module per the approved plan honoring all 5 constraints.
   `npx tsc --noEmit` clean; `npm test` 12/12 passing. Conventional Commit with
   `Closes #1.` and the Claude co-author trailer.

6. **PR opened (not merged, per constraints).** PR #4 —
   `feat(auth): add user authentication service` (base `main` <- head `feat/auth-service`),
   body with summary + test plan + `Closes #1.`
   https://github.com/andybaran/gfeval-greenfield-ws/pull/4

## Branch / PR
- Work branch: `feat/auth-service` (1 commit ahead of `main`; clean diff = auth work only).
- PR: #4, OPEN, base `main`.
- Issue worked: #1 (User authentication service), OPEN — auto-closes on squash-merge.

## Bootstrapping notes (environment friction)
- Repo had no `main`. Created a bootstrap `main` (README + .gitignore) to give the
  gitflow a base to branch from and a PR target, then set it as the repo default.
- An earlier branch (`feat/user-auth-service`) was pushed before `main` existed, so
  GitHub made it the default; after establishing `main` I rebased onto `main` and
  re-pushed as `feat/auth-service` for a clean PR.
- Force-push and remote-branch-deletion were blocked by the harness safety classifier,
  so the stale `feat/user-auth-service` remote branch remains (harmless; no PR from it).
  The active clean PR is #4 from `feat/auth-service`.

## Artifacts created
- Issues: #1, #2, #3
- Project: #10 "Notes App" (linked to repo), all 3 issues added, #1 set to In Progress
- Branches: `main` (bootstrap), `feat/auth-service` (auth work) [+ stale `feat/user-auth-service`]
- PR: #4 (open, not merged)
- Issue #1 comment trail: plan -> review (APPROVED) -> implementation note
