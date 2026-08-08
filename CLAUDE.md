# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

Phases 0–3 are complete (BLE connect flow, manual-mode session logging, local session storage, TCX export/Strava share flow) and Phase 4 (planned mode) is in progress: 4.1 (plan file parser) and 4.2 (plan library) are done and confirmed on-device. GitHub Pages is live at https://chessel85.github.io/TreadmillController/, serving the real single-file app (`index.html`, no build step). There are no build/lint/test commands — this is plain HTML/CSS/JS with no build step by design; do not invent tooling.

Strava's API now requires a paid subscription with no free tier (confirmed 2026-08-08, see `IMPLEMENTATION_PLAN.md`'s Session Log and the `project_strava_paywall_pivot` memory) — there is **no OAuth/API integration** in this app. Instead, a finished session is exported as a `.tcx` file and offered via a Share/download button for the user to upload manually on Strava's own free "Upload Activity" page.

**Phase 4.3 (actually driving the treadmill) is the current blocker**, and it's genuinely unresolved, not just unstarted: the JTX Sprint 6's write-control channel (`0xFFF2`, proprietary "iConsole" protocol — see `docs/ble-findings.md`) has no publicly documented command format, and nothing found via research is a confirmed match for this exact console's firmware (see the `project_iconsole_protocol_unconfirmed` memory for full detail — **read this before touching Phase 4.3**). A separate diagnostic page, `protocol-tester.html` (linked from `index.html` just under the heading), exists specifically to discover the real command bytes — it can enumerate live BLE services/characteristics, send arbitrary hex commands, run an automatic sweep of candidate commands, and export everything to a shareable `.txt` log. **Do not assume a working protocol exists — check `IMPLEMENTATION_PLAN.md`'s Phase 4.3 status and the `project_iconsole_protocol_unconfirmed` memory first**, since this is exactly the kind of thing a fresh session could wrongly assume was solved. `protocol-tester.html` has its own version-naming scheme (Iliad/Odyssey names) separate from `index.html`'s mythical creatures — see the `feedback_protocol_tester_versioning` memory.

Once real command bytes are confirmed, 4.3b (the actual auto-driving sequencer) and then a UI/UX tidy-up pass (task 4.4) follow.

**`IMPLEMENTATION_PLAN.md` is the authoritative, session-to-session task tracker for building this project.** At the start of any work session, read its Status section to see the current phase and next task, do that task, then check it off and update Status/Session Log before finishing — it's designed to be picked up cold across many separate sessions.

Note: `.gitignore` was emptied by the user (previously excluded `*.txt` as a placeholder) so real plan files can be tracked — `rp1.txt` is now a committed sample. As of 2026-08-08 this `.gitignore` change and a further `rp1.txt` edit are sitting **uncommitted** in the working tree, since the user said they'd `git add`/commit/push those themselves rather than have Claude do it — check `git status` at the start of the next session in case they're still sitting there.

## What this project is

Treadmill Controller is a personal (single-user) web page, hosted from this GitHub repo (also serving as the web host), used to control a JTX Sprint 6 treadmill via Bluetooth and to log/upload completed runs to Strava.

Full requirements live in `Product Definition Document.md` — read it before implementing features. Key points to keep in mind:

- **Primary user is blind and uses VoiceOver on iOS.** Accessibility (screen-reader-friendly markup, clear focus order, minimal/simple controls) is not optional polish — it is the core design constraint for any UI work.
- **Target environment**: iPhone 13, up-to-date iOS, using the Bluefy browser (a browser that supports Web Bluetooth on iOS, unlike Safari). Any Bluetooth connectivity code must work within Bluefy's capabilities.
- **Two session modes**:
  - *Manual mode*: the runner controls the treadmill directly; the app only logs activity (speed/incline/duration over time) for later Strava upload.
  - *Planned mode*: the user uploads a plain text file defining a sequence of duration/speed/incline changes, and the app drives the treadmill according to that plan (the treadmill can still be adjusted manually mid-session). There are deliberately no manual speed/incline controls in the app UI itself.
- **Session lifecycle**: connect to treadmill → choose mode → explicit Start action begins logging → pause/resume supported (pausing only affects app-side logging, never the treadmill) → finish (manually, early termination of a plan, or natural plan completion) never changes treadmill speed/incline itself.
- **Strava integration**: implemented as a `.tcx` file export + Share/download button, not OAuth (Strava's API now requires a paid subscription — see Project status above). Strava mainly needs time/distance, but a more detailed breakdown (incline/speed changes over time) should be available to the user even if not all of it goes to Strava.
- **Plan file format**: finalized and implemented (see `parsePlanFile()` in `index.html`, and `rp1.txt` at the repo root for the reference format/example) — `incline:X% Speed:Ykmh Duration:Z` per line (incline 0–15 integer, speed 0.0–20.0 one decimal place, duration as minutes and/or seconds up to 180 min/59 sec), plus `StartLoop:N`/`EndLoop` repeat blocks (nesting supported), full-line `#` comments.
