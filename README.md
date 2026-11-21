# Karkain Build Manager

A CLI/TUI build orchestrator for Karkain workspaces: dependency graphs, incremental rebuilds and build artifacts — distinct from the compiler.

## Status

| Area | Status |
|---|---|
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain Build Manager orchestrates builds rather than compiling source itself. It reads the workspace dependency graph (`workspace graph`), decides what is out of date using content hashes, schedules build steps from the Karkain toolchain, caches artifacts, and records `karkain-diagnostics-v1` codes plus `bench`/`prof` output as build artifacts. It can verify dependencies with `pkg audit`/`pkg verify` and `crypto` hashing. Everything is exposed through a scriptable CLI, JSON artifacts and a TUI dashboard. It is deliberately separate from the Karkain compiler (which does the actual compile) and from Karkain IDE (the full development environment) and Karkain Project Manager (task/milestone planning).

## Who it serves

- Karkain developers with multi-package workspaces who need fast, correct incremental builds.
- CI/agent integrators wanting deterministic build graphs, exit codes and machine-readable artifacts.
- Teams on modest hardware (about 4 GB RAM) who want caching without an external JVM or Starlark runtime.
- Supply-chain-conscious users who want local package verification with no new language in the loop.

## Planned features

- **Dependency graph (planned):** consume `workspace graph` into a target/dependency model with cycle detection.
- **Incremental builds (planned):** content-hash inputs (`crypto` sha256) to skip unchanged targets.
- **Scheduling (planned):** run independent steps cooperatively using `async` channels/pool.
- **Artifact & cache management (planned):** store outputs and cache keys, with JSON build manifests.
- **Diagnostics & reporting (planned):** capture `karkain-diagnostics-v1` codes and `bench`/`prof` output as artifacts.
- **Supply-chain checks (planned):** `pkg audit`/`verify` plus `crypto` hashes for dependency integrity.
- **CLI/TUI dashboard (planned):** build, rebuild, graph view and artifact inspection.
- **JSON API (planned):** scriptable build/query commands for CI and agents.

## Explicit non-goals

- Not the compiler — Karkain v1.2.0 performs compilation; this product orchestrates it.
- Not a general-purpose build DSL for other languages; Karkain workspaces only.
- No native desktop GUI in v1.2.0 (GUI toolkit BLOCKED); GUI is a planned later layer, not shipped.
- No remote execution, hermetic sandboxing or distributed cache claims (gRPC/TLS capabilities BLOCKED/UNVERIFIED).
- No pricing, adoption or benchmark claims.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

Windows x86-64 toolchain VERIFIED (`karkain.exe` reports v1.2.0; C-free native Windows build → executable → run verified). Linux cross-target output PRESENT but execution NOT VERIFIED on this Windows host. macOS NOT VERIFIED.

## Build and run status

No application binary exists yet. Intended generic commands (generic form only; no machine-specific paths):

```
karkain build src/main.kark -o karkain-build
karkain run src/main.kark
```

These become active when implementation starts. On this machine the verified toolchain path uses the Go front-end engine and an explicit C-free native Windows target; the default `kcc` engine is currently unusable in this environment.

## Known blockers

- Native GUI toolkit: BLOCKED — TUI/CLI first; GUI planned only.
- Concurrent HTTP server/routing and TLS/WebSocket: BLOCKED — no remote execution or web dashboard.
- `filesystem` metadata (mtime/permissions): PARTIAL via PowerShell — use content hashing instead.
- `json` PARTIAL (`json_get_field` only) — build manifests use a restricted shape.
- `system`/`env`/`process`/`time`: PARTIAL shell wrappers — probe before reliance; cross-platform execution NOT VERIFIED.

## Repository

GitHub: https://github.com/ajit-ai/Karkain-Build-Manager · Issues: https://github.com/ajit-ai/Karkain-Build-Manager/issues · License: Apache-2.0.
