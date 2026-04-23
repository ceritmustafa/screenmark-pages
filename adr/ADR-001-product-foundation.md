# ADR-001: Package Modules With Productization Layers

## Status
Accepted

## Context
ScreenMark started as a functional prototype. Overlay behavior, capture, permissions, recording, and UI coordination are tightly coupled. The product now needs App Store readiness, monetization, diagnostics, and long-term maintainability.

## Decision
- Keep Swift package modules as the primary code organization.
- Add productization layers instead of continuing to grow monolithic files.
- Treat commerce, diagnostics, and entitlement state as first-class application concerns.
- Keep the app free to download and gate Pro capabilities behind a lifetime unlock.
- Build entitlement resolution so a future monthly subscription can be added without redesigning the core access model.

## Consequences
- More explicit protocols and stores are required up front.
- AppCoordinator will shrink over time as responsibilities move outward.
- Tests become cheaper because purchase and diagnostics flows can be mocked directly.
- App Store submission work still needs an archive/signing shell, but core logic remains package-friendly.
