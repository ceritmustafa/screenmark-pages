# Submission Checklist

## Apple account prerequisites

1. Confirm Apple Developer Program membership is active.
2. Accept all pending App Store Connect agreements.
3. Complete tax and banking setup.
4. Create the app record with SKU `screenmark-macos-1`.

## App Store Connect setup

1. Create subscription group `ScreenMark Pro`.
2. Add `screenmark_pro_monthly` and `screenmark_pro_yearly` as auto-renewable subscriptions.
3. Add `screenmark_pro_lifetime` as a non-consumable purchase.
4. Configure a 7-day free-trial introductory offer for eligible new subscribers on the subscription group.
5. Enable all territories.
6. Upload localized metadata for `en`, `tr`, `de`, `fr`, `es`, `ja`, `pt-BR`, and `zh-Hans`.
7. Set marketing, support, and privacy URLs to the GitHub Pages URLs in `product-matrix.json`.

## Repo and binary checks

1. Run `swift test`.
2. Run `swift build --product ScreenMarkApp`.
3. Run `./Scripts/release_gate.sh`.
4. Run `DRY_RUN=1 ./Scripts/archive_mas.sh`.
5. Produce a signed MAS archive with real identities and provisioning profile.
6. Validate and upload with Transporter or Apple tooling.

## Manual sandbox validation

1. Fresh install on free tier.
2. Monthly introductory trial flow.
3. Yearly introductory trial flow.
4. Restore purchases.
5. Lifetime purchase.
6. Subscription expiry and cancellation behavior.
7. Introductory-offer ineligible user behavior.

## Manual UX validation

1. Permissions on a fresh machine.
2. Multi-display zoom, snip, freeze, and recording.
3. Microphone and system audio recording quality.
4. In-app App Store messaging in all 8 app locales.
