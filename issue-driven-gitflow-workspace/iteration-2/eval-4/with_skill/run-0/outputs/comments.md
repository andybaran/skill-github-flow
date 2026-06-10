==================== ISSUE #1 ====================
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🗺️ Plan (planning agent)

**Approach**
Implement a small, self-contained auth module in CommonJS, matching the existing scaffold (`src/items.js` exports via `module.exports`; tests run with `node --test`). Keep the first cut in-memory (a `users` Map) with no external DB or web framework — the issue explicitly allows a simple in-memory first cut. Passwords are hashed with Node's built-in `crypto.scryptSync` + a per-user random salt (no third-party deps, keeping the scaffold dependency-free). Tokens are opaque random strings (`crypto.randomBytes`) mapped to a userId in an in-memory `sessions` Map — sufficient to let the notes CRUD and sync issues verify a credential and resolve the current user. This deliberately avoids JWT/bcrypt to stay dependency-free for the greenfield first cut; we can revisit when a real datastore/transport is chosen.

**Files to change**
- `src/auth.js` (new) — the auth module exporting:
  - `register({ username, password })` -> `{ id, username }`; throws on duplicate username or missing/empty fields.
  - `login({ username, password })` -> `{ token, user: { id, username } }`; throws on unknown user or bad password.
  - `verifyToken(token)` -> `{ id, username }`; throws on missing/invalid token. This is the helper the notes CRUD / sync modules will consume.
  - `_reset()` — test helper to clear in-memory state between tests.
  - Internals: `hashPassword(password, salt)` using `scryptSync`, constant-time compare via `crypto.timingSafeEqual`, salts + tokens via `crypto.randomBytes`.
- `test/auth.test.js` (new) — `node --test` cases (see Test strategy).
- No changes to `src/items.js`, `package.json` (no new deps), or the existing test.

**Risks / unknowns**
- In-memory state means no persistence across process restarts — acceptable and explicit for the first cut; a future issue swaps the store for a DB.
- No HTTP layer yet (no Express in scaffold). The module exposes plain functions; wiring to routes is deferred to when a transport is chosen. The issue acceptance criteria are all expressible at the function level, so this is fine.
- `scryptSync` is synchronous/CPU-bound; fine for tests and a first cut, revisit for scale.
- Token store is global in-memory; `_reset()` keeps tests isolated.

**Test strategy**
Automated tests in `test/auth.test.js`, run via the existing `npm test` (`node --test`) harness — no new harness needed. Cases:
1. register: a new user registers and returns an id+username; a duplicate username throws.
2. login success: correct credentials return a token and the user.
3. login failure: wrong password throws; unknown user throws.
4. verifyToken: a token from login resolves to the correct user; a bogus/missing token throws.
5. password storage: the stored record does not contain the plaintext password (hash present, plaintext absent).
Each test calls `_reset()` first for isolation. Verified green locally before opening the PR.
--
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🔍 Review (review agent)

Reviewed the plan independently against the repo (`src/items.js` is CommonJS with `module.exports`; `test/items.test.js` uses `node:test` + `node:assert`; `package.json` test script is `node --test`, zero deps). Assessment:

**What is sound**
- Staying CommonJS + zero-dependency (built-in `crypto`) matches the scaffold exactly — no `package.json` churn, no install step, tests keep running under `node --test`. Correct call for a greenfield first cut.
- `scryptSync` + per-user random salt + `timingSafeEqual` is a legitimate password-hashing choice and avoids storing plaintext. Good.
- Exposing `register`/`login`/`verifyToken` as plain functions, with `verifyToken` as the explicit seam for the notes-CRUD and sync issues, is the right boundary — it satisfies every acceptance criterion at the function level without prematurely picking an HTTP framework.
- `_reset()` for test isolation is necessary given module-global in-memory state; good that it is called out.

**Points to nail down (non-blocking, but do them)**
- `timingSafeEqual` throws if the two buffers differ in length. Since both sides are fixed-length scrypt digests of the same keylen this is safe, but ensure the derived-key length is fixed (e.g. 64) so an attacker-supplied password cannot produce a different-length buffer and turn a failed login into a thrown TypeError instead of a clean "bad credentials". Compare hashes of equal, fixed keylen only.
- Make sure `verifyToken` and `login` failures throw a *uniform* error (do not leak "unknown user" vs "bad password" as distinguishable messages if these ever surface to clients). Minor for an in-memory first cut, but cheap to get right now.
- Confirm the duplicate-registration check is on the normalized username (decide case sensitivity); fine to keep it simple (exact match) as long as a test pins the behavior.

**Coverage check** — the 5 planned tests map 1:1 to the acceptance criteria (register + duplicate, login success, login failure, token verify success/failure, plaintext-absent). That is complete. No migrations, no external callers to update (greenfield).

The approach is the simplest thing that works and is correctly scoped. The points above are refinements the implementer can fold in without changing the design.

**Verdict: APPROVED**
--

==================== ISSUE #2 ====================

==================== ISSUE #3 ====================

