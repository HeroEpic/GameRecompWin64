# AGENTS.md

Static recompilation framework for converting legacy 32-bit Windows games into native 64-bit executables. General framework, not game-specific. Inspired by modern recompilation efforts (e.g. Zelda 64 recomp) but designed as a reusable platform.

## Toolchain

- **Compiler**: MinGW-w64 (GCC targeting Windows x64). Use this instead of MSVC.
- **Reverse engineering**: radare2 (`r2`, `rabin2`, etc.) for disassembly and analysis of original 32-bit executables. Drives the `tools/` reverse-engineering utilities.

## Build

CMake + MinGW-w64 (Windows) or Clang/GCC (Linux).

```
cmake -S . -B build
cmake --build build --config Release
```

Or via presets:

```
cmake --preset release
cmake --build --preset release
```

Primary output: `GameRecomp.exe` (Windows x64). Future: Linux x64, Linux ARM64, Android ARM64.

## Test

```
ctest --test-dir build
```

Behavioral validation is non-negotiable and goes beyond unit tests:
- Compare execution traces against the original executable.
- Verify save files, game logic, physics, and rendering outputs.
- Regression tests must confirm behavior is identical to the original after recompilation.

A recompiled function is not complete until validated against the original executable.

## Lint / Format

```
clang-format -i <files>
clang-tidy
cppcheck .
```

## Layout

- `src/core/` — recompiled game logic (the ported 32-bit code)
- `src/engine/` — runtime framework hosting the recompiled logic
- `src/renderer/` — Vulkan renderer (graphics backend only)
- `src/audio/`, `src/input/`, `src/platform/` — platform abstraction layers
- `src/main.cpp` — entrypoint
- `include/` — public headers
- `tools/` — reverse-engineering utilities (not shipped in runtime)
- `tests/` — unit + regression tests
- `assets/`, `docs/` — data and documentation

## Architectural constraints (read before editing)

- **Behavioral fidelity > optimization.** Any optimization MUST preserve original game logic. When in doubt, match the original.
- **Decompiled code is a reference, not the deliverable.** Rewrite into clean, maintainable C++ rather than shipping raw decompiler output.
- **Graphics backend is the only thing the Vulkan renderer replaces.** Gameplay logic stays platform-independent and should never call renderer APIs directly.
- **Avoid Windows-specific APIs.** Prefer cross-platform libraries (SDL3 for windowing/input/audio) so future Linux/ARM64/Android ports don't require rewrites.
- **Stay modular.** Alternative rendering backends or native Linux builds must be addable without major rewrites — respect the existing layer boundaries in `src/`.
- **Per-function validation against the original executable is the definition of done.**

## Gotchas

- `src/core/` code may look decompiler-generated; do not assume it follows the rest of the codebase's style. Clean it up when touching it.
- Renderer changes must not leak into core logic. If you find yourself editing `src/core/` for a graphics task, reconsider.
- Treat any "optimization" opportunity in `src/core/` as suspicious unless it provably preserves behavior.
