# devicemotion-circle

A small test project for evaluating device orientation sensors (`deviceorientation`, angles) as a tilt-based input scheme — built while prototyping control options for **Nebula Clone**, a browser-based multiplayer game in the Agar.io/Nebulous.io style.

The core question this project answers: *can a phone's tilt angle reliably and predictably move an on-screen object, across both portrait and landscape orientation, with no drift or inversion?* The demos here are that reference implementation — not a game, but a control scheme meant to be lifted into one.

## Why tilt controls, and why this instead of touch

Nebula Clone's primary control scheme is mouse/touch-follow, same as the games it's based on. Tilt is being evaluated as an alternative or supplementary input for mobile play, where a touch-drag control competes for screen space with the player's own view of the map. A tilt scheme frees up the whole screen for viewing, at the cost of needing to solve the problems this repo exists to solve: sensor reliability, orientation-awareness, and a control mapping that doesn't feel inverted or laggy.

## What's in this repo

| File | Purpose |
|---|---|
| `index.html` | Landing page — links to both demos below |
| `devicemotion-circle.html` | Tilt-controlled circle demo, orientation-aware (portrait + both landscape rotations) |
| `devicemotion-circle2.html` | Same demo (currently a duplicate of the above, kept as a separate entry point for future divergence — e.g. testing a different smoothing/sensitivity profile side by side) |

Each demo page maps live device orientation angles (`alpha`/`beta`/`gamma`) directly to an absolute on-screen position, rather than treating tilt as an accelerating force. That was a deliberate choice: a given phone orientation always reproduces the same circle position, with no velocity drift to correct for — which matters for a game control scheme, where "tilt back to center = character stops," full stop, is the behavior players expect.

## How to test it

Open either demo on a **phone**, over **https://** (see constraints below), and tap "Enable Orientation Control." On iOS this triggers a permission prompt; on Android it should start immediately. Tilt the device — the circle follows. Each page includes a live HUD showing raw angles, computed screen-relative tilt, detected screen orientation, and circle position, useful for debugging exactly what the sensor is reporting versus what the page computed from it.

## Known constraints

Documented here because they weren't obvious going in, and are worth remembering before wiring this into the actual game:

- **Requires a real `https://` (or `localhost`) origin.** Orientation/motion sensors are blocked entirely outside a secure context. This includes `file://` URLs and, less obviously, Android's `content://` URIs (e.g. opening a file straight from the Downloads app) — neither has a browser-recognized origin at all, so the sensor API is unavailable regardless of what the page's JavaScript does. GitHub Pages satisfies this by default.
- **iOS 13+ requires an explicit permission prompt**, triggered synchronously inside a user tap (`DeviceOrientationEvent.requestPermission()`). It cannot be requested on page load — there must be a real tap in between, or the browser silently rejects it.
- **A listener can attach with no error and still never receive events.** This happens on unsupported/blocked configurations with no visible failure — the permission call resolves fine, but the sensor simply never fires. Worth building a watchdog/timeout around any production version of this, rather than assuming "no error" means "working."
- **`beta`/`gamma` swap roles across orientation, and the swap direction isn't fixed.** These angles are defined in the device's fixed portrait-native frame, not the frame the player currently sees. In landscape, tilting the visible screen left/right is reported through `beta` in one landscape rotation and `gamma` in the other — and which physical rotation Android/iOS calls `90°` vs `270°` (`screen.orientation.angle`) isn't consistent across devices. Any control mapping needs to branch on the live orientation angle and be verified per-device rather than assumed from one test phone.

## Status

Working demo, verified against real device orientation readings across portrait and landscape. Not yet integrated into Nebula Clone — this repo is the isolated proof-of-concept before that wiring happens.
