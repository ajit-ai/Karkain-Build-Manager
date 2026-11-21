# Karkain Build Manager — First Implementation Backlog

## First vertical slice

**Slice goal.** Take a two-package Karkain workspace, hash the inputs with `crypto` sha256, run a build step for a changed package, cache its artifact, and emit a JSON manifest that records the target and its hash — a real incremental-build workflow, no GUI and no compiler modification.

**Karkain modules/files expected.** `src/main.kark` (CLI entry), `src/graph.kark` (parse `workspace graph`), `src/hash.kark` (`crypto` sha256 + hex via `io_read_all`), `src/cache.kark` (`io_write_file`/`io_read_lines`), `src/model.kark`, `karkain.toml`.

**Exact input.** A workspace with `pkgA/main.kark` and `pkgB/main.kark`, where `pkgB` imports `pkgA`, and `pkgA/main.kark` has been modified. Command: `karkain-build build workspace.kark`.

**Exact expected observable output.** Stdout reports `built pkgA (cache miss), pkgB (cache hit)` on the first run; a second identical run reports `pkgA (cache hit), pkgB (cache hit)` (no rebuild). A JSON manifest file `build-manifest.json` is written containing an entry for each target with its sha256 hash, for example `{"workspace":"workspace.kark","targets":[{"id":"pkgA","hash":"<sha256>"},{"id":"pkgB","hash":"<sha256>"}]}`. No GUI window.

**Verification command pattern (Karkain v1.2.0, Go front-end engine, C-free native Windows target).**

```
$env:KARKAIN_ENGINE="go"
karkain build src/main.kark --target native-x86_64-windows -o karkain-build
.\karkain-build build workspace.kark
.\karkain-build build workspace.kark
```

Acceptance: second run performs zero rebuilds (cache hits) and the manifest hashes match `crypto` sha256 of the inputs.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
|---|---|---|---|---|---|---|---|---|
| BM-001 | Workspace graph parsing | `graph.kark` | `workspace graph` PRESENT | Parses targets/edges, detects cycles | Unit | M | `workspace` unexecuted | Graph fixtures pass |
| BM-002 | Content hashing | `hash.kark` | `crypto` PRESENT | sha256 matches known vectors | Unit | S | None | Hash test passes |
| BM-003 | Change detection | `detect.kark` | BM-001, BM-002 | Selects only dirty targets | Integration | M | `filesystem` PARTIAL (avoided) | Dirty-selection test passes |
| BM-004 | Cache & artifacts | `cache.kark` | BM-002; `io` PRESENT | Stores/retrieves by hash | Integration | M | OS atomic-write PARTIAL | Round-trip verified |
| BM-005 | Build CLI | `main.kark` | BM-003, BM-004 | `build` command emits manifest | E2E | M | None | Vertical slice passes |
| BM-006 | Cooperative scheduler | `sched.kark` | BM-001; `async` PRESENT | Runs independent steps via channels/pool | Integration | L | `async` cooperative (no threads) | Schedule test passes |
| BM-007 | Diagnostics artifact capture | `artifact.kark` | `check --format=json` PRESENT | Captures `E-K-*`/`W-K-*` codes | Integration | M | `json` PARTIAL | Capture test passes |
| BM-008 | Supply-chain verify | `verify.kark` | `pkg audit/verify`, `crypto` | Detects tampered dependency | Security | M | `pkg` unexecuted | Tamper test passes |
| BM-009 | Bench/prof reporting | `report.kark` | `bench`, `prof` PRESENT | Records measurements as artifact | Integration | M | Toolchain unexecuted | Report fixture passes |
| BM-010 | TUI dashboard/graph view | `tui.kark` | BM-005..007 | Keyboard graph/artifact inspection | E2E | L | GUI BLOCKED (TUI path allowed) | Manual TUI run passes |

## Dependency order and gates

1. BM-001 → 2. BM-002 → 3. BM-003 → 4. BM-004 → 5. BM-005 (**MVP gate:** vertical slice passes on compiled executable) → 6. BM-006 → 7. BM-007/BM-009 → 8. BM-008 → 9. BM-010. No item proceeds until the prior gate passes actual compile → executable → run verification.

## Status

All items are **NOT STARTED**. Implementation must not begin until all designs and repository initialization are complete. No tests have been run and no application binary exists for this product.
