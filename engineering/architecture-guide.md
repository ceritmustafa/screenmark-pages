# ScreenMark Architecture Guide

> Practical architecture guidance for building and evolving the ScreenMark codebase.

---

## Module Architecture

### Dependency Graph
```
┌─────────────┐
│  ScreenMarkApp   │  Entry point, NSApplicationDelegate
└──────┬───────┘
       │ imports
┌──────▼───────┐
│  ScreenMarkUI    │  SwiftUI views, AppKit coordinators, overlays
└──────┬───────┘
       │ imports
┌──────▼───────┐
│ScreenMarkServices│  Platform integrations (capture, hotkeys, settings, commerce)
└──────┬───────┘
       │ imports
┌──────▼───────┐
│  ScreenMarkCore  │  Domain models, state machines, pure logic
└──────────────┘
       │ imports
   Foundation only
```

### Rules
- **No backward imports**: Core never imports Services; Services never imports UI
- **Core is platform-agnostic**: No AppKit, SwiftUI, ScreenCaptureKit, etc.
- **UI owns presentation**: All views, windows, and coordinators
- **Services own platform**: All Apple framework integrations

---

## Feature-First Organization

When adding a feature that touches multiple layers, work bottom-up:

```
1. ScreenMarkCore:    Add models, state machine support, business logic
2. ScreenMarkServices: Add or extend service (with protocol)
3. ScreenMarkUI:      Add views, wire to coordinator
4. Tests:         Test each layer independently
```

Each layer can be built and tested independently. Core and Services tests don't need UI. UI tests use stubbed services.

---

## Presentation Pattern

### ModeStateMachine (State Holder)
```swift
@MainActor
final class ModeStateMachine: ObservableObject {
    @Published private(set) var sceneState: SceneState

    func toggleMode(_ mode: AppMode) { ... }
    func setTool(_ tool: ToolKind) { ... }
    func undo() { ... }
}
```

- **Single source of truth** for scene state
- **Named methods** for all mutations (no external property setting)
- **Automatic undo** snapshots before each mutation
- **Deterministic**: Same input + state = same output

### AppCoordinator (Orchestrator)
```swift
@MainActor
final class AppCoordinator: ObservableObject {
    let machine: ModeStateMachine
    let captureEngine: CaptureEngineProtocol
    let settingsStore: SettingsStoreProtocol
    // ...

    func handleHotkey(_ action: HotkeyAction) { ... }
}
```

- Subscribes to `machine.$sceneState` for reactive updates
- Coordinates service calls (start capture, start recording, etc.)
- Manages overlay window lifecycle via `OverlayWindowManager`
- **Currently a God Object — decomposition in progress**

### Target Decomposition
```
AppCoordinator (orchestration only)
├── CaptureCoordinator (capture, freeze, zoom)
├── RecordingCoordinator (recording pipeline)
├── InputCoordinator (hotkeys, keyboard, scroll)
└── BreakCoordinator (break timer)
```

---

## Domain Boundaries

### Core Domain
Models that represent the app's problem space:
- `AppMode`: overlay, freeze, zoom, whiteboard, blackboard, snip, break
- `ToolKind`: pen, highlighter, arrow, rectangle, ellipse, text, blur, spotlight
- `SceneState`: composite state of the current session
- `DrawingElement`: individual annotation element
- `ColorProfile`: named color with hex value

### Service Domain
Platform capabilities abstracted behind protocols:
- `CaptureEngineProtocol`: start/stop screen capture
- `RecordingServiceProtocol`: start/stop video recording
- `SettingsStoreProtocol`: read/write user preferences
- `HotkeyEngineProtocol`: register/unregister global hotkeys

### UI Domain
Presentation and coordination:
- SwiftUI views observe state, dispatch actions
- AppKit coordinators manage windows and system integration
- No business logic — only presentation logic

---

## Service Abstractions

### Protocol + Concrete + Stub Triplet
```swift
// Protocol (in ScreenMarkServices)
protocol CaptureEngineProtocol: AnyObject {
    func startCapture(for display: CGDirectDisplayID) async throws
    func stopCapture()
    var isCapturing: Bool { get }
}

// Concrete (in ScreenMarkServices)
@MainActor
final class CaptureEngine: CaptureEngineProtocol { ... }

// Stub (in Tests)
final class StubCaptureEngine: CaptureEngineProtocol {
    var isCapturing = false
    var shouldFail = false
    func startCapture(for display: CGDirectDisplayID) async throws {
        if shouldFail { throw CaptureError.permissionDenied }
        isCapturing = true
    }
    func stopCapture() { isCapturing = false }
}
```

---

## Dependency Injection

### Approach: Init-Based Injection
```swift
@MainActor
final class AppCoordinator {
    init(
        machine: ModeStateMachine,
        captureEngine: CaptureEngineProtocol,
        settingsStore: SettingsStoreProtocol,
        hotkeyEngine: HotkeyEngineProtocol
    ) { ... }
}
```

- **No DI container** — init parameters are sufficient for this codebase size
- Production code passes real implementations
- Test code passes stubs
- Composition happens at the app entry point (`ScreenMarkApplication`)

### Why Not a DI Container?
- 4 modules, ~15 services — container is over-engineering
- Init injection is explicit, compiler-verified, and easy to trace
- Container introduces indirection and makes dependencies less visible
- Revisit when service count exceeds 20+

---

## Side-Effect Management

### Pure vs. Effectful
- **Pure** (ScreenMarkCore): State transitions, model logic — no I/O, no platform calls
- **Effectful** (ScreenMarkServices): Capture, recording, file I/O, StoreKit, notifications
- **Coordination** (ScreenMarkUI): Wires pure logic to effects, manages lifecycle

### Actor Isolation
- `@MainActor` for anything touching UI or AppKit
- Dedicated actors for concurrent state (if needed for capture buffers)
- `async/await` for crossing isolation boundaries

---

## Test Seams

Every boundary between modules is a test seam:
- **Core ↔ Services**: Services use Core models; Core is tested alone
- **Services ↔ UI**: UI uses service protocols; stubs replace real services in tests
- **State machine**: Fully testable by calling methods and asserting `sceneState`
- **Coordinator**: Testable with stubbed services and state machine

---

## Preview Strategy

### Every View Gets a Preview
```swift
#Preview("Default State") {
    ToolPalette(machine: PreviewStateMachine.drawing)
}

#Preview("Empty State") {
    ToolPalette(machine: PreviewStateMachine.empty)
}
```

### Preview Helpers
Create `PreviewStateMachine` with pre-configured states:
```swift
enum PreviewStateMachine {
    static var drawing: ModeStateMachine {
        let m = ModeStateMachine()
        m.toggleMode(.overlay)
        m.setTool(.pen)
        return m
    }
}
```

---

## Adding a New Module

If the codebase grows enough to warrant a new module:

1. Define the boundary: what's in, what's out
2. Update `Package.swift` with the new target and dependencies
3. Follow the existing pattern: separate test target, matching folder structure
4. Ensure no circular dependencies
5. Document in ADR
6. Update this guide's dependency diagram
