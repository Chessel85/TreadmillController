# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

Phases 0–2 are complete (BLE connect flow, manual-mode session logging, local session storage) and Phase 3's core Strava flow is built and confirmed on-device. GitHub Pages is live at https://chessel85.github.io/TreadmillController/, serving the real single-file app (`index.html`, no build step). There are no build/lint/test commands — this is plain HTML/CSS/JS with no build step by design; do not invent tooling.

Strava's API now requires a paid subscription with no free tier (confirmed 2026-08-08, see `IMPLEMENTATION_PLAN.md`'s Session Log and the `project_strava_paywall_pivot` memory) — there is **no OAuth/API integration** in this app. Instead, a finished session is exported as a `.tcx` file and offered via a Share/download button for the user to upload manually on Strava's own free "Upload Activity" page.

Phase 4 (planned mode — driving the treadmill from a plan file) is next, using the user's draft format at `rp1.txt` as a starting point, followed by a UI/UX tidy-up pass (see `IMPLEMENTATION_PLAN.md` task 4.4).

**`IMPLEMENTATION_PLAN.md` is the authoritative, session-to-session task tracker for building this project.** At the start of any work session, read its Status section to see the current phase and next task, do that task, then check it off and update Status/Session Log before finishing — it's designed to be picked up cold across many separate sessions.

Note: `.gitignore` currently excludes `*.txt` (a placeholder so the file isn't empty, not a deliberate exclusion). Since plan files are user-uploaded text files (not committed to the repo) this is unlikely to matter, but if sample/test plan `.txt` files are ever added to the repo itself, the `.gitignore` will need updating so they're tracked.

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
- **Plan file format**: user has drafted a proposed format at `rp1.txt` (repo root) — `incline:X% Speed:Ykmh Duration:Zmin` per line, plus `StartLoop,N`/`EndLoop` repeat blocks — use it as the starting point for Phase 4.1's parser rather than designing from scratch.
