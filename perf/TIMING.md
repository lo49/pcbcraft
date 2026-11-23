# Matched pan + wire-drag timing (PR #478, review item 3)

All numbers below are **CPU replay** (`run_ui` + CPU tessellation of the real editor), not GPU presentation. See Limitations.

## Runner
- `schematic_pan_drag` — one added public diagnostic entry point in `crates/app/src/performance.rs` and a thin CLI example `crates/app/examples/schematic_pan_drag.rs`. The identical 210-line measurement-only patch was appended to both worktrees; no production measurement path was changed.
- Workload per iteration: 8 warm-up frames; 10 middle-button drag frames that pan the schematic camera; click to select a wire; 12 frames that reshape-drag that wire (a commit on release); one idle frame. Fresh app/session per iteration. Logical time advances at 60 Hz without sleeping.
- `appearance_setup` is measured once per run, outside the replay samples.

## Fixture
- `bus64-schematic.pc`, SHA-256 `c4bca6d6f154ce5239d0ffea226d1b9cebabba6973eee75c81e009f9911f86e1`
- Two sheets, 64 wires on a shared global `BUS` net (32 per sheet), no symbols. Derived from the shared `multisheet-styled.pc` generator output with the saved appearance stripped and the project format set to 48, a format both revisions decode. The same bytes were used for all three cases.
- PR-styled case applies the shipped `nets.set_appearance` command (color `[58,136,238]`, width `500` = 5 pt) to the `BUS` anchor before replay. That is the only input difference for case (c).

## Environment
- server2: Intel i5-14500T (20 logical CPUs), 31 GiB RAM; Debian 13 (trixie), kernel 6.12.107+deb13-amd64, governor `powersave`, no display.
- rustc/cargo 1.95.0; `--locked --release -p pcbcraft-app --features performance-harness`; native CPU replay, no window/GPU.

## Table (round 2, 30 iterations; p95 is nearest-rank over 30 per-iteration sums)

| Case | Revision | Fixture bytes | Pan UI p50 / p95 (ms) | Drag UI p50 / p95 (ms) | Pan+drag UI p50 / p95 (ms) | Pan+drag tessellate p50 / p95 (ms) |
|---|---|---|---|---|---|---|
| (a) main default | `09031e47` (full: `09031e47af9cab28df3dc204afc8d42ac4311f9a`) | base | 16.13 / 16.58 | 28.55 / 28.96 | **44.73 / 45.39** | 28.98 / 29.87 |
| (b) PR default | `673ac706` (full: `673ac706baaea858b2e6febce9dc6e68ae6fca11`) | base | 16.48 / 16.72 | 29.22 / 29.68 | **45.71 / 46.39** | 29.08 / 29.78 |
| (c) PR styled | `673ac706` | base + in-session override | 17.49 / 17.71 | 33.26 / 33.75 | **50.75 / 51.37** | 29.15 / 30.11 |

Deltas, round 2: (b)-(a) total UI p50 **+0.98 ms** (p95 +1.01 ms); (c)-(b) total UI p50 **+5.04 ms** (p95 +4.98 ms). The override adds ~4.05 ms on drag frames and ~1.01 ms on pan frames.

Round 1 (same commands, immediately before, 30 iterations) gave (a) 44.90/46.35, (b) 46.60/61.71, (c) 51.74/55.63 ms total UI p50/p95. Its PR-default p95 contained one 43 ms idle-frame outlier; round 2 is the cleaner matched set. The two rounds bracket the same p50 story: PR default within ~1–2 ms of main, styled about +5 ms on PR.

Binary SHA-256: main `74ecdeb416d35d5444a1417c1d13e5b2649470dcdef66fe6079e1f1702089e3c`; PR `5d4f42810781f0b1809894d8fccd606eb25dee213532d342f5ed9624aef0efa1`.

## Commands
```sh
ssh sotchi@192.168.31.215
cd /home/sotchi/pcbcraft
git worktree add --detach /home/sotchi/pcbcraft-bench-main 09031e47af9cab28df3dc204afc8d42ac4311f9a
git worktree add --detach /home/sotchi/pcbcraft-bench-pr   673ac706baaea858b2e6febce9dc6e68ae6fca11

# In each worktree: append perf_patch.rs to crates/app/src/performance.rs and add
# schematic_pan_drag.rs as crates/app/examples/schematic_pan_drag.rs (identical sources).
source ~/.cargo/env
cd /home/sotchi/pcbcraft-bench-main
cargo build --locked --release -p pcbcraft-app --features performance-harness --example schematic_pan_drag
cd /home/sotchi/pcbcraft-bench-pr
cargo build --locked --release -p pcbcraft-app --features performance-harness --example schematic_pan_drag

cd /home/sotchi/pcbcraft-review-evidence
/home/sotchi/pcbcraft-bench-main/target/release/examples/schematic_pan_drag bus64-schematic.pc env-main-default.json 30 default final2-main-default.json
/home/sotchi/pcbcraft-bench-pr/target/release/examples/schematic_pan_drag   bus64-schematic.pc env-pr-default.json   30 default final2-pr-default.json
/home/sotchi/pcbcraft-bench-pr/target/release/examples/schematic_pan_drag   bus64-schematic.pc env-pr-styled.json    30 styled  final2-pr-styled.json
python3 summarize.py final2-main-default.json final2-pr-default.json final2-pr-styled.json
```

## Limitations
- CPU service cost only: real egui input, editor layout, shape generation and CPU tessellation. No eframe window, GPU submission, compositor or physical input latency. Server2 has no display, so `board_perf_window` cannot run; no browser rendered replay was attempted (no Chrome and no wasm-bindgen/wasm-opt CLI on server2).
- Single shared server with 20 cores, `powersave` governor, no CPU affinity or idle-machine guarantee. One-iteration-per-revision interleaving was not possible with separate binaries; round 2 is a back-to-back repeat of round 1, and the two rounds are reported rather than cherry-picked.
- p50/p95 over 30 samples are directional, not a tail-latency baseline. The styled case's command setup (~1.2 ms, once) is outside the per-frame numbers and is recorded in the raw report.
- The fixture is a synthetic two-sheet, 64-wire project with no symbols and no board; it isolates the schematic wire paint path, not the full application.
- The measurement-only patch is additive and identical on both revisions, but the two revisions' surrounding code differs by the whole PR; absolute cross-revision deltas therefore include unrelated main changes between `09031e47` and `673ac706` (the latter merged upstream/main into the PR branch).
- The styled project hash differs between rounds because `nets.set_appearance` allocates a fresh net identity UUID; the base fixture bytes and the replay input sequence are identical in all cases. Original `/home/sotchi/pcbcraft` and `/home/bouilo01/pcbcraft-timing` were not modified.
