# AGENTS.md

Project recompiles legacy 32-bit Windows games into native 64-bit executables via static recompilation. It is a general framework, not a single-game port. Long-term targets: modular renderer (Vulkan), Wine/Proton compatibility, Linux/ARM64 ports.

## Build (CMake + MSVC on Windows, Clang/GCC on Linux)

```sh
cmake -S . -B build
cmake --build build --config Release
```

Or via presets (if `CMakePresets.json` is present):

```sh
cmake --preset release
cmake --build --preset release
```

Main binary: `GameRecomp.exe` (Windows x64).

## Test

```sh
ctest --test-dir build
```

Behavioral validation is not automated: compare execution traces against the original executable; verify save files, game logic, physics, render output. Each recompiled function must be validated against the original executable before being considered complete.

## Lint / format

- Format: `clang-format -i <files>`
- Static analysis: `clang-tidy`
- Optional: `cppcheck .`

## Layout (entrypoints)

- `src/core/` — recompiled game logic
- `src/engine/` — runtime framework
- `src/renderer/` — Vulkan renderer (graphics backend only)
- `src/main.cpp` — entrypoint
- `tools/` — reverse-engineering utilities
- `tests/` — unit + regression tests

## Critical constraints (project-specific, easy to violate by default)

- **Behavioral accuracy trumps optimization.** Any optimization must preserve original game logic. Regressions against original behavior block completion.
- **Decompiled code is a reference, not a destination.** Rewrite into clean, maintainable C++; do not ship raw decompiler output.
- **Renderer is graphics-only.** Gameplay logic must stay platform-independent. Do not couple gameplay to Vulkan or any specific backend.
- **Avoid Windows-specific APIs.** Use cross-platform libs (SDL3 for windowing/input/audio) even on Windows, to keep Linux/ARM64 ports viable.
- **Keep modules modular.** Alternative renderers and native Linux builds must be addable without major rewrites.
