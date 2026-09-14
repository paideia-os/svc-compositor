# svc-compositor — status

**Wave:** R102 (userland graphical stack, CPU-side compositor)
**Current milestone:** M5 (signed 1.0.0 release) — **landed** + Wave Y drain of
five M1/M2/M3 substrate primitives (#2, #3, #5, #10, #11) + Wave PP
cohort landing five active M1/M2/M3 closers (#1, #4, #6, #7, #8) + Wave
III's M3-002 input pump (#9) + Wave OOO's 5 integration probes
(SVC-CO-01..05).
**Version:** 1.4.0 (Wave OOO — 5 integration probes deepen coverage of
already-landed M2/M3 closers: a real pixel-roundtrip blit, a
real-cadence render-loop timing assertion, a second commit-message
decode at a non-adjacent damage-table slot, a stricter WEAK-stub-exact
fb-scanout-query check, and the tree's first real
`input_pump_deliver_event` exercise. No `src/` changes — test-only
landing on top of Wave III's v1.3.0).

See `design/graphics/r102-user-plan.md` §4.4 in the
[paideia-os](https://github.com/paideia-os/paideia-os) repo for the
full five-milestone breakdown this checklist mirrors.

## Milestone checklist

### M1 — Scaffold + caps.decl + frozen wire protocol + broker registration

- [x] **M1-001** — repo scaffold landed at v1.2.0 (Wave PP / #1).
      README.md, LICENSE (MIT), CHANGELOG.md, tools/build.sh, and
      `release/manifest.pdxsig.txt` (source-form, from the M5-001
      closer) were already in place; this closer adds the cap-kind
      enumeration `caps.decl` was missing (`cap.uses` lines for
      KIND_USER, KIND_IPC_ENDPOINT, KIND_SURFACE, KIND_INPUT_EVENT
      alongside the existing `cap.holds = KIND_FB_SCANOUT`). Closes #1.
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

- [x] **M2-001** — loader-LFB scanout body landed at v1.2.0 (Wave PP
      / #4) as `src/scanout.pdx` (`Module Scanout`): WEAK-stub
      `scanout_init()` (fixed 1920x1080 @ pitch 7680, sentinel base
      pointer 0xFFFFF00000000000 — no `sys_bootinfo_get_lfb` or real
      KIND_FB_SCANOUT cap exists yet, §7.1.1) + `scanout_blit_rect`
      (clipped row-copy into the scanout buffer via `rep_movsb`).
      Real-body probe witness at `tests/probe_scanout.pdx` publishes
      `svc-compositor loader-lfb ok\n` (29 bytes) after exercising
      both functions. Closes #4.
- [x] **M2-002** — window table + Z-order substrate constants landed
      at v1.1.0 as `src/window_table.pdx` (64-B window row layout,
      state sentinels, Z-order range [0, 0xFFFF],
      SCC_MAX_WINDOWS = 256). Active row-store + sort passes
      deferred (need paideia-os pdxclock M2-002 HPET substrate).
      Probe witness at `tests/probe_window_table.pdx` publishes
      `svc-compositor window-table ok\n` (31 bytes). Closes #5.
- [x] **M2-003** — 60 Hz render loop over HPET landed at v1.2.0 (Wave
      PP / #6) as `src/render_loop.pdx` (`Module RenderLoop`):
      `sys_clock_read_ns` (sysno 66) busy-wait to the 16_666_666 ns
      frame period, `compositor_render_frame` draining Commit's
      per-surface damage table into `scanout_blit_rect` calls, and a
      64-byte PresentRecord@0.1-shaped emission (`render_loop_present`).
      `render_loop_run(n)` gives a bounded driver for probe/witness
      use. Real-body probe witness at `tests/probe_render_loop.pdx`
      publishes `svc-compositor render-loop ok\n` (30 bytes). Closes #6.
- [x] **M2-004** — `COMMIT_SURFACE` handler landed at v1.2.0 (Wave PP
      / #7) as `src/commit.pdx` (`Module Commit`): parses the 24-byte
      COMMIT_SURFACE message (gated on the FROZEN 0x02 ordinal, not
      the dispatch brief's 0x11 typo — see file header) into a new
      256-slot per-surface damage table `render_loop.pdx` drains every
      tick. `PresentRecord@0.1` emission itself lives in M2-003 (#6)
      per the design doc's own split. Real-body probe witness at
      `tests/probe_commit.pdx` round-trips a full message through the
      handler and asserts the stored fields before publishing
      `svc-compositor commit-surface ok\n` (33 bytes). Closes #7.

### M3 — Real scanout cap + input pump + query surface

- [x] **M3-001** — real `KIND_FB_SCANOUT` cap query landed at v1.2.0
      (Wave PP / #8) as `src/fb_scanout_cap.pdx`
      (`Module FbScanoutCap`): `kind_fb_scanout_query` issues the real
      `sys_cap_invoke` syscall (sysno 4) for the scanout base pointer,
      WEAK-stub fallback when the slot does not resolve (the only
      possible outcome today — the kind itself is still unminted per
      §7.2.1). pitch/dims are WEAK-stubbed unconditionally — cap_invoke's
      B5-004 MVP returns one u64, not a structured geometry record.
      Full per-kind geometry dispatch remains blocked on osarch minting
      KIND_FB_SCANOUT for real; `src/scanout.pdx`'s hard-coded VA (#4)
      stays the compositor's actual scanout target until that lands.
      Real-body probe witness at `tests/probe_fb_scanout_cap.pdx`
      publishes `svc-compositor kind-fb-scanout ok\n` (34 bytes).
      Closes #8.
- [x] **M3-002** — input pump over R101 focus-routed channel landed at
      v1.3.0 (Wave III / #9) as `src/input_pump.pdx` (`Module
      InputPump`): resolves `svc.input` via `sys_svc_lookup` (sysno
      43), stamps each polled InputEventRecord@0.1's `window_id` from
      a local FocusCache mirror (real libpdx-event FocusCache not yet
      linkable — file header note 1), and forwards to that window's
      registered client endpoint via `sys_ipc_send` (sysno 42). The
      per-window client-endpoint table is this closer's own
      self-contained mirror, mirroring commit.pdx's (#7) precedent
      of not depending on window_table.pdx's still-unallocated row-
      store — a future closer that lands the real accept-loop
      row-store should reconcile the two. `input_pump_stamp_and_route`
      is split out from the syscall wrapper specifically so it can be
      probed without a live broker. Real-body probe witness at
      `tests/probe_input_pump.pdx` round-trips a full stamp+lookup
      cycle (success, no-focus, no-client, and bad-window paths) and
      asserts each outcome before publishing `svc-compositor
      input-pump ok\n` (29 bytes). Closes #9.
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
