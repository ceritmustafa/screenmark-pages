# App Review Notes

## Core explanation

ScreenMark is a local-first macOS presentation overlay app. It adds zoom, annotation, freeze frame, whiteboard, snip, timer, and recording controls from the menu bar.

## Why Screen Recording permission is required

Screen Recording is required for the app's core overlay and capture features:

- live zoom
- freeze frame
- snip / snapshot export
- screen recording

The app does not capture the screen until the user explicitly activates a feature.

## Why Microphone permission is required

Microphone access is only used when the user enables microphone capture for a recording session. If the user does not record with microphone input, the app does not use the microphone.

## Why Accessibility / global hotkey behavior exists

ScreenMark provides global shortcuts so presenters can trigger zoom, annotation, whiteboard, timer, and related actions while presenting in another app. Accessibility or input-monitoring style access is used only so the app can react to those shortcut events.

The app does not log or upload typed content.

## Commerce and privacy notes

- No account is required.
- Purchases are handled through the App Store with StoreKit.
- No user content is sent to a ScreenMark server.
- Exported images and recordings stay on the user's Mac unless the user moves them elsewhere.
