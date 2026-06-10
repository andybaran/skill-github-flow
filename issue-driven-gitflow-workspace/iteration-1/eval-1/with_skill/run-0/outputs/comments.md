author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🗺️ Plan (planning agent)

**Approach.** The root cause is a single `Array.prototype.reduce` call in `src/cart.js` that omits the initial accumulator value. Adding `0` as the initial value fixes both failure modes at once: the empty array no longer throws, and populated carts seed the accumulator with a number instead of the first item object. This is the minimal, idiomatic fix — no API or signature change.

**Files to change.**
- `src/cart.js` — in `cartTotal`, change `items.reduce((sum, item) => sum + item.price * item.qty)` to pass `0` as the second argument to `reduce`: `items.reduce((sum, item) => sum + item.price * item.qty, 0)`. No other lines change; the export and JSDoc stay as-is.
- `test.js` — no change needed. It already asserts the populated, single-item, and empty-cart cases (`cartTotal([])` → `0`). It currently fails on all three because of the bug and should go green after the fix.

**Risks / unknowns.**
- Very low risk: `reduce(..., 0)` is the standard sum idiom. Numeric correctness for non-empty carts is preserved (sum starts at 0).
- Edge: if `items` is `null`/`undefined`, `.reduce` would still throw — but that is out of scope; the issue and tests only cover array inputs including the empty array. Not expanding scope.
- No callers beyond `test.js` exist in the repo, so no downstream breakage.

**Test strategy.** Run `node test.js`; all three checks (`sums a populated cart` → 35, `single item cart` → 28, `empty cart totals to 0` → 0) must pass and the process must exit 0. This directly maps to the issue's acceptance criteria.
--
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🔍 Review (review agent)

Independently verified the plan against the code.

- **Right file / right line.** `src/cart.js` line 9 is `return items.reduce((sum, item) => sum + item.price * item.qty);` — confirmed the missing initial value is the sole defect. Adding `, 0` is the correct, minimal fix.
- **Both failure modes covered by one change.** Empty array (`Reduce of empty array with no initial value` throw) and the populated-cart object-seed garbage both stem from the omitted seed; `0` resolves both. Numeric result is unchanged for valid carts since adding from 0 is identity.
- **Tests already pin the acceptance criteria.** `test.js` asserts 35, 28, and the empty-cart → `0` case (line 30). No test change is needed, and the plan correctly declines to add one. Good call not to touch `test.js`.
- **Scope discipline is right.** Declining to handle `null`/`undefined` `items` is correct — out of scope for this issue and not in the tests. Don't gold-plate.

No edge cases missed, no affected callers beyond `test.js`, simplest approach chosen.

**Verdict: APPROVED**
--
