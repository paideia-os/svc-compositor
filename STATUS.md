# svc-compositor — status

**Wave:** R102 (userland graphical stack, CPU-side compositor)
**Current milestone:** M5 (signed 1.0.0 release) — **landed** + Wave Y drain of
five M1/M2/M3 substrate primitives (#2, #3, #5, #10, #11).
**Version:** 1.1.0 (Wave Y drain — five M1/M2/M3 primitive substrates
land as pure `pub let` constant tables + honest-witness probes;
active handler bodies remain deferred per each module header).

See `design/graphics/r102-user-plan.md` §4.4 in the
[paideia-os](https://github.com/paideia-os/paideia-os) repo for the
full five-milestone breakdown this checklist mirrors.

## Milestone checklist

### M1 — Scaffold + caps.decl + frozen wire protocol + broker registration

- [ ] **M1-001** — repo scaffold (README done; `caps.decl` +
      `manifest.pdxproj` + `src/` skeleton still open per this repo's
      own #1..#3).
- [x] **M1-002** — `caps.decl` (`KIND_FB_SCANOUT` stub + IPC endpoints
      for client/WM/broker) + frozen wire protocol landed at v1.1.0
      as `caps.decl` (root) + `src/wire_protocol.pdx` (SCC_REQ_* /
      SCC_REPLY_* ordinals + PROTOCOL_VERSION = "R102-v0"). Probe
      witness at `tests/probe_wire_protocol.pdx` publishes
      `svc-compositor wire-protocol frozen ok\n` (39 bytes). Closes
      #2.
- [x] **M1-003** — broker registration + accept-loop substrate
      constants landed at v1.1.0 as `src/broker_accept.pdx`
      (SCC_BROKER_NAME, broker request codes, 48-B subscription row
      layout, row-state sentinels, SCC_MAX_SUBSCRIPTIONS = 256).
      Active accept-loop body deferred (needs paideia-os pdxbroker
      M1 sys_ipc_recv wire). Probe witness at
      `tests/probe_accept_loop.pdx` publishes
      `svc-compositor accept-loop ok\n` (30 bytes). Closes #3.

### M2 — Loader-LFB scanout + window table + 60 Hz render loop

- [ ] **M2-001** — loader-LFB scanout body (hard-coded VA) — still open.
- [x] **M2-002** — window table + Z-order substrate constants landed
      at v1.1.0 as `src/window_table.pdx` (64-B window row layout,
      state sentinels, Z-order range [0, 0xFFFF],
      SCC_MAX_WINDOWS = 256). Active row-store + sort passes
      deferred (need paideia-os pdxclock M2-002 HPET substrate).
      Probe witness at `tests/probe_window_table.pdx` publishes
      `svc-compositor window-table ok\n` (31 bytes). Closes #5.
- [ ] **M2-003** — 60 Hz render loop over HPET — still open.
- [ ] **M2-004** — `COMMIT_SURFACE` handler + `PresentRecord@0.1`
      emission — still open.

### M3 — Real scanout cap + input pump + query surface

- [ ] **M3-001** — real `KIND_FB_SCANOUT` cap — still open.
- [ ] **M3-002** — input pump over R101 focus-routed channel — still open.
- [x] **M3-003** — query surface record types + reply-header layout
      landed at v1.1.0 as `src/query_surface.pdx`
      (WindowRecord@0.1, DisplayGeometryRecord@0.1,
      FocusEventSummary@0.1, reply-header, SCC_QUERY_MAX_RECORDS =
      256). Active handler bodies deferred (LIST_WINDOWS iterator,
      GET_FOCUS resolver, GET_GEOMETRY scanout-cap reader all need
      M3-001 real cap live). Probe witness at
      `tests/probe_query.pdx` publishes `svc-compositor query ok\n`
      (24 bytes). Closes #10.
- [x] **M3-004** — `QUERY_SCREENSHOT_REGION` record layout + clamp
      constants + `SCREENSHOT_TOO_LARGE` sentinel landed at v1.1.0
      as `src/screenshot_query.pdx` (24-B request, 64-B header
      w/ BLAKE3-256 digest slot, SCC_SSQ_MAX_DIM_PIXELS = 4096,
      SCC_SSQ_MAX_PAYLOAD_BYTES = 64 MiB). Active readback +
      BLAKE3 body deferred (blocked on M3-001 real cap). Probe
      witness at `tests/probe_screenshot.pdx` publishes
      `svc-compositor screenshot ok\n` (29 bytes). Closes #11.

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
