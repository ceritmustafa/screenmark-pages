# Refactor Proposal: [What's Being Refactored]

**Author**: [Name]
**Date**: [YYYY-MM-DD]
**Status**: Proposed | Approved | In Progress | Done
**Affected Files**: [List key files]

---

## Current State

Describe the current code and its problems.

- **File/Type**: ...
- **Lines of code**: ...
- **Responsibilities**: ... (list all — if >3, it's too many)
- **Test coverage**: ...

## Problem

What specifically is wrong? Why does it need refactoring?

- [ ] Too large (>300 lines)
- [ ] Multiple responsibilities
- [ ] Untestable (tight coupling)
- [ ] Duplicated code
- [ ] Unclear boundaries
- [ ] Other: ...

## Proposed Refactoring

### Target Structure
```
Before:
  GodObject.swift (1000+ lines, 5 responsibilities)

After:
  FocusedTypeA.swift (~150 lines, 1 responsibility)
  FocusedTypeB.swift (~120 lines, 1 responsibility)
  FocusedTypeC.swift (~100 lines, 1 responsibility)
  SlimmedOriginal.swift (~200 lines, coordination only)
```

### Extraction Plan
| Step | What | From | To | Size |
|------|------|------|----|------|
| 1 | Extract X | Original | NewTypeA | M |
| 2 | Extract Y | Original | NewTypeB | M |
| 3 | Clean up | Original | — | S |

## Constraints

- [ ] Public API must not change
- [ ] All existing tests must pass unchanged
- [ ] One extraction per commit
- [ ] Other: ...

## Testing Plan

- Existing tests provide safety net: [ ] Yes / [ ] Need to add first
- New tests needed after refactoring:

## Risks

| Risk | Mitigation |
|------|------------|
| | |

## Definition of Done

- [ ] All tests pass
- [ ] No type exceeds 300 lines
- [ ] Each type has single responsibility
- [ ] No new compiler warnings
- [ ] Documentation updated
