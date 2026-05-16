# [v10.3.0] Prototype extension per-instance overhead
Closes #459
## Summary
This change removes per-instance prototype extension work by moving extension execution to module initialization time, and keeps extension idempotent.
## Problem
`extend.call(this, autoComplete)` was executed in the constructor on every `new autoComplete()` call. Prototype methods are shared and identical across instances, so re-attaching them per instance adds unnecessary overhead.
## Root Cause
Prototype extension logic lived in constructor flow instead of module setup flow, causing repeated method assignment work.
## Solution
1. In `src/autoComplete.js`, removed constructor-level extension call:
   - Removed `extend.call(this, autoComplete)` from constructor.
   - Added module-level one-time call: `extend(autoComplete)`.
2. In `src/services/extend.js`, made extension idempotent:
   - Added internal marker `__autoCompleteExtended` on prototype.
   - Early-return if prototype is already extended.
   - Updated function doc type from instance object to constructor function.
This ensures extension runs at import time and remains safe even if invoked again.
## Files changed
- `src/autoComplete.js`
- `src/services/extend.js`
- `dist/autoComplete.js`
- `dist/autoComplete.min.js`
- `dist/autoComplete.js.gz`
- `dist/autoComplete.min.js.gz`
- `docs/demo/js/autoComplete.js`
- `docs/demo/js/autoComplete.min.js`
- `docs/demo/js/autoComplete.js.gz`
- `docs/demo/js/autoComplete.min.js.gz`
## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [x] Enhancement / performance improvement (non-breaking)
- [ ] Breaking change
- [ ] Documentation-only change
## How Has This Been Tested?
- [x] Ran `npm run build` successfully.
- [x] Verified generated dist and demo bundles update correctly.
- [x] Verified branch diff only contains expected source and generated artifact changes for this optimization.
## Risk / Impact
Low risk. Public API behavior is unchanged; only extension timing is adjusted to avoid redundant work. Idempotent guard further reduces risk of duplicate prototype mutation.
## Rollback Plan
If any regression appears:
1. Move extension call back into constructor in `src/autoComplete.js`.
2. Remove extension marker guard in `src/services/extend.js`.
3. Rebuild dist/demo artifacts and revert this issue commit.
## Checklist
- [x] Code follows project style.
- [x] Self-review completed.
- [x] Build completes without new warnings.
