# Developer Onboarding Guide

> Welcome to the ScreenMark codebase. This guide gets you productive quickly.

---

## 1. Prerequisites

- macOS 15 (Sequoia) or later
- Xcode 16 or Swift 6.1 toolchain
- Git

## 2. Setup

```bash
# Clone
git clone <repository-url>
cd ScreenMark

# Install git hooks
./Scripts/setup_git_hooks.sh

# Build
swift build --product ScreenMarkApp

# Test
swift test

# Run
./Scripts/open_app.sh
```

## 3. What Is ScreenMark?

ScreenMark is a macOS presentation overlay tool. It provides:
- Cursor zoom (classic + live)
- Drawing (pen, highlighter, shapes, arrows, text)
- Freeze mode (capture + annotate)
- Whiteboard / blackboard
- Screen capture (snip)
- Recording
- Break timer

It runs as a **menu bar app** with global hotkeys for activation.

## 4. Architecture in 60 Seconds

```
ScreenMarkCore      → Domain models + state machines (no platform code)
ScreenMarkServices  → Platform integrations (capture, hotkeys, settings, commerce)
ScreenMarkUI        → SwiftUI views + AppKit coordinators + overlays
ScreenMarkApp       → App entry point
```

Key pattern: **Unidirectional data flow**
```
User action → Coordinator → ModeStateMachine → @Published SceneState → Views
```

## 5. Key Files to Read First

| # | File | Why |
|---|------|-----|
| 1 | `Sources/ScreenMarkCore/ScreenMarkModels.swift` | All domain types |
| 2 | `Sources/ScreenMarkCore/ModeStateMachine.swift` | Central state management |
| 3 | `Sources/ScreenMarkUI/AppCoordinator.swift` | Main orchestrator |
| 4 | `ROADMAP.md` | Development plan and known issues |
| 5 | `docs/engineering/handbook.md` | Engineering standards |
| 6 | `docs/engineering/architecture-guide.md` | Architecture details |

## 6. Development Workflow

```bash
# Build and launch
./Scripts/open_app.sh

# Run tests
swift test

# CI check (same as GitHub Actions)
./Scripts/ci_check.sh

# Skip auto-push (git hook) for one commit
SCREENMARK_SKIP_AUTO_SYNC=1 git commit -m "WIP: ..."
```

## 7. Conventions Quick Reference

- 4-space indent
- `UpperCamelCase` types, `lowerCamelCase` members
- One type per file, file named after type
- `@MainActor` on UI classes
- State changes through `ModeStateMachine` only
- Tests: `test<Behavior>_<condition>_<expected>()`
- Commit: short imperative subjects

## 8. AI-Assisted Development

This repo includes Copilot agents and skills in `.github/prompts/`:

```
.github/prompts/agents/    → 18 specialized AI agents
.github/prompts/skills/    → 20 reusable playbooks
```

Invoke in VS Code Copilot Chat or reference in prompts.

## 9. Where to Find Things

| Need | Location |
|------|----------|
| Coding standards | `docs/engineering/handbook.md` |
| Architecture guidance | `docs/engineering/architecture-guide.md` |
| Testing guidance | `docs/engineering/testing-strategy.md` |
| Quality checklists | `docs/engineering/quality-checklists.md` |
| Document templates | `docs/templates/` |
| Known issues | `docs/project/KNOWN_ISSUES.md` |
| Development roadmap | `ROADMAP.md` |

## 10. Getting Help

- Read the relevant agent prompt in `.github/prompts/agents/`
- Follow the matching skill playbook in `.github/prompts/skills/`
- Check `docs/project/LESSONS_LEARNED.md` for past pitfalls
- Check `docs/project/KNOWN_ISSUES.md` for tracked problems
