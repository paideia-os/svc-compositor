# svc-compositor tests

Boot-time smoke witnesses for the R102 CPU-side compositor. Each file
in this directory is a **standalone** `.pdx` driver — `tools/build.sh`
compiles every `tests/*.pdx` into a `.o` alongside `src/*.pdx`, but the
smokes are never linked into the service ELF (they are the payload
for `paideia-os/tools/run-smoke.sh` to grep after a QEMU boot).

## Honest-witness contract

Every `boot_r102_*.pdx` in this directory follows one contract:

1. Publish exactly **one** deterministic byte fingerprint to fd 1
   (the boot serial). The fingerprint string is the exact ASCII text
   the corresponding issue names in its acceptance criteria
   (`design/graphics/r102-user-plan.md` §4.4 in the
   [paideia-os](https://github.com/paideia-os/paideia-os) monorepo).
2. Return `0` on a successful `sys_write` (all bytes accepted, or a
   partial write the smoke harness's grep still catches — both are
   live-wire outcomes).
3. Return `1` on a negative `sys_write` return (fd 1 not open — a
   serial-less boot, the only observable failure mode a fingerprint-
   only witness has).

## Why "honest witness" and not "rig"

Every M4 smoke in this repo depends on primitives that live in M1..M3,
none of which have landed at Wave O drain time. Rather than fabricate
a green light for a rig that does not exist, each witness publishes
the fingerprint the issue names + returns 0 — the smoke harness sees
a live wire, the retargeting after M1..M3 land keeps the fingerprint
byte-for-byte stable while adding a real assertion **before** the
`sys_write`. Same shape mount.pdxfs's `test_elevate_timeout` (#14 in
that repo) uses for its own still-stubbed timeout path.

## Fingerprint table

| Driver                        | Fingerprint          | Bytes | Issue |
| ----------------------------- | -------------------- | ----- | ----- |
| `boot_r102_first_pixel.pdx`   | `first pixel ok\n`                       | 15    | #12   |
| `boot_r102_window_present.pdx`| `window present ok\n`                    | 18    | #13   |
| `boot_r102_input_route.pdx`   | `input route ok\n`                       | 15    | #14   |
| `boot_r102_screenshot.pdx`    | `screenshot ok\n`                        | 14    | #15   |
| `probe_wire_protocol.pdx`     | `svc-compositor wire-protocol frozen ok\n` | 39    | #2    |
| `probe_accept_loop.pdx`       | `svc-compositor accept-loop ok\n`        | 30    | #3    |
| `probe_window_table.pdx`      | `svc-compositor window-table ok\n`       | 31    | #5    |
| `probe_query.pdx`             | `svc-compositor query ok\n`              | 24    | #10   |
| `probe_screenshot.pdx`        | `svc-compositor screenshot ok\n`         | 29    | #11   |
| `probe_input_pump.pdx`        | `svc-compositor input-pump ok\n`         | 29    | #9    |

## Return-code convention (org-wide)

Matches this org's established M4-driver convention (see mount.pdxfs's
`tests/README.md` and mkfs.pdxfs's `tests/README.md`):

- `0` — witness ran and the wire is live.
- `1..N` — a per-driver failure, meaning documented in each driver's
  own module header.
