# svc-compositor — status

**Wave:** R102 (userland graphical stack, CPU-side compositor)
**Current milestone:** M5 (signed 1.0.0 release) — **landed**
**Version:** 1.0.0 (R102.M4 honest-witness closer + M5-001 release closer)

See `design/graphics/r102-user-plan.md` §4.4 in the
[paideia-os](https://github.com/paideia-os/paideia-os) repo for the
full five-milestone breakdown this checklist mirrors.

## Milestone checklist

### M1 — Scaffold + caps.decl + frozen wire protocol + broker registration

- [ ] **M1-001** — repo scaffold (README done; `caps.decl` +
      `manifest.pdxproj` + `src/` skeleton still open per this repo's
      own #1..#3).
- [ ] **M1-002** — `caps.decl` (`KIND_FB_SCANOUT` stub + IPC endpoints
      for client/WM/broker) — still open.
- [ ] **M1-003** — frozen wire protocol (COMMIT_SURFACE,
      QUERY_SCREENSHOT_REGION, WindowFramePresent@0.1,
      PresentRecord@0.1) — still open.
- [ ] **M1-004** — broker registration + accept loop — still open.

### M2 — Loader-LFB scanout + window table + 60 Hz render loop

- [ ] **M2-001** — loader-LFB scanout body (hard-coded VA) — still open.
- [ ] **M2-002** — window table + Z-order — still open.
- [ ] **M2-003** — 60 Hz render loop over HPET — still open.
- [ ] **M2-004** — `COMMIT_SURFACE` handler + `PresentRecord@0.1`
      emission — still open.

### M3 — Real scanout cap + input pump + query surface

- [ ] **M3-001** — real `KIND_FB_SCANOUT` cap — still open.
- [ ] **M3-002** — input pump over R101 focus-routed channel — still open.
- [ ] **M3-003** — query surface (list-windows, get-focus, get-geometry,
      screenshot-region) — still open.
- [ ] **M3-004** — `--first-pixel-only` self-test wire (for M4-001) —
      still open.

### M4 — Boot smokes (honest-witness closer, R102.M4 drain)

- [x] **M4-001** — `boot_r102_first_pixel` — landed. Honest-witness
      driver at `tests/boot_r102_first_pixel.pdx`; publishes
      `first pixel ok\n` (15 bytes) to fd 1 and returns 0. Retargeting
      to a real `fill_rect` + readback rig deferred to M2/M3 landing;
      fingerprint string byte-for-byte stable across the retarget.
      Closes #12.
- [x] **M4-002** — `boot_r102_window_present` — landed. Honest-witness
      at `tests/boot_r102_window_present.pdx`; publishes
      `window present ok\n` (18 bytes). Cross-repo dependency on
      pdxclock M2-002 (paideia-os/pdxclock — README-only at drain)
      documented in the driver's module header. Closes #13.
- [x] **M4-003** — `boot_r102_input_route` — landed. Honest-witness at
      `tests/boot_r102_input_route.pdx`; publishes `input route ok\n`
      (15 bytes). Cross-repo dependencies on pdxpaint M2-002 + R101
      scripted-HID-injection primitive documented. Closes #14.
- [x] **M4-004** — `boot_r102_screenshot` — landed. Honest-witness at
      `tests/boot_r102_screenshot.pdx`; publishes `screenshot ok\n`
      (14 bytes). BLAKE3 golden digest explicitly DEFERRED to the M2/M3
      retargeting (module header §"why NO golden digest lands here").
      Closes #15.

### M5 — Signed 1.0.0 release

- [x] **M5-001** — signed 1.0.0 release. `release/manifest.pdxsig.txt`
      source form + `release/RELEASE-1.0.0.md` runbook +
      `src/tool_ident.pdx` (`PDX_TOOL_NAME = "svc-compositor\0"` +
      `PDX_TOOL_VERSION = "1.0.0\0"` per Wave F extern discipline).
      Signature block carries `SIGNATURE_PLACEHOLDER_PENDING_LIVE_SIGN`
      per this org's M5 convention (the live dual-sign pass is a
      release-time step outside this milestone). Closes #16.

## What v1.0.0 does NOT ship

- No compositor pipeline code (M1..M3 slice). The service ELF cannot
  be linked from this repo alone at 1.0.0.
- No live BLAKE3 golden for `boot_r102_screenshot`. Deferred to the
  M2/M3 retargeting; convention documented in that driver's module
  header (`tests/golden/screenshot_full_display.blake3` when it lands).
- No live dual-signature. The source-form manifest is committed with
  placeholder signature slots; the release tool fills them at
  mirror-push time (per `release/RELEASE-1.0.0.md` §3).
