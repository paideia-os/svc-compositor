# svc-compositor

The compositor service. The one process that holds the framebuffer cap (`KIND_FB_SCANOUT`), owns the window table, drives the 60 Hz render loop, pumps input events to focused windows, and exposes the `svc.compositor` query surface (list-windows, get-focus, get-geometry, screenshot-region). Single-compositor by construction — no plugin surface. First `svc-*` prefix in the paideia-os org.

Part of the **paideia-os** organization. MIT-licensed.

## Wave

R102 (softarch userland graphical stack) — the CPU-side framebuffer stack
that lands the first graphical UI on paideia-os before the G-series
GPU-accelerated compositor matures. Companion to the osarch R101 kernel-side
plan.

## Design reference

- Design lives in the monorepo at [`design/graphics/r102-user-plan.md`](https://github.com/paideia-os/paideia-os/blob/main/design/graphics/r102-user-plan.md) §2.4 / §4.4.
- Kernel-side companion: `design/graphics/r101-kernel-plan.md`.

## Milestones

Per the plan, this repo lands across five milestones:

- **M1** — repo scaffold; caps.decl (KIND_FB_SCANOUT stub + IPC endpoints for client/WM/broker); frozen wire protocol; broker registration + accept loop
- **M2** — loader-LFB scanout body (hard-coded VA); window table + Z-order; 60 Hz render loop over HPET; COMMIT_SURFACE handler; PresentRecord@0.1 emission
- **M3** — real KIND_FB_SCANOUT cap; input pump over R101 focus-routed channel; query surface wired (list/focus/geometry/screenshot)
- **M4** — boot smokes: first_pixel, window_present, input_route, screenshot
- **M5** — signed 1.0.0 release

Every issue is filed against one of these five milestones; see the Issues tab.

## Scaffolding

No code lands with this repo scaffold — scaffolding lives in the M1
issues (`caps.decl`, `src/` skeleton, public API stubs, argv parsing).
Repo shape mirrors R100 satellites: paideia-as manifest at root,
`caps.decl` at root, `src/` module tree, `tests/`, `release/`,
`doc/<name>.pdxdoc`, dual-signed `manifest.pdxsig` at 1.0.0.

## License

MIT. See [LICENSE](LICENSE).
