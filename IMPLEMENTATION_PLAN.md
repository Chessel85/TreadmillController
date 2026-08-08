# Implementation Plan: Treadmill Controller

## How to use this file (read this first, every session)
1. Read the **Status** section below to see the current phase and next task.
2. Do **only** the next unchecked task — keep sessions small and focused so they fit comfortably in context.
3. Check the task off, update **Status**, and add a short dated note under **Session Log** if anything was learned, decided, or changed from the plan.
4. Commit the code changes and this file together.
5. If a task turns out to be bigger than expected, split it into sub-tasks in place rather than doing a huge session.

## Status
- **Current phase:** Phase 0 — De-risking spikes
- **Next task:** 0.1 — BLE capability investigation
- **Last updated:** 2026-08-08

## Reference
Requirements: `Product Definition Document.md`. Full feasibility/risk analysis behind this plan's ordering is summarized in the Session Log below (2026-08-08 entry) rather than duplicated here.

---

## Phase 0 — De-risking spikes
Goal: answer the unknowns that could reshape scope, before investing in build-out. No production code required for these.

- [ ] **0.1 BLE capability investigation.** Using a generic BLE scanner app (e.g. nRF Connect or LightBlue) on the iPhone, scan the JTX Sprint 6 and record: does it advertise BLE at all, what services/characteristics exist, is FTMS (`0x1826`) present, and — critically — is anything writable (not just readable)? Write findings to `docs/ble-findings.md`. **This determines whether Phase 4 (planned-mode auto-control) is buildable as scoped, or needs to be redefined as a read-only/cueing feature.**
- [ ] **0.2 Bluefy + VoiceOver spot check.** Confirm Bluefy's device-picker and permission dialogs are usable with VoiceOver on the target iPhone. Note any friction.
- [ ] **0.3 Repo/deploy scaffolding.** Decide tech approach (plan: plain HTML/CSS/JS, no build step, to keep GitHub Pages deploy trivial — revisit only if a real need for a framework emerges). Enable GitHub Pages on the repo. Deploy a minimal static "hello world" page and confirm it loads on the iPhone via Bluefy.

## Phase 1 — Accessible shell & Bluetooth connect flow
Goal: the skeleton every other feature sits inside.

- [ ] **1.1 Page shell & mode selection.** Semantic, VoiceOver-friendly landing screen: connect-to-treadmill entry point, then choice of Manual vs Planned mode (locked once a session starts, per PDD Ref #2).
- [ ] **1.2 Treadmill connect flow.** `navigator.bluetooth.requestDevice` flow with an accessible "scanning / found / connected / failed" state sequence (PDD Ref #1).

## Phase 2 — Manual mode logging (primary objective — build this before Strava/planned mode)
Goal: deliver the PDD's stated primary objective end-to-end, independent of whether Phase 0.1 turns out favorably.

- [ ] **2.1 Session state machine.** Start / pause / resume / finish for manual sessions (PDD Ref #3, #6, #7). No treadmill control involved.
- [ ] **2.2 Telemetry capture.** If Phase 0.1 found readable BLE characteristics, subscribe and log speed/incline changes with timestamps as they occur (PDD Ref #5). If not readable, fall back to duration-only logging.
- [ ] **2.3 Local session storage.** Persist the in-progress and completed session log (e.g. `localStorage`/IndexedDB) so a dropped connection or accidental reload doesn't lose data.

## Phase 3 — Strava integration
Goal: get a finished manual session onto Strava — this alone satisfies the PDD's primary objective.

- [ ] **3.1 Strava app registration + OAuth connect.** Register a Strava API app; implement the authorize redirect and code exchange (client_secret embedded client-side, per the accepted-risk decision — repo stays public, URL isn't publicized).
- [ ] **3.2 Upload finished session.** Convert a completed manual session into an indoor-run activity upload (time/distance; no GPS needed) via Strava's upload endpoint.
- [ ] **3.3 Re-auth/token refresh.** Minimal-friction reconnect for subsequent sessions (PDD Ref #9's "minimal authentication").

## Phase 4 — Planned mode (contingent on Phase 0.1 findings)
Goal: PDD Ref #4 and #8. **Before starting, revisit Phase 0.1's findings and choose a direction:**
- If writable BLE control exists → build full auto-control as scoped.
- If only readable/no BLE control exists → redefine this phase as a "plan cueing" feature instead (app announces upcoming speed/incline changes via VoiceOver, user adjusts manually) and update this plan + the PDD to reflect the reduced scope.

- [ ] **4.1 Plan file format + parser.** Define and document a simple text format (duration, speed, incline per line); implement parsing and validation.
- [ ] **4.2 Plan upload/selection UI.** Upload a plan file from the user's PC; list previously used plans (PDD Ref #4).
- [ ] **4.3 Drive or cue the session** per the direction chosen above.

## Phase 5 — Reliability & polish
- [ ] **5.1 Screen wake-lock + user messaging.** Request a wake lock during active sessions; clearly communicate (VoiceOver-friendly) that the phone must stay unlocked and Bluefy foregrounded for the session to keep logging (screen-lock kills the BLE session — confirmed during feasibility research).
- [ ] **5.2 Connection-loss recovery.** Handle a dropped BLE connection or accidental page reload mid-session without losing logged data.
- [ ] **5.3 Full VoiceOver pass.** Walk every screen end-to-end with VoiceOver on-device.
- [ ] **5.4 Real-session sanity check with Spotify.** Run a real session with Spotify streaming to Bluetooth headphones throughout, confirming no interference (expected to just work per feasibility research — this is a verification task, not a build task).

## Session Log
- **2026-08-08**: Feasibility review completed prior to this plan. Key findings carried forward: (a) JTX Sprint 6 BLE writable-control is unverified — JTX's own connectivity guide doesn't list the Sprint 6 at all, only Sprint-7/8 Pro/9 — hence Phase 0.1 gates Phase 4; (b) iOS drops/throttles BLE on screen-lock or backgrounding — hence Phase 5.1; (c) Strava's client_secret can't be fully hidden in a static site — user accepted this risk (public repo, unpublicized URL, rotate secret if leaked) — hence Phase 3.1's approach; (d) Bluefy is the only Web Bluetooth path on iOS (Safari doesn't support it) and is a single third-party dependency; (e) simultaneous Spotify + treadmill BLE is technically sound (independent Bluetooth roles) — added to PDD as Ref #10.
