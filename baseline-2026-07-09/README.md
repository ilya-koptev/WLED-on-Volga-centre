# WLED baseline — Volga centre (captured 2026-07-09)

Raw configuration snapshot of the existing WLED installation, read live over the
network. This is the **starting point** for building a new WLED project/config.

## Device

| | |
|---|---|
| Name | `WLED-Gledopto` |
| Hardware | Gledopto Elite (ESP32) |
| Firmware | WLED 0.15.1-beta2 (`Gledopto_Elite_2D-EXMU`) |
| MAC | `20:e7:c8:61:27:14` |
| IP | 192.168.69.100 (static), LAN 192.168.69.0/24, gw .1 |
| LEDs | 600× WS2812 RGB (type 22, GRB) on GPIO16, single output |
| Power limit (ABL) | 850 mA @ 55 mA/LED — very low for 600 LEDs |
| FPS | 42 |
| Button / Relay / IR | GPIO17 / GPIO18 / disabled |
| Usermod | AudioReactive **enabled** (I2S mic pins 32/15/14, gain 60, AGC on) |
| MQTT / Alexa / Hue / NTP | all off |
| Sync | receive on, send off; realtime E1.31/sACN/DDP on (port 5568) |
| OTA lock | **off** (anyone on LAN can reflash) |

## Files (raw, exactly as returned by the device)

| File | Endpoint | What it is |
|---|---|---|
| `cfg.json` | `/cfg.json` | Full device config (network, hardware, LED, sync, usermods). Passwords are masked by WLED (length only). Restorable via Config → Security → Restore. |
| `presets.json` | `/presets.json` | All presets (only preset **1 "Test1"** is defined; slots 2–31 empty). Restorable the same way. |
| `state.json` | `/json/state` | Live state at capture: on, bri 128, active preset 1, 5 segments. |
| `info.json` | `/json/info` | Runtime info (fw, LED count, heap, usermod status). |
| `effects.json` | `/json/eff` | Effect name list (187 effects in this build). |
| `palettes.json` | `/json/pal` | Palette name list (71 palettes). |
| `fxdata.json` | `/json/fxda` | Per-effect metadata (slider/color/option semantics). |
| `nodes.json` | `/json/nodes` | Discovered sync nodes (empty — none). |
| `full.json` | `/json` | Combined state+info+effects+palettes (superset dump). |

> `ledmap.json` / `ledmap0.json` return **404** — no custom 2D pixel map exists on
> the device (`info.maps = [{id:0}]` is the default identity map; the strip is
> treated as 1D). Not included.
>
> `wsec.json` (Wi-Fi / AP / OTA passwords in cleartext) was **deliberately not
> captured** — no secrets are committed here.

## Segment layout (preset 1 "Test1")

WLED `stop` is **exclusive** → a segment covers pixels `[start … stop-1]`, `len = stop-start`.

| Seg | Name | start | stop | Pixels | Effect | Colors |
|---|---|---|---|---|---|---|
| 0 | Big circle | 0 | 213 | 0–212 | fx 11 Scan Dual | blue-violet + red |
| 3 | Connect | 214 | 333 | 214–332 | fx 0 Solid | amber |
| 2 | Branch 1 | 334 | 366 | 334–365 | fx 0 Solid | amber |
| 1 | Small circle | 367 | 576 | 367–575 | fx 10 Scan | amber |
| 4 | Branch 2 | 577 | 599 | 577–598 | fx 0 Solid | red |

## Notes for the new project

- **Suspected off-by-one:** each segment starts +1 after the previous one's `stop`,
  leaving pixels **213, 333, 366, 576, 599** unassigned (they stay dark). If the strip
  is physically continuous (no gaps), shift each `start` to the previous `stop`
  (0–213, 213–334, 334–366, 366–576, 576–600) so all 600 LEDs are used. **Not yet
  confirmed live.**
- **AudioReactive is enabled but unused visually** — none of the current effects
  (Solid / Scan / Scan Dual) are sound-reactive. Music currently changes nothing.
- **Palettes unused** — everything runs on manual per-segment colors (`pal 0`).
- Two-level brightness: segments at 255 scaled by global 128 → effective ~50%.
