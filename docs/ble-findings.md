# BLE Capability Findings — JTX Sprint 6

**Date:** 2026-08-08
**Method:** Scanned and connected to the treadmill using the LightBlue app (iPhone), standing next to the powered-on JTX Sprint 6. Screenshots in `docs/ble-screenshots/`.
**Caveat:** The only JTX manual available at the time was for the Sprint-8, not the Sprint-6. This turned out not to matter for BLE purposes — see below.

## Device identity

- Advertised name: **`iConsole+0168`**
- UUID (this specific unit): `7486F867-4708-AF29-19ED-09A830F9D536`

"iConsole" is a third-party console/control-board product used across many budget treadmill brands (not JTX-specific), which explains why it matched the Sprint-8 manual's Zwift/Kinomap pairing instructions (both of which referred to pairing with an "iConsole" device) even though that manual is for a different model.

## Services & characteristics found

### Service `0xFFF0` — proprietary iConsole protocol
| Characteristic | Properties | Purpose |
|---|---|---|
| `0xFFF1` | Notify | Telemetry stream, proprietary binary format (undocumented — would need reverse-engineering to decode) |
| `0xFFF2` | Write, Write without Response | **Control channel.** This is the one write-capable characteristic usable for treadmill control. Command format is the proprietary "iConsole" protocol — not officially documented by JTX, but this exact `0xFFF0/0xFFF1/0xFFF2` pattern is a known, widely-used one on budget/rebadged treadmills and has been reverse-engineered before by hobbyist open-source treadmill-control projects. |

### Service `0x1826` "Fitness Machine" — standard Bluetooth SIG FTMS
| Characteristic | Properties | Meaning |
|---|---|---|
| `0x2ACC` | Read | Fitness Machine Feature (capability flags) |
| `0x2AD4` | Read | Supported Speed Range |
| `0x2AD5` | Read | Supported Incline Range |
| `0x2AD6` | Read | Supported Resistance Level Range |
| `0x2AD7` | Read | Supported Heart Rate Range |
| `0x2AD8` | Read | Supported Power Range |
| `0x2ACD` | **Notify** | **Treadmill Data** — live speed/incline/distance/time in the officially documented FTMS binary format |

**No Fitness Machine Control Point (`0x2AD9`) or Fitness Machine Status (`0x2ADA`) characteristic was found.** We scrolled to the end of the full characteristic list (see `jtx-4-...png`) and confirmed the FTMS service is **read/notify only** — it exposes standard, well-documented live telemetry, but not standard writable control.

### Device Information Service (`0x180A`, standard)
Hardware Revision `20.00.20.00`, Firmware Revision `04`, Serial Number `00.D2` — informational only, no relevance to control/telemetry.

### Custom service `0x02F00000-0000-0000-0000-000000FFFE00`
| Characteristic | Properties | Purpose |
|---|---|---|
| `...FF03` | Read | Unknown |
| `OTA Response` | Notify, Read | Firmware OTA (over-the-air update) response channel |
| `...FF00` | Read | Unknown |
| `...FF01` | Write, Write without Response | Firmware OTA write channel |

This is a firmware-update (DFU/OTA) service, identifiable by the "OTA Response" characteristic name. **Not relevant to runtime control — do not write to this for treadmill control purposes; it's for flashing firmware and could be destructive if misused.**

## Answers to the Phase 0.1 questions

- **Does it advertise BLE at all?** Yes — `iConsole+0168`.
- **What services/characteristics exist?** See above: proprietary `0xFFF0` (iConsole), standard `0x1826` (FTMS, read/notify only), standard `0x180A` (Device Info), and a custom OTA/DFU service.
- **Is FTMS (`0x1826`) present?** Yes, but **read/notify only** — no standard Control Point. Good for telemetry, not usable alone for control.
- **Is anything writable?** Yes — `0xFFF2` (proprietary iConsole protocol). This is the only viable control channel; the OTA write characteristic is out of scope/unsafe to use for control.

## Implications for the plan

- **Phase 2 (telemetry logging):** Use the standard FTMS `0x2ACD` Treadmill Data notify characteristic. It's officially documented by the Bluetooth SIG, so decoding it is straightforward — no reverse-engineering needed. This is the preferred telemetry source over the proprietary `0xFFF1`.
- **Phase 4 (planned-mode auto-control):** Writable control exists (`0xFFF2`), so **full auto-control is buildable as originally scoped** — but only via the undocumented proprietary iConsole protocol, not standard FTMS. Before starting Phase 4, we'll need to determine the actual command byte format for `0xFFF2` (start/stop/set-speed/set-incline). Plan to research existing open-source iConsole/FTMS-bridge reverse-engineering projects for a known command table rather than reverse-engineering from scratch by trial and error against the real machine.
- Per the Phase 4 decision rule in the plan ("if writable BLE control exists → build full auto-control as scoped"), **this condition is met.** No redefinition to a read-only/cueing feature is needed.
