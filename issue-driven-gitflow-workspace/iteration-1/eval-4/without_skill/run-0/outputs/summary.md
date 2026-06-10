# Greenfield kickoff — Notabene note-taking app

Ordered narrative of what I did to kick off the project in
`andybaran/gfeval-greenfield-base`.

## 0. Environment fix (clone/auth)

The clone failed with "Repository not found" even though I had admin access via
the API. Root cause: a global git `insteadOf` rule rewrites all
`https://github.com/` URLs to SSH, and SSH on this machine authenticates as a
different account (`akb-altar-bike`) that can't see this private repo. The macOS
CommandLineTools gitconfig also forced an `osxkeychain` credential helper.

Fix: I ran all git commands through a small wrapper that sets
`GIT_CONFIG_SYSTEM=/dev/null` and points `GIT_CONFIG_GLOBAL` at a clean config
that (a) drops the SSH `insteadOf` rewrite, (b) resets the credential helper to
`gh auth git-credential`, and (c) sets the `andybaran` identity. The repo was
genuinely empty (no commits), so there was nothing to clone — I initialized it
locally and pushed. No token was embedded in any URL or `.git/config`.

## 1. Scaffolded the monorepo (on `main`)

Created an npm-workspaces TypeScript monorepo and pushed it as the baseline:

- `README.md` — services table, architecture sketch, tech stack, dev workflow
- `package.json` — workspaces (`packages/*`, `services/*`) + build/test/lint scripts
- `tsconfig.base.json`, `.gitignore`
- `packages/shared` (`@notabene/shared`) — shared domain types: `User`, `Note`,
  `AuthTokenClaims`, `SyncEvent`, `SYNC_PROTOCOL_VERSION`
- `.github/workflows/ci.yml` — install / build / test on push + PR

Commit `e534c7b`, pushed to `origin/main`.

## 2. Created one tracking issue per service

- #1 Auth service: registration, login, JWT issuance
- #2 Notes API: CRUD endpoints
- #3 Sync service: real-time sync across devices

Each issue has a scope checklist and notes on cross-service dependencies.

## 3. Built each service on its own branch + PR (none merged)

Each service was branched from `main`, implemented as a reasonable first cut
with tests, pushed, and opened as a PR that closes its issue.

- **#4** `feat/auth-service` -> closes #1
  - `POST /register` (bcrypt, 8-char min), `POST /login` (constant-shape
    compare), `POST /verify`, `GET /health`; in-memory `UserStore`;
    JWT issue/verify on the shared claims type; `node:test` token round-trip.
- **#5** `feat/notes-api` -> closes #2
  - Full CRUD (`POST/GET/GET:id/PATCH/DELETE /notes`); JWT auth middleware;
    ownership checks (404 on others' notes); in-memory `NoteStore` that bumps
    `version`+`updatedAt`; emits a `SyncEvent` per mutation; store tests.
- **#6** `feat/sync-service` -> closes #3
  - JWT-authenticated WebSocket `/sync`; per-user `ConnectionRegistry` +
    presence; broadcast to a user's other devices; `POST /events` ingress for
    notes-api; last-write-wins reconciliation on `version`; reconcile tests.

## 4. Linked issues to PRs

Added a comment on each of #1/#2/#3 pointing at its PR (#4/#5/#6).

## Constraints honored

- No PR merged (all three are OPEN).
- Repo not deleted.

## Note on Projects

`gh project list --owner andybaran` shows a pre-existing project "Notes App"
(#10). I did not create or modify any GitHub Project; it predates this run.

## Final GitHub state

- Branches: `main` + `feat/auth-service`, `feat/notes-api`, `feat/sync-service`
- Issues: #1, #2, #3 (all open)
- PRs: #4, #5, #6 (all open, not merged)
