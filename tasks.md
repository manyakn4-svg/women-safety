# Implementation Plan: Women Safety Alert App

## Overview

Build a single `index.html` file with all CSS and JavaScript inlined. Tasks are ordered so each step produces a runnable file — start with the shell, add features one by one, wire everything together at the end.

## Tasks

- [x] 1. Create the HTML shell with tab navigation and global state
  - Create `index.html` with `<!DOCTYPE html>`, `<head>` (viewport meta, title), and `<body>`.
  - Add a `<style>` block with base CSS: CSS variables for colors (primary, danger, warning, safe), dark background, font stack, reset.
  - Add a `<nav>` tab bar with 12 tab buttons (icons + labels) and a `<main>` with 12 `<section id="panel-*">` elements (empty for now).
  - Add a `<footer>` status bar with `#status-online` and `#status-location` spans.
  - Add a `<script>` block with the `AppState` object, `loadState()` / `saveState()` functions using `localStorage`, and a `switchTab(id)` function that shows the active panel and hides the rest.
  - Wire tab button clicks to `switchTab`.
  - _Requirements: 9.5_

- [x] 2. Implement Guardian Circle (Feature 7)
  - [x] 2.1 Build the Guardian Circle UI and CRUD logic
    - Render an "Add Contact" form (name + phone inputs + Add button) inside `#panel-guardian`.
    - Implement `addContact(name, phone)`: enforce max-5 cap, push to `AppState.contacts`, call `saveState()`, re-render list.
    - Implement `renderContacts()`: for each contact render a card with name, number, Call (`tel:`), SMS (`sms:`), Edit, and Delete buttons.
    - Implement `deleteContact(id)` and `editContact(id)` (inline edit via prompt or inline form).
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

  - [ ]* 2.2 Write property test for guardian contact cap
    - **Property 2: Guardian contact cap**
    - **Validates: Requirements 7.1**
    - Call `addContact` in a loop > 5 times and assert `AppState.contacts.length <= 5`.

- [~] 3. Implement Shadow Tracker (Feature 2)
  - [x] 3.1 Build location tracking and display
    - In `#panel-tracker`, add a coordinates display `<div id="coords">` and a "Share Location" button.
    - On app init, call `navigator.geolocation.watchPosition(onPosition, onGeoError)`.
    - `onPosition(pos)`: store `{ lat, lng }` in memory, update `#coords` and `#status-location` footer span.
    - `onGeoError()`: display a user-friendly permission-denied message in `#panel-tracker`.
    - "Share Location" button: build a `https://maps.google.com/?q=lat,lng` link and open `sms:` to the first guardian contact with the link.
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [~] 4. Implement Panic Pulse (Feature 1)
  - [~] 4.1 Build the panic button with hold-to-confirm countdown
    - In `#panel-panic`, add a large circular `<button id="btn-panic">` and a `<canvas id="panic-canvas">` overlay for the countdown ring.
    - On `pointerdown`: start a `setInterval` that increments a counter every 100 ms and redraws the arc on the canvas (0 → 100% over 3 s).
    - On `pointerup` / `pointerleave`: clear the interval, reset the canvas.
    - At 3 s (counter reaches 30): call `triggerSOS()` and clear the interval.
    - `triggerSOS()`: iterate `AppState.contacts`, open `sms:?body=<encoded message with coords>` for each; if `!navigator.onLine`, push payload to `localStorage.pendingSOS` instead.
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

  - [ ]* 4.2 Write property test for panic hold-to-confirm safety
    - **Property 1: Panic hold-to-confirm safety**
    - **Validates: Requirements 1.4**
    - Simulate `pointerdown` then `pointerup` at t < 3 s and assert `triggerSOS` was NOT called.

- [~] 5. Implement Siren Mode (Feature 6)
  - [~] 5.1 Build the siren toggle with audio and strobe
    - In `#panel-siren`, add a large toggle button `#btn-siren`.
    - `startSiren()`: create `AudioContext`, `OscillatorNode` (type `sawtooth`), `GainNode`; start oscillator; use `setInterval` to sweep frequency between 800 Hz and 1200 Hz every 500 ms; store refs in module-level vars.
    - `stopSiren()`: disconnect and close `AudioContext`; clear interval.
    - Strobe: separate `setInterval` toggling `document.body.style.background` between `#fff` and `#000` at 100 ms while siren is active; cleared on stop.
    - Button click toggles `sirenActive` flag and calls `startSiren` / `stopSiren`.
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [~] 6. Implement Echo Guard (Feature 3)
  - [~] 6.1 Build the fake call scheduler and overlay
    - In `#panel-echo`, add inputs for fake caller name and number, a delay `<select>` (0 s, 30 s, 60 s, 120 s), and a "Schedule Call" button.
    - "Schedule Call": read form values into `AppState.settings`, call `saveState()`, then `setTimeout(showFakeCall, delay)`.
    - `showFakeCall()`: render a full-screen overlay `#fake-call-overlay` with caller name, number, Accept and Decline buttons; start ringtone via Web Audio API (looping sine wave at 440 Hz).
    - Accept / Decline buttons: hide overlay, stop ringtone.
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5_

- [~] 7. Implement Vision Shield (Feature 4)
  - [~] 7.1 Build camera capture and photo gallery
    - In `#panel-vision`, add a `<video id="camera-feed" autoplay playsinline>`, a "Capture" button, and a `<div id="photo-gallery">`.
    - On panel activation: call `navigator.mediaDevices.getUserMedia({ video: true })` and set stream as `video.srcObject`.
    - "Capture" button: draw current video frame to a hidden `<canvas>`, call `canvas.toDataURL('image/jpeg', 0.7)`, push result to `AppState.photos`, call `saveState()`, call `renderGallery()`.
    - `renderGallery()`: for each base64 string in `AppState.photos`, render an `<img>` thumbnail and a "Download" button that creates a temporary `<a download="photo.jpg" href=dataURL>` and clicks it.
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

  - [ ]* 7.2 Write property test for photo storage integrity
    - **Property 7: Photo storage integrity**
    - **Validates: Requirements 4.3**
    - After a simulated capture (mock canvas `toDataURL`), assert the stored string starts with `data:image/jpeg;base64,` and that setting it as `img.src` fires `load` not `error`.

- [~] 8. Implement AI Threat Sense (Feature 5)
  - [~] 8.1 Build keyword-based threat analyzer
    - In `#panel-threat`, add a `<textarea id="situation-input">`, an "Analyze" button, and a `<div id="threat-result">`.
    - Define `HIGH_KEYWORDS`, `MEDIUM_KEYWORDS`, `LOW_KEYWORDS` arrays in the script.
    - `analyzeThreat(text)`: lowercase + tokenize; count matches per tier; return `{ level, score, suggestions }` where level is determined by highest-tier match (high > medium > low > none).
    - "Analyze" button: call `analyzeThreat`, render colored badge (red/yellow/green) and suggestion list in `#threat-result`.
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [ ]* 8.2 Write property test for threat level monotonicity
    - **Property 4: Threat level monotonicity**
    - **Validates: Requirements 5.2**
    - For a base text, assert that appending a high-severity keyword never decreases `analyzeThreat(text).score`.

- [~] 9. Implement Safe Path Navigator (Feature 8)
  - [~] 9.1 Build destination input and navigation links
    - In `#panel-safepath`, add a destination `<input id="destination">`, a "Navigate" button, a "Share Route" button, and a static safety tips `<ul>`.
    - "Navigate": encode destination, open `https://www.google.com/maps/dir/?api=1&travelmode=walking&destination=<encoded>` in a new tab.
    - "Share Route": open `sms:?body=<encoded message with destination + current coords>` to first guardian contact.
    - Render 5–6 static safety tips (e.g., "Stay on well-lit streets", "Share your ETA with a contact").
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_

- [~] 10. Implement Incident Vault (Feature 11)
  - [~] 10.1 Build incident logging, list, and export
    - In `#panel-vault`, add a `<textarea id="incident-desc">`, a "Use current location" `<input type="checkbox">`, a "Log Incident" button, an "Export JSON" button, and a `<div id="incident-list">`.
    - "Log Incident": create `{ id: Date.now(), timestamp: new Date().toISOString(), description, location? }`, unshift into `AppState.incidents`, call `saveState()`, call `renderIncidents()`.
    - `renderIncidents()`: render newest-first; each entry shows timestamp, description, location (if any), and a "Delete" button calling `deleteIncident(id)`.
    - "Export JSON": create a `Blob` from `JSON.stringify(AppState.incidents)`, create a temporary `<a download="incidents.json">` and click it.
    - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5_

  - [ ]* 10.2 Write property test for incident log ordering
    - **Property 5: Incident log ordering**
    - **Validates: Requirements 11.3**
    - Log two incidents A then B; assert `AppState.incidents[0].id === B.id` (newest first).

- [~] 11. Implement Stealth Mode (Feature 10)
  - [~] 11.1 Build the calculator overlay and secret unlock
    - Add `<div id="stealth-overlay">` as the first child of `<body>`, styled to cover the full viewport with a calculator appearance (dark theme, digit grid).
    - Implement a simple calculator: digit/operator buttons update a display `<span>`; `=` evaluates the expression using `safeEval(expr)`.
    - `safeEval(expr)`: validate `expr` against `/^[0-9+\-*/().]+$/`; if valid call `Function('return ' + expr)()`; else show "Error".
    - Secret unlock: if the current expression equals `AppState.settings.secretCode` (default `"1234"`) when `=` is pressed, hide `#stealth-overlay` and show `#app`.
    - Triple-tap re-entry: attach a tap counter on `<footer>`; three taps within 1 s show `#stealth-overlay` and hide `#app`.
    - On first load, show `#stealth-overlay` by default (hide `#app`).
    - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

  - [ ]* 11.2 Write property test for calculator expression safety
    - **Property 8: Calculator expression safety**
    - **Validates: Requirements 10.4**
    - Pass strings like `"alert(1)"`, `"1;alert(1)"`, `"__proto__"` to `safeEval` and assert they return `"Error"` without executing side effects.

  - [ ]* 11.3 Write property test for stealth visibility isolation
    - **Property 3: Stealth code isolation**
    - **Validates: Requirements 10.1, 10.4**
    - Assert that when `#stealth-overlay` is visible, `#app` has `display === 'none'`, and vice versa.

- [~] 12. Checkpoint — core features complete
  - Verify the file opens in a browser without console errors.
  - Confirm tab switching works for all 12 panels.
  - Confirm guardian contacts persist across page reload.
  - Confirm stealth mode hides/shows the app correctly.
  - Ensure all tests pass, ask the user if questions arise.

- [~] 13. Implement Offline SOS Beacon (Feature 9)
  - [~] 13.1 Register inline Service Worker and handle offline queuing
    - At the bottom of the `<script>` block, define the SW code as a template literal string (cache-on-install strategy caching `'./'`).
    - Register the SW: `const blob = new Blob([swCode], { type: 'application/javascript' }); navigator.serviceWorker.register(URL.createObjectURL(blob))`.
    - Add `window.addEventListener('online', onOnline)` and `'offline'` listeners; `onOnline` updates the footer indicator and calls `flushPendingSOS()`.
    - `flushPendingSOS()`: read `localStorage.pendingSOS`, open `sms:` links for each queued payload, clear the key.
    - Update `triggerSOS()` to branch on `navigator.onLine`.
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_

  - [ ]* 13.2 Write property test for offline SOS queuing
    - **Property 6: Offline SOS queuing**
    - **Validates: Requirements 9.4**
    - Mock `navigator.onLine = false`, call `triggerSOS()`, assert payload exists in `localStorage.pendingSOS` and no `sms:` navigation occurred.

- [~] 14. Implement Community Shield (Feature 12)
  - [~] 14.1 Build unsafe zone map with Leaflet
    - Add Leaflet CSS and JS CDN links in `<head>` (with `crossorigin` attribute).
    - In `#panel-community`, add a `<div id="map">` (height: 300px), a description `<input id="zone-desc">`, and a "Mark Unsafe Zone" button.
    - `initMap()`: initialize Leaflet map centered on current coords (or default 0,0); called lazily when the Community Shield tab is first activated.
    - `renderUnsafeZones()`: clear existing markers; for each entry in `AppState.unsafeZones`, add a Leaflet marker with a popup containing the description and a "Remove" button.
    - "Mark Unsafe Zone": create `{ id: Date.now(), lat, lng, description }`, push to `AppState.unsafeZones`, call `saveState()`, call `renderUnsafeZones()`.
    - "Remove" in popup: call `removeUnsafeZone(id)` → filter array → `saveState()` → `renderUnsafeZones()`.
    - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5_

- [~] 15. Final wiring and polish
  - [~] 15.1 Wire all features to shared AppState and triggerSOS
    - Ensure `triggerSOS()` is called from Panic Pulse (task 4) and is reachable from AI Threat Sense suggestions.
    - Ensure Shadow Tracker coordinates are used by Panic Pulse SOS message, Safe Path Navigator share, and Incident Vault location checkbox.
    - Ensure Guardian Circle contacts are used by Shadow Tracker share, Safe Path Navigator share, and Panic Pulse SOS.
    - _Requirements: 1.3, 2.3, 7.3, 7.4, 8.5_

  - [~] 15.2 Apply consistent UI styling across all panels
    - Add CSS for: tab active state, panel cards, button variants (primary, danger, ghost), contact cards, incident entries, threat badge colors, siren active state, fake call overlay, stealth calculator grid.
    - Ensure the app is usable on a 375px-wide mobile viewport (flexbox/grid layouts, touch-friendly tap targets ≥ 44px).
    - _Requirements: all_

- [~] 16. Final checkpoint — full integration
  - Open `index.html` in a browser and verify all 12 features are functional end-to-end.
  - Confirm `localStorage` persistence for contacts, incidents, unsafe zones, photos, and settings.
  - Confirm offline indicator appears when network is disabled in DevTools.
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP.
- All features are self-contained within a single `index.html` — no build step required.
- Leaflet (Community Shield) is the only external CDN dependency; all other features use native browser APIs.
- Property tests can be implemented as simple inline `console.assert` calls or with a micro test runner in a separate `<script type="module">` block.
