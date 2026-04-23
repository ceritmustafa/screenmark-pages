# ScreenMark — Project Info

> Last updated: 2026-03-23

## Overview

ScreenMark is a macOS presentation and screen annotation tool. It runs as a menu bar app and provides zoom, drawing, snipping, recording, whiteboard/blackboard, and break timer features — all accessible via global keyboard shortcuts or the status bar popover.

## Architecture

| Layer | Target | Description |
|-------|--------|-------------|
| **Core** | `ScreenMarkCore` | Value types, state machine, shortcut resolver, feature gating |
| **Services** | `ScreenMarkServices` | Settings, hotkey engine, capture engine, recording, permissions, commerce |
| **UI** | `ScreenMarkUI` | AppCoordinator, overlay windows, status bar, settings/onboarding views, snapshot export |
| **App** | `ScreenMarkApp` | Entry point, AppDelegate |

**No external dependencies** — pure Apple frameworks (ScreenCaptureKit, AVFoundation, StoreKit, SwiftUI, AppKit).

## Key Patterns

- **Coordinator + State Machine**: `AppCoordinator` orchestrates all subsystems; `ModeStateMachine` manages scene state with unidirectional data flow.
- **Per-Display Overlay Windows**: `OverlayWindowManager` creates one frameless `NSWindow` (level: `.screenSaver`) per connected display. Windows are rebuilt on display change notifications.
- **Carbon Hotkey Engine**: Global shortcuts registered via `EventTap` / Carbon Event Manager. Session-level keys handled by `OverlayHostingView.keyDown()`.
- **Combine Subscriptions**: `machine.$sceneState` drives async UI updates (overlay sync, status bar tint, capture state).

## Module Dependency Graph

```
ScreenMarkApp
├── ScreenMarkUI
│   ├── ScreenMarkServices
│   │   └── ScreenMarkCore
│   └── ScreenMarkCore
├── ScreenMarkServices
│   └── ScreenMarkCore
└── ScreenMarkCore
```

## Supported Modes

| Mode | Shortcut (default) | Description |
|------|-------------------|-------------|
| Zoom (Classic) | ⌃⌥Z | Magnified full-screen/lens/pinned view |
| Live Zoom | ⌃⌥L | Real-time smooth zoom (Pro) |
| Draw | ⌃⌥D | Pen/highlighter free-form drawing |
| Text | ⌃⌥T | Text annotation (left/right aligned) |
| Shape | ⌃⌥S | Rectangle, ellipse, arrow |
| Snip | ⌃⌥X | Screenshot selection → clipboard/file |
| Freeze | ⌃⌥F | Freeze screen frame |
| Whiteboard | ⌃⌥W | White background canvas |
| Blackboard | ⌃⌥B | Black background canvas |
| Timer | ⌃⌥K | Break countdown timer |
| Record | ⌃⌥R | Screen recording (Pro) |
| Leader Palette | ⌃⌥Space | Command palette overlay |

## Data Flow

```
Global Hotkey → HotkeyEngine → AppCoordinator.handle()
                                      ↓
                              ModeStateMachine.handle()
                                      ↓
                            @Published SceneState updated
                                      ↓
                    ┌─────────────────┼──────────────────┐
                    ↓                 ↓                  ↓
          OverlaySceneView   StatusItemController  OverlayWindowManager
          (renders scene)    (updates tint)        (sync visibility)
```

## Build & Test

```bash
swift build --product ScreenMarkApp     # Debug build
swift test                          # Run all tests
./Scripts/build_app.sh              # Release build → ~/Applications/ScreenMark.app
./Scripts/open_app.sh               # Build + launch
./Scripts/ci_check.sh               # CI: build + test
```

## Key Files

| File | Lines | Role |
|------|-------|------|
| `AppCoordinator.swift` | ~1100+ | Main coordinator, handles all mode switching |
| `ModeStateMachine.swift` | ~440 | Scene state management, undo, drawing |
| `OverlayWindowManager.swift` | ~170 | Per-display window lifecycle |
| `OverlayViews.swift` | ~1000+ | SwiftUI overlay rendering |
| `StatusItemController.swift` | ~325 | Menu bar popover UI |
| `ScreenMarkServices.swift` | ~1090 | Settings, hotkeys, capture, recording |
| `SnapshotExporter.swift` | ~900+ | PDF/PNG/TIFF export, snip crop |
| `ScreenMarkModels.swift` | ~450 | All value types and enums |
