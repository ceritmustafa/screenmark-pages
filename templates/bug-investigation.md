# Bug Investigation: [Short Description]

**Date**: [YYYY-MM-DD]
**Severity**: P0 (Critical) | P1 (High) | P2 (Medium) | P3 (Low)
**Status**: Investigating | Root Cause Found | Fixed | Won't Fix
**Affected Module**: [ScreenMarkCore / ScreenMarkServices / ScreenMarkUI]

---

## Symptom

What happens? Be specific about the observable behavior.

## Expected Behavior

What should happen instead?

## Reproduction Steps

1. Step 1
2. Step 2
3. Step 3

**Frequency**: Always | Intermittent (roughly X% of attempts) | Rare

## Environment

- macOS version:
- ScreenMark version/commit:
- Display configuration (single/multi):
- Relevant permissions:

## Investigation

### Hypothesis 1
- **Theory**: ...
- **Evidence for**: ...
- **Evidence against**: ...
- **Verdict**: Confirmed / Rejected

### Hypothesis 2 (if needed)
- **Theory**: ...
- **Evidence**: ...
- **Verdict**: ...

## Root Cause

What is the actual root cause? In which file/method?

## Fix

What was changed to fix it?

```swift
// Before (broken)
...

// After (fixed)
...
```

## Regression Test

What test was added to prevent recurrence?

```swift
func testBugFix_condition_expectedBehavior() {
    // ...
}
```

## Lessons Learned

What can we do to prevent this class of bug?

## References

- Crash log:
- Related issues:
- Related commits:
