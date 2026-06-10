# Issues touched

## Issue #1

author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🗺️ Plan (planning agent)

**Approach** — Extend `listItems()` in `src/items.js` to accept an optional `{ limit, offset }` options object and return a paginated slice of `ITEMS`. Defaults: `limit` = 20, `offset` = 0; clamp `limit` to the range [1, 100] (max page size 100) and floor `offset` at 0. Keep the call signature backward-compatible: `listItems()` with no args returns the default first page (20 items). Document the contract in a doc comment and the README.

**Files to change**
- `src/items.js` — change `listItems()` to `listItems({ limit = 20, offset = 0 } = {})`; coerce/validate inputs (non-numeric or < 1 limit → default 20; limit > 100 → 100; offset < 0 → 0), then `return ITEMS.slice(offset, offset + limit)`. Add a doc comment describing limit/offset semantics and defaults.
- `test/items.test.js` — add unit tests proving limit/offset slice correctly (see Test strategy).
- `README.md` — one line documenting the pagination params and defaults.

**Risks / unknowns**
- The existing test `listItems returns all seeded items` asserts `listItems().length === 42`. Changing the default to return 20 will **break it**. Resolution: update that test to reflect the new default-page contract (default page = 20), since returning all 42 by default is exactly the behavior the issue asks to replace. This is intentional, not a regression.
- Input validation surface: deciding clamp-vs-throw for out-of-range values. Plan clamps to sane bounds (matches the issue's "sane defaults" wording) rather than throwing, keeping the function total.

**Test strategy** — Automated, via the existing `node --test` harness (`npm test`). Add cases to `test/items.test.js`:
1. `listItems()` (no args) returns 20 items, first id 1.
2. `listItems({ limit: 5, offset: 0 })` returns ids 1–5.
3. `listItems({ limit: 10, offset: 10 })` returns ids 11–20 (offset slices correctly).
4. `listItems({ limit: 1000 })` is clamped to 100 items.
5. `listItems({ offset: 40, limit: 20 })` returns the final 2 items (tail slice shorter than limit).
Update the existing default-count test to assert 20. All run green under `node --test` before the PR opens.
--
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🔍 Review (review agent)

I checked the plan against `src/items.js` (the `ITEMS` array of 42 `{id, name}` objects, ids 1–42) and `test/items.test.js` (single test asserting length 42 via `node --test`). Fresh-eyes assessment:

- **Right files.** `src/items.js` and `test/items.test.js` are the only logic surfaces; README is the right doc home. Nothing else imports `listItems` (only `ITEMS` is also exported, and the slice change doesn't touch `ITEMS`), so there are no other callers to break.
- **Existing-test change is correct, not a smell.** The issue explicitly wants to stop returning all 42 by default, so updating the `=== 42` assertion to `=== 20` is the intended contract change. Good that the plan calls this out rather than hiding it.
- **Clamp-not-throw is the right call** for "sane defaults" — keeps the function total and avoids an error path the issue didn't ask for. Make sure `NaN`/negative `limit` falls back to the default 20 (not to the min of 1); the plan says this — keep it.
- **Edge cases covered.** Tail slice (offset 40 → 2 items) and over-max clamp (1000 → 100) are both tested. Offset beyond the array (e.g. 100) would yield `[]` via `slice`, which is fine; optional to add but not required.
- **Test strategy is genuinely automated** on the existing harness — no new tooling, runs under `npm test`. This satisfies the automated-verification bar.

The approach is the simplest one that works and the acceptance criterion (a unit test proving limit/offset slice correctly) is directly covered by cases 2 and 3.

**Verdict: APPROVED**
--
