# ScreenMark Quality Checklists

> Reference checklists for PR review, release, security, performance, accessibility, technical debt, bug triage, and incident debugging.

---

## PR Review Checklist

### Before Approving
- [ ] **Builds** without warnings: `swift build --product ScreenMarkApp`
- [ ] **Tests pass**: `swift test`
- [ ] **New behavior tested** with focused unit tests
- [ ] **Module boundaries** respected (no backward imports)
- [ ] **State flow** correct (action → state machine → published state → view)
- [ ] **Error handling** explicit (no silent `try?`, no empty `catch`)
- [ ] **Concurrency** safe (`@MainActor` where needed, no `@unchecked Sendable`)
- [ ] **Accessibility** labels on new interactive elements
- [ ] **Previews** on new SwiftUI views
- [ ] **Naming** clear and consistent with conventions
- [ ] **No force unwraps** in production code
- [ ] **Commit messages** clear and imperative

---

## Release Checklist

### Code
- [ ] All tests pass: `swift test`
- [ ] Build succeeds: `swift build --product ScreenMarkApp`
- [ ] No new compiler warnings
- [ ] Release gate passes: `./Scripts/release_gate.sh`

### Verification
- [ ] Core flows tested manually: freeze, draw, zoom, snip, whiteboard, record, break
- [ ] Multi-monitor behavior verified
- [ ] Permissions flow works (fresh and returning user)
- [ ] Hotkeys register and respond after fresh launch

### Performance
- [ ] Launch time <2 seconds
- [ ] Smooth overlay rendering (60fps) with 20+ annotations
- [ ] CPU <5% when idle
- [ ] No memory leaks in extended session

### Artifacts
- [ ] Version number updated in `BuildMetadata.swift`
- [ ] Release notes prepared
- [ ] App bundle built and signed: `./Scripts/build_app.sh`
- [ ] Notarized (if distributing outside App Store)

### Documentation
- [ ] KNOWN_ISSUES.md current
- [ ] README.md reflects released features
- [ ] ROADMAP.md updated

### Post-Release
- [ ] Git tag: `git tag -a v<X.Y.Z> -m "Release v<X.Y.Z>"`
- [ ] Next version prep (increment build number)

---

## Security Review Checklist

- [ ] No secrets (API keys, tokens, passwords) in source code
- [ ] Sensitive data stored in Keychain (not UserDefaults)
- [ ] No sensitive data in log output
- [ ] Permissions requested at point of use with clear explanation
- [ ] Permission denial handled gracefully
- [ ] File operations validate paths before read/write
- [ ] No `@unchecked Sendable` (thread safety risk)
- [ ] Hardened Runtime enabled for distribution builds
- [ ] App Sandbox evaluated (document exceptions in ADR)
- [ ] Screen capture data not persisted without user consent

---

## Performance Audit Checklist

- [ ] **Main thread**: No I/O or heavy computation on main thread
- [ ] **Launch**: Cold start under 2 seconds
- [ ] **Rendering**: 60fps overlay with normal annotation load
- [ ] **Memory**: No leaks (verify `deinit` called, Combine subscriptions cancelled)
- [ ] **CPU idle**: <5% when app is active but not in use
- [ ] **Capture pipeline**: Frame buffers released promptly
- [ ] **Recording**: No dropped frames at target resolution
- [ ] **Collections**: No O(n²) operations on growing collections
- [ ] **State updates**: `@Published` changes batched where possible
- [ ] **Timers**: Invalidated when not needed (energy impact)

---

## Accessibility Audit Checklist

- [ ] **VoiceOver**: All interactive elements announced with meaningful labels
- [ ] **Keyboard**: All features reachable via keyboard
- [ ] **Tab order**: Logical navigation sequence
- [ ] **Focus ring**: Visible on focused elements
- [ ] **Dynamic Type**: Layout adapts to largest text size
- [ ] **Reduce Motion**: Animations respect system setting
- [ ] **Increase Contrast**: UI elements visible with high contrast
- [ ] **Color independence**: No color-only indicators
- [ ] **Standard shortcuts**: ⌘W (close), ⌘Q (quit), ⌘, (settings)
- [ ] **Custom shortcuts**: Documented and non-conflicting

---

## Technical Debt Review Checklist

- [ ] **Correctness debt**: `@unchecked Sendable`, force unwraps, race conditions
- [ ] **Architecture debt**: God Objects (>300 LOC), missing protocols, tight coupling
- [ ] **Test debt**: Critical paths without test coverage
- [ ] **Documentation debt**: Public APIs without doc comments
- [ ] **Modernization debt**: Deprecated API usage, Swift 5 → 6 migration items
- [ ] **Maintainability debt**: Duplicated code, files >500 LOC, unclear naming
- [ ] **Items tracked**: New findings added to ROADMAP.md or TODO.md

---

## Bug Triage Playbook

### When a Bug is Reported
1. **Reproduce**: Can you trigger it reliably?
2. **Severity**: Crash? Data loss? Visual glitch? Minor inconvenience?
3. **Scope**: One feature? One platform? All users?
4. **Regression?**: Was this working before? Check `git log`.
5. **Root cause**: In which module? (Core, Services, UI)

### Priority Matrix
| Impact | Frequency | Priority |
|--------|-----------|----------|
| Crash / data loss | Any | 🔴 P0 — Fix immediately |
| Feature broken | Always | 🟠 P1 — Fix this cycle |
| Feature broken | Sometimes | 🟡 P2 — Plan fix |
| Visual/UX issue | Always | 🟡 P2 — Plan fix |
| Visual/UX issue | Rare | 🔵 P3 — Backlog |

### Bug Report Template
- **Symptom**: What happens?
- **Expected**: What should happen?
- **Steps**: How to reproduce?
- **Environment**: macOS version, display configuration
- **Logs**: Console output, crash log

---

## Incident / Debugging Playbook

### When the App Crashes or Misbehaves in Production

1. **Capture**: Crash log, console output, system state
2. **Reproduce**: Minimal steps to trigger
3. **Isolate**: Which module, which method, which state?
4. **Trace**: Follow data flow from trigger to failure
5. **Hypothesize**: Most likely root cause
6. **Verify**: Confirm with targeted test or logging
7. **Fix**: Minimal, surgical — address root cause
8. **Test**: Regression test that fails without the fix
9. **Document**: Add to KNOWN_ISSUES.md with resolution
10. **Prevent**: Consider what systemic change prevents this class of bug
