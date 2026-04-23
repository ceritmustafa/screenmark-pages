# Technical Design: [Title]

**Author**: [Name]
**Date**: [YYYY-MM-DD]
**Status**: Draft | Approved | Implemented | Superseded
**Related**: [Issue/Feature link]

---

## Problem Statement

What problem are we solving? Why now?

## Goals

1. Primary goal
2. Secondary goal

## Non-Goals

- What this design explicitly does NOT address

## Background

Relevant context, existing patterns, prior decisions. Link to related ADRs.

## Proposed Design

### Overview

High-level description of the approach.

### Detailed Design

#### New Types
```swift
// Key type signatures and protocols
protocol NewServiceProtocol {
    func operation() async throws -> Result
}
```

#### State Changes
How does this affect `ModeStateMachine` or `SceneState`?

#### Data Flow
```
User action → [describe flow] → UI update
```

#### Module Impact
| Module | Changes |
|--------|---------|
| ScreenMarkCore | |
| ScreenMarkServices | |
| ScreenMarkUI | |

### Error Handling
How are errors handled at each layer?

### Migration Plan
How do we get from the current state to the new design? What's the incremental path?

## Alternatives Considered

### Alternative A: [Name]
- **Approach**: ...
- **Pros**: ...
- **Cons**: ...
- **Why not**: ...

### Alternative B: [Name]
- **Approach**: ...
- **Pros**: ...
- **Cons**: ...
- **Why not**: ...

## Testing Strategy

- Unit tests:
- Integration tests:
- Manual verification:

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| | | | |

## Task Breakdown

| # | Task | Size | Dependencies |
|---|------|------|-------------|
| 1 | | S/M/L | — |
| 2 | | S/M/L | 1 |

## Open Questions

- [ ] Question 1
- [ ] Question 2
