author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🗺️ Plan (planning agent)

**Approach**
Drive theming entirely through the CSS custom properties already defined in `src/styles.css` (`--bg`, `--fg`, `--accent`). Add a `[data-theme="dark"]` selector on `<html>` that overrides those variables, a checkbox toggle on the settings page, and a tiny script that persists the choice to `localStorage` and applies it on load. To avoid a flash of light theme on navigation, the apply-on-load logic runs from a small inline head script on every page; the toggle wiring lives in `src/settings.js`.

**Files to change**
- `src/styles.css` — add a `html[data-theme="dark"] { --bg: #121212; --fg: #ededed; --accent: #5b9dff; }` block plus a `border-color` token so the header divider works in dark mode. Add minimal styles for the toggle row.
- `settings.html` — add a `.setting-row` with a labeled checkbox `#theme-toggle` ("Dark mode") under Preferences; add the shared inline head script that reads `localStorage.theme` and sets `document.documentElement.dataset.theme` before paint.
- `index.html` — add the same inline head script so the dark theme applies on the home page too (acceptance criterion: applies to all pages).
- `src/settings.js` — initialize the checkbox from the persisted value and add a `change` listener that sets/clears `data-theme` and writes `localStorage.theme`.

**Risks / unknowns**
- Flash of unstyled/light theme (FOUC) on load if the apply logic runs too late — mitigated by the inline head script that runs before body render.
- Duplicating the inline script across two HTML files is mild repetition; acceptable for a 2-page static app with no build step (a shared module would need to be a render-blocking `<script src>` in head, same cost).
- localStorage may be unavailable (private mode); wrap access in a try/catch so the toggle still works in-session.

**Test strategy**
Manual verification (no test harness in repo): open `settings.html`, toggle Dark mode → background/text invert immediately; reload → stays dark and checkbox stays checked; navigate to `index.html` → still dark; toggle off → returns to light and persists. Confirm no console errors. Validate HTML/JS by loading in a browser.
--
author:	andybaran
association:	owner
edited:	false
status:	none
--
## 🔍 Review (review agent)

I checked the plan against the actual files on `main` (`src/styles.css`, `settings.html`, `index.html`). The CSS-variable approach is the right one — `body` already consumes `--bg`/`--fg` via `var()`, so overriding the variables on `html[data-theme="dark"]` cleanly recolors everything that uses them with zero churn.

Things the plan gets right:
- Driving theme via the existing `:root` custom properties is the simplest mechanism that works.
- Inline head script to prevent FOUC is the correct call — a deferred `src/settings.js` would flash light first.
- try/catch around `localStorage` is appropriate (private-mode robustness).

Points to confirm/tighten (none blocking, but fold into implementation):
1. **Hardcoded divider color.** `header` uses `border-bottom: 1px solid #e5e5e5` — a literal, *not* a variable. On a dark background that light-grey line will look wrong. The plan mentions adding a `border-color` token; make sure you actually replace the literal in the `header` rule with `var(--border)` and define `--border` in both `:root` and the dark block. This is the one real visual bug if missed.
2. **Toggle initialization source of truth.** Initialize the checkbox `checked` state from `document.documentElement.dataset.theme === "dark"` (already set by the head script) rather than re-reading localStorage in `settings.js`, so there is a single source of truth and no risk of the two diverging.
3. **Accent contrast.** `--accent` going to `#5b9dff` on `#121212` is fine for nav links; no action needed, just noting it passes a contrast sanity check.

The duplicated inline script across two files is acceptable for a 2-page no-build app, as the plan argues. Test strategy (manual: toggle → reload → cross-page nav) covers the acceptance criteria.

The plan is sound and ready to implement with point 1 treated as required.

**Verdict: APPROVED**
--
