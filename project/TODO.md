# ScreenMark TODO

## Rules
- Every implementation task must have an ID, acceptance criteria, and linked tests.
- Every bug fix must add a regression test and an entry to `KNOWN_ISSUES.md`.
- A task is not `Done` until build, automated tests, and manual acceptance checks are complete.

## Sprint 6 — App Store Readiness (Now)

- `SUB-001` Create `ScreenMark.entitlements` with App Sandbox + audio-input + user-selected file access.
  Acceptance: app builds with sandbox enabled; screen capture, global hotkeys, and microphone still function.
  Tests: manual verification on clean build.

- `SUB-002` Fix `Info.plist`: LSUIElement false → true (hide dock icon for menu bar app).
  Acceptance: app does not appear in Dock after relaunch.
  Tests: manual smoke test.

- `SUB-003` Bump version: CFBundleShortVersionString 0.1.0 → 1.0.0.
  Acceptance: About panel or system info shows 1.0.0.
  Tests: build smoke.

- `SUB-004` Verify bundle ID (`com.mustafacerit.screenmark`) matches App Store Connect team; confirm StoreKit product IDs exist in ASC.
  Acceptance: documented decision in RELEASE_GATE.md.
  Tests: StoreKit sandbox purchase test on device.

- `SUB-005` Document archive + signing pipeline (direct notarization or Xcode workspace approach).
  Acceptance: one documented command/script produces a signed .app ready for upload.
  Tests: archive smoke test.

- `SUB-006` Run `insecure-defaults` security audit; document findings.
  Acceptance: no critical or high findings unresolved; audit result in RELEASE_GATE.md.
  Tests: automated grep scans per skill checklist.

- `SUB-007` Complete manual test matrix from RELEASE_GATE.md (freeze, zoom, whiteboard, snip, recording, locked features, restore).
  Acceptance: all matrix items checked and signed off.
  Tests: manual.

## Sprint 7 — Store Listing (Next)

- `MKT-001` Produce 6 App Store screenshots at 2880×1800 (plan in APP_STORE_MARKETING_KIT.md).
- `MKT-002` Record 20-25 second app preview video (script in APP_STORE_MARKETING_KIT.md).
- `MKT-003` Write App Store metadata: description, keywords, age rating, privacy policy URL, support URL.

## Sprint 8 — v1.0 Quality (Later)

- `QUAL-003` Swift 6 strict concurrency — Package.swift .v5 → .v6; resolve all actor isolation warnings.
- `QUAL-004` Extend os.Logger to ScreenMarkCore and ScreenMarkUI modules (currently only ScreenMarkServices).
- `PERF-001` Recording pipeline DisplayLink-based migration (B5 from ROADMAP).
- `PERF-002` Display hot-plug race condition fix (B4 from ROADMAP).

## Done

- `BASE-001` Repository inspection, architecture review, and execution plan completed on 2026-03-09.
- `MAS-001` App Store feasibility spike: screen capture (SCStream + TCC), global hotkeys (Carbon RegisterEventHotKey + NSEvent.addGlobalMonitorForEvents + Accessibility TCC), launch at login (SMAppService) — all sandbox-compatible. Documented 2026-04-14.
- `ARCH-001` Split monolithic coordination behind protocol seams — AppCoordinator 5 files, ScreenMarkServices 5 files (Sprint 5).
- `COM-001` StoreKit 2 lifetime + subscription + trial foundation — Sprint 1-4.
- `COM-002` Entitlement precedence (lifetime > subscription > trial > free) — implemented in EntitlementState.hasProAccess.
- `COM-003` Monthly subscription + migration-safe feature gating — Sprint 2-3.
- `QUAL-001` Persistent diagnostics and known-issue memory (DiagnosticsStore) — complete.
- `UX-001` Remove shipping-debug overlay chrome — Sprint 1 (dev toggles removed, ISS-001 closed).
- `UI-001` Multi-section settings (purchases as first-class tab) — Sprint 3-4.
- `UI-002` Onboarding, permission walkthrough, and paywall comparison — Sprint 3 (OnboardingView, ProUpgradeSheet).
- `REL-001` Release gate script — RELEASE_GATE.md exists.
