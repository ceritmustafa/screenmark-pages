# ScreenMark Testing Strategy

> Complete testing strategy for the ScreenMark codebase.

---

## Test Architecture

### Test Targets
| Target | Tests For | Dependencies |
|--------|-----------|-------------|
| `ScreenMarkCoreTests` | Models, state machines, business logic | ScreenMarkCore |
| `ScreenMarkServicesTests` | Settings, commerce, diagnostics | ScreenMarkCore, ScreenMarkServices |
| `ScreenMarkUITests` | Coordinator logic, export | ScreenMarkCore, ScreenMarkUI |

### Test Pyramid
```
        ╱ Manual ╲           Targeted: visual, multi-monitor, permissions
       ╱──────────╲
      ╱ Integration╲        Some: coordinator + stubbed services
     ╱──────────────╲
    ╱   Unit Tests   ╲      Majority: logic, state, computation
   ╱──────────────────╲
```

---

## Unit Testing

### What to Unit Test
| Layer | What to Test | Priority |
|-------|-------------|----------|
| Core | State machine transitions (every mode × every action) | 🔴 Critical |
| Core | Feature access for every entitlement state | 🔴 Critical |
| Core | Shortcut resolver for all hotkey actions | 🟠 High |
| Core | Model encoding/decoding, equality, computed props | 🟡 Medium |
| Services | SettingsStore persistence, defaults, migration | 🔴 Critical |
| Services | CommerceService entitlement state transitions | 🟠 High |
| Services | DiagnosticsService event logging | 🟡 Medium |
| UI | Coordinator action dispatching | 🟠 High |
| UI | SnapshotExporter format handling | 🟡 Medium |

### Test Structure: Arrange-Act-Assert
```swift
func testToggleFreeze_whenInDrawMode_returnsToDrawMode() {
    // Arrange
    let machine = ModeStateMachine()
    machine.toggleMode(.overlay)
    machine.setTool(.pen)

    // Act
    machine.toggleMode(.freeze)
    machine.toggleMode(.freeze) // toggle back

    // Assert
    XCTAssertEqual(machine.sceneState.activeMode, .overlay)
    XCTAssertEqual(machine.sceneState.activeTool, .pen)
}
```

### Test Naming
```
test<Behavior>_<condition>_<expected>

testUndo_whenStackEmpty_doesNothing()
testFeatureAccess_trialExpired_blocksProFeatures()
testStartCapture_whenPermissionDenied_throwsError()
```

---

## Stubs and Fakes

### Design Principle
Stubs are simple, controllable implementations of service protocols. They record calls and return configured values.

```swift
@MainActor
final class StubCaptureEngine: CaptureEngineProtocol {
    var isCapturing = false
    var shouldFail = false
    var startCallCount = 0

    func startCapture(for display: CGDirectDisplayID) async throws {
        startCallCount += 1
        if shouldFail { throw CaptureError.permissionDenied }
        isCapturing = true
    }

    func stopCapture() {
        isCapturing = false
    }
}
```

### Rules
- Stubs live in the test target that uses them (in a `Helpers/` subfolder)
- One stub per service protocol
- Stubs are deterministic — no randomness, no timing
- No dynamic mocking libraries — Swift protocols are sufficient

### Existing Stubs
- `StubPurchaseService` — in `ScreenMarkServicesTests`
- `StubNotificationService` — in `ScreenMarkServicesTests`

---

## Async Testing

```swift
@MainActor
func testAsyncCapture_success() async throws {
    let engine = StubCaptureEngine()
    let coordinator = CaptureCoordinator(engine: engine)

    try await coordinator.startCapture(displayID: CGMainDisplayID())

    XCTAssertTrue(engine.isCapturing)
    XCTAssertEqual(engine.startCallCount, 1)
}
```

- Use `async` test methods directly (Xcode 16+)
- Mark `@MainActor` if testing main-actor-isolated types
- Avoid `XCTestExpectation` + `waitForExpectations` for simple async
- Use `Task.yield()` when testing Combine `@Published` updates

---

## Deterministic Testing

### Rules
- No `sleep()` or `Task.sleep()` in unit tests
- No network calls
- No file system reads/writes (use in-memory stubs)
- No dependency on system clock (inject `Date` provider if needed)
- No dependency on test execution order

### For Timing-Dependent Code
```swift
// Instead of using real timers:
protocol TimeProvider {
    var now: Date { get }
}

struct StubTimeProvider: TimeProvider {
    var now: Date = Date(timeIntervalSince1970: 1000)
}
```

---

## What Not to Test

- **SwiftUI view layout**: Use `#Preview` + manual verification
- **AppKit window positioning**: Integration/manual testing territory
- **Apple framework internals**: Don't test that `URLSession` works
- **Trivial code**: Simple stored properties with no logic
- **Private methods**: Test through the public interface

---

## Test Coverage Goals

| Milestone | Target | Focus |
|-----------|--------|-------|
| v0.2 | 20% | Core state machine, Settings, Feature access |
| v0.3 | 35% | Service protocols + stubs, Coordinator testing |
| v0.4 | 45% | Error handling, edge cases, async paths |
| v1.0 | 50%+ | Comprehensive coverage, no critical gaps |

Current: ~7.4% (29 tests)

### Coverage is a Metric, Not a Goal
- Cover **behavior**, not lines
- 80% coverage with meaningless tests is worse than 40% with focused ones
- Focus on: state transitions, error handling, business logic

---

## Running Tests

```bash
swift test                              # All tests
swift test --filter ScreenMarkCoreTests     # One target
swift test --filter ModeStateMachineTests  # One class
swift test --parallel                   # Parallel execution
```

---

## Adding a New Test File

1. Create `Tests/<Target>/<Subject>Tests.swift`
2. `import XCTest` and `@testable import <Module>`
3. Create `final class <Subject>Tests: XCTestCase`
4. Add `setUp()` if shared fixture needed
5. Write tests following naming convention
6. Run `swift test --filter <Subject>Tests`
7. Verify test appears in full suite: `swift test`
