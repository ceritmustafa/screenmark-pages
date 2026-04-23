# ScreenMark Known Issues

## Entry Template
- `ID`
- `Symptom`
- `Root cause`
- `Resolution`
- `Regression test`
- `Prevention`

## Open

_(Tüm önceki open issue'lar Sprint 1-5'te kapatıldı — bkz. Resolved. Yeni issue tespit edilirse buraya eklenir.)_

## Resolved

- `ISS-001`
  Symptom: shipping overlay shows developer-oriented session diagnostics to end users.
  Root cause: overlay chrome renders raw mode/tool/element debug text as part of the production overlay surface.
  Resolution: dev toggle buttons removed in Sprint 1 (commit 520474e); overlay no longer exposes dev state in production.
  Regression test: release build manual verification — Sprint 1 manual check passed.
  Prevention: no developer-only chrome ships without an explicit debug gate.

- `ISS-002`
  Symptom: purchase, entitlement, and diagnostics flows were missing entirely.
  Root cause: prototype focused on overlay behavior without a productization layer.
  Resolution: StoreKit 2 full integration — Sprint 2-4 (commits 002712e, 6495d7d, eae930d). Entitlement cache, notifications, contextual paywall, purchasesPane redesign all complete.
  Regression test: AppCoordinatorTests (62 tests), EntitlementStoreTests (12 tests), all passing.
  Prevention: every monetized feature must ship with entitlement checks and restore coverage.

- `ISS-003`
  Symptom: large coordination files made safe changes expensive and error-prone.
  Root cause: lifecycle, capture, permissions, recording, and UI concerns accumulated in single files.
  Resolution: Sprint 5 — ScreenMarkServices.swift (1117 lines) split into 5 files; AppCoordinator (2223 lines) split into 5 extension files (total ~1151 lines core + 4 extensions).
  Regression test: 173 tests passing; AppCoordinatorTests expanded from 48 → 62.
  Prevention: new feature work must attach to an existing seam or create one first.

- `ISS-004`
  Symptom: status bar menu text was hard to read and the quick controls surface looked primitive.
  Root cause: the app relied on the system NSMenu presentation instead of a controlled, branded surface.
  Resolution: replace the menu with a custom SwiftUI popover that owns contrast, spacing, and action hierarchy.
  Regression test: build and smoke-test the status item flow during UI verification.
  Prevention: menu bar UI should ship from a single owned design surface, not default system menu styling.

- `ISS-005`
  Symptom: Launch at Login could appear enabled even when ServiceManagement registration failed.
  Root cause: settings were persisted before the login item operation succeeded.
  Resolution: persist the toggle only after a successful registration/unregistration call.
  Regression test: `SettingsStoreTests.testUpdateLaunchAtLoginPersistsOnlyAfterSuccessfulServiceUpdate`.
  Prevention: side-effectful preference writes must commit only after the platform API confirms success.
