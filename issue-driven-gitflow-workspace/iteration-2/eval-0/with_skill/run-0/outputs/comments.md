=== Issue #1 ===
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🗺️ Plan (planning agent)

**Approach**
Add a dark-mode toggle alongside the existing checkboxes in the `#settings` section. Drive appearance with a single `data-theme="dark"` attribute on `<html>` (or a `dark` class on `<body>`), styled in `styles.css`. Reuse the existing `localStorage` persistence pattern in `settings.js` so the dark-mode choice is saved and re-applied on load — but apply the saved theme *before* paint to avoid a flash, and keep the generic `#settings input` change-handler working for the other toggles.

**Files to change**
- `settings.html` — add `<label>Dark mode <input type="checkbox" id="darkmode"></label>` inside `#settings`. No other markup change.
- `src/styles.css` — add dark-theme rules scoped under `[data-theme="dark"]` (background, text color, and ensure the existing `body`/`label` rules still read well in dark). Keep light as the default.
- `src/settings.js` — (1) on load, read `localStorage.getItem('darkmode')`, set the checkbox's `checked` and apply/remove `data-theme` on `<html>`; (2) add a `change` handler for `#darkmode` that toggles `data-theme` live. The existing `forEach` over `#settings input` already persists `darkmode` to `localStorage` (it stores `e.target.checked` keyed by `id`), so persistence comes for free — but the theme *application* (toggling the attribute) is new logic that must be added without breaking the generic handler.

**Risks / unknowns**
- The existing generic handler stores `checked` as the string `"true"`/"false"`; on load I must compare against the string `'true'`, not a boolean — easy to get wrong.
- Flash-of-light-before-dark on reload if the attribute is applied late; apply it as early as possible in the script (script is at end of body, acceptable for this tiny page).
- Must not double-handle `#darkmode` (generic handler + dedicated handler both firing) in a way that conflicts — they don't (one persists, one applies theme), but worth verifying.

**Test strategy**
⚠️ **This repo has NO test harness** — no `package.json`, no CI workflow, no test runner or test files anywhere. There is nothing to hang an automated test on. Per our automated-verification standard, I am **not** quietly defaulting to manual verification: **the user must decide** whether to stand up a minimal automated test harness (e.g. a small Node + jsdom or Vitest setup, or a Playwright DOM test that loads `settings.html`, toggles `#darkmode`, and asserts `localStorage` + the `data-theme` attribute) before this change lands, accepting the added dependency/setup cost — or to knowingly accept this one change on manual verification only.

Recommended automated test *if* a harness is approved: a jsdom/Vitest unit test that loads the DOM, dispatches a `change` on `#darkmode`, and asserts (a) `document.documentElement` gains `data-theme="dark"`, (b) `localStorage.darkmode === 'true'`, and (c) reloading re-applies the attribute.

**Decision needed from the user before merge:** add a test harness, or accept manual verification for this change?
--
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🔍 Review (review agent)

Independently checked the plan against `settings.html`, `src/settings.js`, and `src/styles.css`.

**What's sound**
- File set is correct and minimal: the three files named are exactly the ones involved, and no build step / framework is introduced (matches the acceptance criteria).
- The claim that persistence is free is **verified against the code**: `settings.js` does `document.querySelectorAll('#settings input').forEach(... localStorage.setItem(e.target.id, e.target.checked))`. `#darkmode` is an `input` inside `#settings`, so its `change` is already persisted under key `darkmode`. Good — no duplicate persistence logic needed.
- The string-vs-boolean risk is correctly flagged: `setItem` coerces to `"true"`/`"false"`, so the load path must test `=== 'true'`. This is the most likely bug and the plan calls it out.
- Driving theme off a single `data-theme="dark"` attribute on `<html>` with CSS scoped under `[data-theme="dark"]` is the simplest approach that works and keeps light as the default.

**Points to tighten (non-blocking)**
- The dedicated load logic must NOT rely on the generic `forEach` having applied the theme — it won't; the generic handler only persists on `change`, it never reads back or applies anything on load. So the new load code is solely responsible for both setting `checkbox.checked` AND applying `data-theme` on page load. The plan implies this; just make it explicit so the implementer doesn't assume the loop helps on load.
- Flash-of-light: acceptable to accept for a page this trivial; no need to move the script into `<head>`. Don't over-engineer.

**On testing** — I concur fully with the planner: there is genuinely **no test harness** in this repo (confirmed: no `package.json`, no `.github`, no test files). The plan correctly refuses to rubber-stamp 'manual verification' and escalates the harness decision to the user. That is the right call per our automated-verification standard, and it is the one thing that must be resolved with the user before this PR is merged. The plan does not block on it for *writing* the code, which is fine — but the PR must not land until the user has chosen.

The approach is sound and ready to implement; the harness question is a merge-gate, not a plan defect.

**Verdict: APPROVED**
--
