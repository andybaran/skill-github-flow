===== ISSUE #1 =====
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🗺️ Plan (planning agent)

**Stack decision (greenfield):** This repo is empty apart from the README, so there is no existing stack to match. I propose **Node.js + TypeScript + Express** with **Jest** for tests — a conventional, widely-understood choice for a notes app with auth/CRUD/sync services, and it keeps all three planned services in one toolchain. Persistence starts as an in-memory/repository-interface store so #1 is self-contained and testable without a DB dependency; the repository interface lets us swap in a real DB later without touching the auth logic.

### Approach
Build a self-contained `auth` module exposing (a) HTTP routes for `register` and `login`, and (b) a reusable `requireAuth` middleware plus a `verifyToken` util that other services (#2 notes, #3 sync) will import to resolve the current user. Passwords are hashed with `bcrypt`; sessions are stateless **JWTs** signed with a server secret. A `UserRepository` interface backs storage (in-memory impl for now). This keeps the public contract other services depend on small and explicit.

### Files to change
- `package.json`, `tsconfig.json`, `jest.config.js` — project scaffolding (first code in the repo).
- `src/auth/types.ts` — `User`, `UserRepository` interface, request/response DTOs.
- `src/auth/userRepository.ts` — `InMemoryUserRepository` implementing the interface.
- `src/auth/password.ts` — `hashPassword` / `verifyPassword` (bcrypt wrappers).
- `src/auth/token.ts` — `signToken(userId)` / `verifyToken(token)` (jsonwebtoken wrappers, secret from `AUTH_JWT_SECRET` env with a dev fallback).
- `src/auth/service.ts` — `registerUser` (reject duplicate email, hash pw) and `loginUser` (verify pw, issue token); validation + typed errors.
- `src/auth/middleware.ts` — `requireAuth` Express middleware: reads `Authorization: Bearer`, verifies, attaches `req.userId`; 401 on missing/invalid.
- `src/auth/routes.ts` — Express router wiring `POST /auth/register`, `POST /auth/login`, and an example `GET /auth/me` protected by `requireAuth` (demonstrates the boundary other services reuse).
- `src/app.ts` / `src/index.ts` — assemble the Express app and start the server.
- `src/auth/__tests__/auth.test.ts` — unit/integration tests.
- `README.md` — short auth usage/section + env var note.

### Risks / unknowns
- **JWT secret management:** dev fallback is convenient but must NOT be used in prod — plan documents `AUTH_JWT_SECRET` as required outside dev; the fallback only applies when `NODE_ENV !== production`.
- **In-memory store** loses data on restart — acceptable for #1, but the `UserRepository` interface is the seam so #2/#3 are not coupled to it.
- **Public contract churn:** #2 and #3 import `requireAuth`/`verifyToken`; if their shapes change later it ripples. Mitigated by keeping that surface minimal and documented now.
- **Timing/enumeration:** login should not reveal whether email vs. password was wrong — return a generic "invalid credentials".

### Test strategy
Jest with supertest against the Express app:
- register: success; duplicate-email rejected (409); missing/invalid fields rejected (400).
- login: success returns a token; wrong password rejected (401); unknown email rejected (401, generic message).
- token: `verifyToken` round-trips a signed token; tampered/expired token rejected.
- middleware: `GET /auth/me` is 401 without a token, 401 with a bad token, 200 + user id with a valid token.
Passwords asserted to be stored hashed (not equal to plaintext).
--
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🔍 Review (review agent)

Reviewed the plan against the issue acceptance criteria and the (empty) repo. The stack choice is reasonable for a greenfield repo and the planner correctly justified it rather than assuming a pre-existing one. The `UserRepository` seam and the minimal `requireAuth`/`verifyToken` public surface are the right calls since #2 and #3 explicitly depend on them. Coverage of the acceptance criteria is complete. A few points to tighten — none are blockers, but the implementer should honor them:

1. **JWT expiry must be set.** The plan mentions rejecting an expired token in tests but never says tokens *get* an expiry. `signToken` must set `expiresIn` (e.g. 1h) or the "expired token rejected" test cannot pass and tokens would live forever. Make this explicit.
2. **Generic-credentials response — keep it truly generic.** Good that the plan calls out enumeration. Ensure unknown-email and wrong-password return the *same* status and body (401, "invalid credentials"). The plan currently lists both as 401 which is correct; just confirm identical body.
3. **bcrypt cost factor & async.** Use bcrypt async APIs (not sync) and a sane cost (10–12). Worth stating so the implementer does not block the event loop with `hashSync`.
4. **Email normalization.** Duplicate-email rejection should be case-insensitive (normalize/lowercase + trim before store and lookup), otherwise `A@x.com` and `a@x.com` register as two users. Minor but a real correctness gap for the duplicate-email criterion.
5. **Dont leak the hash.** `GET /auth/me` and any user response must not serialize `passwordHash`. Trivial but easy to forget.

None of these change the file list or architecture — they are implementation constraints on code already planned. The approach is the simplest one that satisfies the criteria.

**Verdict: APPROVED**

(Implementer: treat points 1–5 as required constraints; they fall entirely within the files already listed.)
--
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🛠️ Implementation (implementation agent)

Implemented the approved plan on branch `feat/auth-service` → **PR #4**.

All five review constraints honored:
1. JWT expiry set to 1h (`token.ts`).
2. Unknown email and wrong password return identical `401 { error: "invalid credentials" }` (`service.ts`).
3. bcrypt async, cost 12 (`password.ts`).
4. Email normalized (trim + lowercase) so duplicate detection is case-insensitive (`service.ts` + `userRepository.ts`).
5. Password hash never serialized — `toPublicUser` strips it (`service.ts`).

Verification: `npx tsc --noEmit` clean; `npm test` → 12/12 passing (register success/duplicate/validation/hash-at-rest, login success/wrong-pw/unknown-email, token round-trip + tamper, requireAuth no-token/bad-token/valid-token).

PR squash-merge will auto-close this issue via `Closes #1.`
--


===== ISSUE #2 =====


===== ISSUE #3 =====


