# ScreenMark Engineering Handbook

> Authoritative reference for engineering standards, conventions, and expectations.

---

## 1. Swift Coding Standards

### Style Rules
- **Indentation**: 4 spaces, no tabs
- **Line length**: Soft limit 120 characters
- **Braces**: Same-line opening brace
- **Trailing commas**: Use in multi-line collections and argument lists
- **Self**: Omit `self.` unless required for disambiguation or closure capture
- **Access control**: Explicit — default to `private`, promote as needed

### Naming
| Entity | Convention | Example |
|--------|-----------|---------|
| Types, protocols | `UpperCamelCase` | `OverlayWindowManager`, `CaptureEngineProtocol` |
| Functions, properties | `lowerCamelCase` | `toggleFreezeMode()`, `strokeWidth` |
| Enum cases | `lowerCamelCase` | `.drawingPen`, `.zoomClassic` |
| Constants | `lowerCamelCase` | `let maxUndoDepth = 100` |
| Type parameters | Single uppercase or descriptive | `<Element>`, `<Output>` |
| File names | Match primary type | `ModeStateMachine.swift` |
| Test files | `<Subject>Tests.swift` | `ModeStateMachineTests.swift` |

### File Organization
```swift
// 1. Imports (minimal)
import SwiftUI

// 2. Type declaration with doc comment
/// Brief description.
@MainActor
final class MyCoordinator: ObservableObject {

    // 3. Public properties
    @Published var state: State

    // 4. Private properties
    private let service: ServiceProtocol
    private var cancellables = Set<AnyCancellable>()

    // 5. Initialization
    init(service: ServiceProtocol) { ... }

    // 6. Public methods
    func performAction() { ... }

    // 7. Private methods
    private func handleStateChange() { ... }
}

// 8. Protocol conformances (in extensions)
extension MyCoordinator: SomeProtocol { ... }
```

---

## 2. SwiftUI Architecture Conventions

### View Composition
- **Max 80 lines per view** — extract subviews as separate structs
- **No business logic in `body`** — call coordinator/state machine methods
- **State ownership clear** — `@State` for local, `@ObservedObject` for external
- **Previews required** — `#Preview` block on every new view

### State Management
- `ModeStateMachine` is the single source of truth for scene state
- Views observe `@Published SceneState` reactively
- Actions flow up (view → coordinator → state machine)
- State flows down (state machine → published state → view)

### View-Coordinator Relationship
```swift
struct ToolPalette: View {
    @ObservedObject var machine: ModeStateMachine

    var body: some View {
        HStack {
            ForEach(ToolKind.allCases) { tool in
                ToolButton(tool: tool, isSelected: machine.sceneState.activeTool == tool) {
                    machine.setTool(tool)
                }
            }
        }
    }
}
```

---

## 3. Testing Strategy

### Test Pyramid
1. **Unit tests** (majority): Pure logic, state machines, services with stubs
2. **Integration tests** (some): Coordinator + stubbed services working together
3. **Manual testing** (targeted): UI, multi-monitor, permissions, visual correctness

### What Must Be Tested
- All state machine transitions
- All business logic and computations
- Error handling paths
- Settings persistence and migration
- Feature gating decisions

### What Not to Unit Test
- SwiftUI view layout (use previews + manual verification)
- Apple framework behavior (ScreenCaptureKit, StoreKit internals)
- Trivial code with no logic

### Naming: `test<Behavior>_<condition>_<expected>()`
```swift
func testToggleFreeze_whenDrawing_returnsToDrawMode()
func testUndo_whenStackEmpty_doesNothing()
```

### Stubs Over Mocks
```swift
final class StubCaptureEngine: CaptureEngineProtocol {
    var startCalled = false
    var shouldFail = false

    func startCapture() async throws {
        startCalled = true
        if shouldFail { throw CaptureError.permissionDenied }
    }
}
```

---

## 4. Error Handling Strategy

### Rules
1. **No silent `try?`** without a comment explaining why the error is ignorable
2. **No empty `catch {}`** — at minimum, log the error
3. **No `try!`** in production code
4. **Typed errors** for services: `enum CaptureError: LocalizedError`
5. **Log before handling**: `Logger.capture.error("Failed: \(error)")`
6. **User-facing errors** must have `LocalizedError` conformance

### Error Flow
```
Service throws typed error
  → Coordinator catches and logs
  → Coordinator decides: retry / notify user / degrade gracefully
  → User sees actionable message (if relevant)
```

---

## 5. Dependency Management Principles

- **Zero external dependencies** — this is a feature, not a limitation
- Apple frameworks provide everything needed: SwiftUI, AppKit, AVFoundation, ScreenCaptureKit, StoreKit2
- Evaluate any proposed dependency against: maintenance, binary size, platform support, license
- If a dependency is truly needed: pin version explicitly, document rationale in ADR

---

## 6. State Management Principles

- **Single source of truth**: `ModeStateMachine.sceneState`
- **Unidirectional flow**: User action → state mutation → UI update
- **Observable**: `@Published` properties observed via Combine or SwiftUI
- **Deterministic**: Same input + same state = same output
- **Undo-capable**: State snapshots captured before mutations
- **No scattered state**: Settings in `SettingsStore`, scene state in `ModeStateMachine`

---

## 7. Networking Principles

(For future networking features)
- `async/await` only — no completion handlers
- Protocol-based API clients for testability
- URLSession — no third-party HTTP libraries
- HTTPS only, respect App Transport Security
- Typed error enums with meaningful cases
- No sensitive data in logs

---

## 8. Persistence Principles

- **UserDefaults** for simple key-value preferences
- **Keychain** for sensitive data (tokens, credentials)
- **File system** for large data (recordings, exports)
- **Type-safe keys** — no string literal keys scattered in code
- **Migration versioning** — schema version tracked, incremental migration
- **Protocol abstraction** — every store has a protocol for testing

---

## 9. Accessibility Expectations

- Every interactive element has `.accessibilityLabel()`
- Stateful elements have `.accessibilityValue()`
- Logical VoiceOver reading order
- Full keyboard navigation for all features
- Dynamic Type support (`@ScaledMetric`)
- Respect Reduce Motion, Increase Contrast system settings
- Color is never the sole indicator of state

---

## 10. Performance Expectations

- App launch under 2 seconds
- Overlay rendering at 60fps with 20+ annotations
- CPU <5% when idle
- No memory leaks in 30-minute session
- Main thread never blocked by I/O or computation
- Recording pipeline: no frame drops at target resolution

---

## 11. Security & Privacy Expectations

- No secrets in source code
- Keychain for sensitive data
- Least-privilege permissions (request when needed, explain why)
- Screen capture data never persisted without user consent
- Log nothing sensitive (no tokens, no passwords, no capture content)
- Hardened Runtime for distribution
- App Sandbox when feasible

---

## 12. Documentation Expectations

- All public types and methods have `///` DocC comments
- Complex logic has inline comments explaining *why*
- Architecture decisions recorded as ADRs
- Project status in `docs/project/` (KNOWN_ISSUES, LESSONS_LEARNED)
- No documentation that just restates the code

---

## 13. Pull Request Expectations

- **Title**: Clear, imperative, under 72 characters
- **Description**: What changed, why, how to test
- **Scope**: Single concern per PR (don't mix features and refactors)
- **Tests**: New behavior has tests; all tests pass
- **Build**: No new compiler warnings
- **Review**: Self-review before requesting review
- **Screenshots**: For any visual change

---

## 14. Definition of Done

A task is "done" when:
- [ ] Implementation complete and compiles without warnings
- [ ] Tests written and passing
- [ ] Accessibility considered (labels, keyboard, VoiceOver)
- [ ] Error handling explicit
- [ ] Documentation updated (doc comments, external docs if affected)
- [ ] `swift test` passes
- [ ] Code reviewed (self or peer)
- [ ] No regressions in existing behavior

---

## 15. Release Readiness Checklist

- [ ] All tests pass
- [ ] No new compiler warnings
- [ ] Version number updated
- [ ] Release notes written
- [ ] Key flows manually verified
- [ ] Performance acceptable
- [ ] Accessibility audit passed
- [ ] Signed and notarized (for distribution)
- [ ] `./Scripts/release_gate.sh` passes
- [ ] KNOWN_ISSUES.md updated
