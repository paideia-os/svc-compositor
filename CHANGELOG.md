# svc-compositor — CHANGELOG

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
