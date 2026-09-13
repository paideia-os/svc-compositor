# svc-compositor v1.0.0 -- release note + mirror-push workflow (M5-001)

**Repo:** github.com/paideia-os/svc-compositor
**Wave:** R102 tool (design/graphics/r102-user-plan.md §4.4)
**Version at first release:** 1.0.0
**Upstream policy:** `design/tooling/plan.md` §6.3 (paideia-os) --
package repository layout; `design/02-development-environment.md`
§1140 + §1164 (paideia-os) -- hybrid Ed25519+ML-DSA-65 signing,
release-line key custody; `design/graphics/r102-user-plan.md` §4.4
(paideia-os) -- M5-001 scope (issue #16 in this repo).

This document is both the **release note** for what v1.0.0 ships and
the operator runbook for cutting the signed release and pushing it to
the paideia-os package mirror at
`https://pkgs.paideia-os/main/svc-compositor/1.0.0/`. The workflow
mirrors mkfs.pdxfs's `release/RELEASE-1.0.0.md` byte-for-byte in
structure -- see that file for the worked template this one follows.

The actual `git tag v1.0.0` + `git push origin v1.0.0` is a **manual
step main performs separately from this milestone** -- see §5.

---

## 1. What v1.0.0 ships

Everything landed at M4 (honest-witness boot smokes) plus the M5-001
release closer. See `CHANGELOG.md` for the itemised list,
`STATUS.md` for the per-issue checklist.

- **Four honest-witness boot smokes** at `tests/boot_r102_*.pdx` --
  each publishes exactly one deterministic byte fingerprint to fd 1
  (the boot serial `paideia-os/tools/run-smoke.sh` greps) and
  returns 0 on a live-wire write.
- **`src/tool_ident.pdx`** -- `ToolIdent` module publishing
  `PDX_TOOL_NAME = "svc-compositor\0"` and
  `PDX_TOOL_VERSION = "1.0.0\0"` per this org's Wave F UND-extern
  discipline.
- **`release/manifest.pdxsig.txt`** -- source-form dual-sign manifest.

## 2. What v1.0.0 does NOT ship

- **No compositor pipeline code.** The M1..M3 slice (caps.decl,
  window table, 60 Hz render loop, `KIND_FB_SCANOUT` cap, query
  surface, broker accept loop) remains open per this repo's own
  STATUS.md. This release is an honest bookkeeping closer for M4/M5
  so a later M1..M3 landing can retarget the four witnesses in
  place (real assertion body added BEFORE the `sys_write`,
  fingerprint string unchanged) without churning the release-line
  git tag or the mirror layout.
- **No live BLAKE3 golden for `boot_r102_screenshot`.** Deferred to
  the M2/M3 retargeting; convention documented in that driver's
  module header.
- **No live dual-signature.** See §3.

## 3. Why the signature block is placeholder at v1.0.0

Consistent with every other M5 closer in this org (see mkfs.pdxfs and
libpdx-volume's own RELEASE-1.0.0.md §3), the actual dual-sign pass
is a release-time step performed with live key material outside this
repo. `release/manifest.pdxsig.txt`'s `[signatures]` block carries
`SIGNATURE_PLACEHOLDER_PENDING_LIVE_SIGN` in every slot; the release
tool fills them at mirror-push time via
`paideia-pq-sign::sign_release_artifact(sk, manifest_path)` called
once with the paideia-release-line Ed25519 secret key and once with
the ML-DSA-65 secret key. Verification is AND-semantics: both
signatures MUST verify; failure of either rejects the package.

## 4. Mirror layout (`pkgs.paideia-os`)

```
https://pkgs.paideia-os/main/svc-compositor/1.0.0/
├── manifest.pdxsig                        (binary, dual-signed)
├── manifest.pdxsig.txt                    (source form, this file's sibling)
├── src/tool_ident.pdx
├── tests/README.md
├── tests/boot_r102_first_pixel.pdx
├── tests/boot_r102_input_route.pdx
├── tests/boot_r102_screenshot.pdx
├── tests/boot_r102_window_present.pdx
├── STATUS.md
├── CHANGELOG.md
├── README.md
└── LICENSE
```

No `bin/` entry -- v1.0.0 ships no compiled service ELF (contrast
mkfs.pdxfs, which does).

## 5. Cutting the tag (manual step, main-only)

```bash
# From a fresh checkout at the exact commit that publishes this file:
git tag -a v1.0.0 -m "svc-compositor v1.0.0 -- R102.M4 honest-witness closer + M5-001 signed release"
git push origin v1.0.0
```

After the tag lands on `origin`, the release tool runs the mirror-
push workflow described in §3. This document ships in that mirror
push so a future auditor sees the same runbook alongside the
artifacts it describes.

## 6. Post-release checklist

- [ ] `git tag v1.0.0` pushed to `origin` (manual step, main).
- [ ] Release tool computed every `<BLAKE3-*>` placeholder in
      `release/manifest.pdxsig.txt` against the v1.0.0 tag tree.
- [ ] Release tool dual-signed the manifest with the paideia-release-
      line keys (Ed25519 + ML-DSA-65).
- [ ] Mirror push to `https://pkgs.paideia-os/main/svc-compositor/1.0.0/`
      succeeded.
- [ ] `paideia-os` submodule bumped in the parent monorepo (if this
      repo is tracked as a submodule).
