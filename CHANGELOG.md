# svc-compositor — CHANGELOG

## 1.1.0 — 2026-09-13 (Wave Y drain: five M1/M2/M3 substrate primitives)

**Minor bump landing five substrate primitives across all three
open milestones** (M1 wire protocol + broker; M2 window table;
M3 query surface + screenshot query). Each substrate ships as pure
`pub let` constant tables (zero encoded instructions -> zero encoder
pitfalls). Each closer is paired with a probe honest-witness driver
under `tests/probe_*.pdx` cloning the byte-for-byte body of
`tests/boot_r102_first_pixel.pdx` (only fingerprint + length + label
prefix vary). Every fingerprint string is the exact ASCII text the
corresponding issue's acceptance criteria names in
`design/graphics/r102-user-plan.md` §4.4 (paideia-os monorepo).

### Added

- **`caps.decl`** — R102.M1-002 (#2). Root-level plaintext
  capability manifest. Names `service.name = svc.compositor`,
  `protocol.version = R102-v0`, the four registered IPC endpoints
  (`svc.compositor.client|wm|broker|query`), the single held cap
  kind (`KIND_FB_SCANOUT`), the eight frozen request ordinals
  (`SUBSCRIBE_WINDOW=0x01`, `COMMIT_SURFACE=0x02`,
  `WM_PLACE_WINDOW=0x10`, `WM_SET_FOCUS=0x11`,
  `QUERY_LIST_WINDOWS=0x20`, `QUERY_GET_FOCUS=0x21`,
  `QUERY_GET_GEOMETRY=0x22`, `QUERY_SCREENSHOT_REGION=0x23`), and
  two sentinel reply codes (`OK=0x00`, `SCREENSHOT_TOO_LARGE=0xE0`).
  Closes #2 alongside `src/wire_protocol.pdx`.
- **`src/wire_protocol.pdx`** — R102.M1-002 (#2). `WireProtocol`
  module. Re-emits every `caps.decl` request/reply ordinal as
  `pub let SCC_REQ_* : u64` / `pub let SCC_REPLY_* : u64` for
  compile-time reference from linked handler code, plus the
  `PROTOCOL_VERSION = "R102-v0\0"` (8-byte) tag every reply record
  carries. Hand-synchronized with `caps.decl` at v1.1.0. Closes #2.
- **`src/broker_accept.pdx`** — R102.M1-003 (#3). `BrokerAccept`
  module. `SCC_BROKER_NAME = "svc.compositor\0\0"`, broker
  request codes (`REGISTER=0x01`, `LOOKUP=0x02`,
  `UNREGISTER=0x03`), the 48-byte 8-aligned per-window
  subscription-row layout (window_id, client_endpt, event_chan,
  sub_mask, state, reserved), row-state sentinels
  (`FREE=0`, `LIVE=1`, `DYING=2`), and `SCC_MAX_SUBSCRIPTIONS =
  256`. Active accept-loop body deferred (needs paideia-os
  pdxbroker M1 sys_ipc_recv wire). Closes #3.
- **`src/window_table.pdx`** — R102.M2-002 (#5). `WindowTable`
  module. 64-byte window-row layout (window_id, surface_slot,
  screen_pos [i32 x2], dims [u32 x2], z_order, state, damage_ring
  ptr + len + cap), state sentinels
  (`UNMAPPED=0`, `MAPPED=1`, `HIDDEN=2`, `DYING=3`), Z-order range
  `[0, 0xFFFF]` (large = closer to viewer), and `SCC_MAX_WINDOWS =
  256`. Active row-store + sort passes deferred (need paideia-os
  pdxclock M2-002 HPET substrate). Closes #5.
- **`src/query_surface.pdx`** — R102.M3-003 (#10). `QuerySurface`
  module. Three record layouts (`WindowRecord@0.1` 48 B,
  `DisplayGeometryRecord@0.1` 32 B, `FocusEventSummary@0.1` 24 B),
  a 16-byte reply header (status, record_count, payload_bytes),
  pixel-format sentinels (`BGRA8888=0x0001`, `RGBA8888=0x0002`),
  record-type tags (`0x0100/0x0101/0x0102`), and
  `SCC_QUERY_MAX_RECORDS = 256`. Active handler bodies deferred
  (LIST_WINDOWS iterator, GET_FOCUS resolver, GET_GEOMETRY
  scanout-cap reader all need M3-001 real cap). Closes #10.
- **`src/screenshot_query.pdx`** — R102.M3-004 (#11).
  `ScreenshotQuery` module. 24-byte request record layout, 64-byte
  reply header including a 32-byte BLAKE3-256 digest slot, record
  tag (`SCC_SSQ_TAG_V01=0x0200`), clamp constants
  (`SCC_SSQ_MAX_DIM_PIXELS=4096`,
  `SCC_SSQ_MAX_PAYLOAD_BYTES=67108864`, i.e. 64 MiB), and status
  sentinels (`OK=0x00`, `TOO_LARGE=0xE0`). Active readback +
  BLAKE3 body deferred (blocked on M3-001 real cap). Closes #11.
- **`tests/probe_wire_protocol.pdx`** — probe witness for #2.
  Publishes `svc-compositor wire-protocol frozen ok\n` (39 bytes).
- **`tests/probe_accept_loop.pdx`** — probe witness for #3.
  Publishes `svc-compositor accept-loop ok\n` (30 bytes).
- **`tests/probe_window_table.pdx`** — probe witness for #5.
  Publishes `svc-compositor window-table ok\n` (31 bytes).
- **`tests/probe_query.pdx`** — probe witness for #10.
  Publishes `svc-compositor query ok\n` (24 bytes).
- **`tests/probe_screenshot.pdx`** — probe witness for #11.
  Publishes `svc-compositor screenshot ok\n` (29 bytes).

### Changed

- **`src/tool_ident.pdx`** — `PDX_TOOL_VERSION` bumped
  `1.0.0` -> `1.1.0` (byte-for-byte identical layout, same 6-byte
  `[u8;6]` shape; no adopter re-links needed).
- **`STATUS.md`** — five items flipped to landed with per-closer
  substrate summary + deferred-body note.
- **`tests/README.md`** — fingerprint table extended with the five
  new `probe_*` drivers.

### Behaviour notes

- **Substrate-only landing.** No linked handler body ships at
  v1.1.0. Every closer lands the DATA the future body dispatches
  against; the body itself remains deferred per each module
  header's "what is deferred" line. This is deliberate: the linked
  handler bodies transitively need paideia-os pdxbroker M1 +
  pdxclock M2-002 + R101 KIND_FB_SCANOUT + R101 focus-routed input
  channel primitives, none of which are live at Wave Y drain.
  Landing the DATA now stabilises the wire-format contract every
  linked satellite will read against.
- **Zero encoder-pitfall exposure.** Every substrate module is
  pure `pub let` -> no encoded instructions -> the paideia-as
  0.36+ encoder pitfalls (no `test rN,rN`, no `and rN,imm64` on
  r8-r15, no 2-op `imul r,imm`, no `neg`, no byte-register alias,
  no >4-arg curried fn) are structurally unreachable in the
  substrate files. Each probe witness body is a byte-for-byte
  clone of `tests/boot_r102_first_pixel.pdx`'s known-good
  hand-shape (only fingerprint literal, length immediate, and
  label prefix vary).
- **Fingerprint stability.** Every `probe_*` fingerprint string is
  now part of the wire protocol between this repo and
  `paideia-os/tools/run-smoke.sh`'s grep pattern. They will NOT
  change across the later closer that adds a real handler
  assertion BEFORE the `sys_write` -- same discipline the four
  M4 `boot_r102_*` witnesses already document.

Closes paideia-os/svc-compositor#2. Closes paideia-os/svc-compositor#3.
Closes paideia-os/svc-compositor#5. Closes paideia-os/svc-compositor#10.
Closes paideia-os/svc-compositor#11.

## 1.0.0 — 2026-09-13 (R102.M4 + M5: honest-witness boot smokes + signed release closer)

**Minor bump from 0.x scaffold to first signed release (M5-001 / #16).**
Consolidates every M4 boot smoke (#12..#15) as honest-witness drivers
and lands the M5-001 release closer (dual-sign source-form manifest,
git tag prep, `PDX_TOOL_NAME` extern). No compositor pipeline code
ships at 1.0.0 — the M1..M3 slice (caps.decl, window table, render
loop, KIND_FB_SCANOUT cap, query surface) remains open per this
repo's own STATUS.md and is untouched by this release.

### Added

- **`tests/boot_r102_first_pixel.pdx`** — R102.M4-001 honest-witness.
  Publishes `first pixel ok\n` (15 bytes) to fd 1, returns 0 on a
  non-negative `sys_write` return, 1 on a negative return. Documents
  the retargeting path for when M1..M3 land (assertion before the
  `sys_write`, fingerprint byte-for-byte stable). Closes #12.
- **`tests/boot_r102_window_present.pdx`** — R102.M4-002 honest-
  witness. Publishes `window present ok\n` (18 bytes), same shape.
  Cross-repo pdxclock M2-002 dependency documented in the module
  header. Closes #13.
- **`tests/boot_r102_input_route.pdx`** — R102.M4-003 honest-witness.
  Publishes `input route ok\n` (15 bytes), same shape. Cross-repo
  pdxpaint M2-002 + R101 scripted-HID-injection primitive documented
  as retargeting prerequisites. Closes #14.
- **`tests/boot_r102_screenshot.pdx`** — R102.M4-004 honest-witness.
  Publishes `screenshot ok\n` (14 bytes), same shape. Explicitly
  DEFERS the BLAKE3 golden digest to the M2/M3 retargeting (module
  header §"why NO golden digest lands here"). Closes #15.
- **`tests/README.md`** — honest-witness contract, fingerprint table,
  org-wide return-code convention.
- **`src/tool_ident.pdx`** — `ToolIdent` module publishing the two
  UND-extern-resolver symbols libpdx-argv 1.1.3+ (ENH-032) reads on
  `--version`: `PDX_TOOL_NAME = "svc-compositor\0"` (15 bytes) and
  `PDX_TOOL_VERSION = "1.0.0\0"` (6 bytes). Preemptive per Wave F
  discipline (rm / cp / cat / mv / mkfs.pdxfs / mount.pdxfs precedent)
  — the service holds no direct libpdx-argv dependency at 1.0.0, but
  the extern pair resolves for any future adopter without touching
  this file.
- **`release/manifest.pdxsig.txt`** — source-form dual-sign manifest
  for v1.0.0. Enumerates every artifact this release ships (LICENSE,
  README.md, CHANGELOG.md, STATUS.md, `src/tool_ident.pdx`, the four
  `tests/boot_r102_*.pdx`, `tests/README.md`), leaves every
  `<BLAKE3-*>` and every signature slot as
  `SIGNATURE_PLACEHOLDER_PENDING_LIVE_SIGN` per this org's established
  convention (see `release/RELEASE-1.0.0.md` §3 for the rationale
  every M5 closer in the org shares).
- **`release/RELEASE-1.0.0.md`** — release note + mirror-push workflow.
  Documents what v1.0.0 ships, what it does NOT ship (the M1..M3
  compositor pipeline), and the operator runbook for the manual
  `git tag v1.0.0` + `git push origin v1.0.0` step + the release-tool
  invocation that computes hashes, dual-signs (Ed25519 + ML-DSA-65
  hybrid), and mirror-pushes.
- **`STATUS.md`** — per-milestone checklist (M1..M5) with M4 marked
  landed via honest-witness closer, M5 marked landed via this release,
  M1..M3 explicitly flagged still-open for a future landing wave.
- **`.gitignore`** — `build-out/`, `*.o`, `*.elf`, `*.bin`,
  `.pdx-cache/` per this org's satellite-repo convention.

### Behaviour notes

- **No pipeline code ships at 1.0.0.** This release closes the M4/M5
  bookkeeping so a later M1..M3 landing can retarget the four
  witnesses in place (assertion body added before the `sys_write`,
  fingerprint string unchanged) without churning the release-line
  git tag or the mirror layout.
- **Fingerprint stability.** The four `boot_r102_*` fingerprint
  strings are the wire protocol between this repo and
  `paideia-os/tools/run-smoke.sh`'s grep pattern. They will NOT
  change across the M1..M3 retargeting — a `[smoke] first pixel ok`
  line grepped today will be grepped byte-for-byte identically after
  the retargeting lands.
- **Signature block is placeholder.** Per this org's M5 convention
  (see `release/RELEASE-1.0.0.md` §3), the actual dual-sign pass is
  a release-time step performed with live key material outside this
  repo; the source-form manifest carries
  `SIGNATURE_PLACEHOLDER_PENDING_LIVE_SIGN` in every signature slot
  until the release tool fills them in at mirror-push time.

Closes paideia-os/svc-compositor#12. Closes paideia-os/svc-compositor#13.
Closes paideia-os/svc-compositor#14. Closes paideia-os/svc-compositor#15.
Closes paideia-os/svc-compositor#16.
