# fix: keep results list visible when reopening at threshold edge case

Closes #410

## Problem
When the input value reaches the configured threshold and the search produces a single result (for example input `abcd` and one list item `abcd`), the results list can remain hidden in an internal state-sync edge case.

The issue is caused by `open(ctx)` returning early when `ctx.isOpen` is already `true`, which skips re-applying the DOM visibility state (`hidden` attribute removal).

## Root Cause
In `src/controllers/listController.js`, the previous `open(ctx)` implementation did:
- `if (ctx.isOpen) return;`
- then (only when not already open) set `aria-expanded`, remove `hidden`, set `ctx.isOpen = true`, emit `open`.

If the internal open state and DOM visibility ever drifted (for example hidden attribute still present while `ctx.isOpen === true`), subsequent calls to `open(ctx)` would return immediately and never restore visibility.

## Solution
Make `open(ctx)` idempotent for visibility state, while preserving event semantics:

1. Capture prior state:
- `const wasOpen = ctx.isOpen;`

2. Always apply visibility/open state:
- set `aria-expanded=true`
- remove `hidden`
- set `ctx.isOpen = true`

3. Emit `open` event only on closed→open transition:
- `if (wasOpen) return;`
- otherwise emit `eventEmitter("open", ctx)`

This guarantees the list is shown whenever `open(ctx)` is invoked with results, including threshold + single-result scenarios.

## Files Changed
- `src/controllers/listController.js`
- Rebuilt artifacts:
  - `dist/autoComplete.js`
  - `dist/autoComplete.min.js`
  - `dist/autoComplete.js.gz`
  - `dist/autoComplete.min.js.gz`
  - `docs/demo/js/autoComplete.js`
  - `docs/demo/js/autoComplete.min.js`
  - `docs/demo/js/autoComplete.js.gz`
  - `docs/demo/js/autoComplete.min.js.gz`

## Type of Change
- [x] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] This change requires a documentation update

## How Has This Been Tested?
- [x] Build verification
- [x] Targeted behavioral reasoning against #410 reproduction

Commands run:
1. `npm --prefix /home/calelin/dev/autoComplete.js run build`
2. `git -C /home/calelin/dev/autoComplete.js status --short --branch`

Results:
- Build completed successfully.
- Working tree remained clean after build (no unexpected drift).

## Risk / Impact
Low.
- Behavior only changes in `open(ctx)` guard logic.
- `open` event emission behavior remains unchanged for already-open state.
- Fix improves resilience against internal-state / DOM-visibility drift.

## Rollback Plan
Revert the commit that modifies `src/controllers/listController.js` and regenerated artifacts.
