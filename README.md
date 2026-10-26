# u360

An Xbox 360 recompilation workbench. It dumps disc images, parses the executable
container, lifts Xenon (PowerPC) code to C, and compiles that C into a native
library. A PySide6 GUI and a headless CLI drive the same pipeline.

## Requirements

- Python 3.11+
- CMake 3.20+ and a C99 compiler (clang, gcc, or MSVC) for the build step
- Your own disc images and console key material

## Install and run

    pip install -e ".[dev]"
    u360                      # launches the GUI
    pytest -q                 # run the test suite

## CLI

    u360 dump game.iso -o ./dumps/game
    u360 inspect ./dumps/game/default.xex
    u360 recomp guest.bin --base 0x82000000 --entry 0x82001000 -o ./out --build

## Layout

    src/u360/
      xiso.py            XDVDFS disc reader
      xex.py             XEX2 header parser; decryption stage stub
      ppc/decode.py      PowerPC instruction decoder with CFG metadata
      recomp/analysis.py function discovery
      recomp/lifter.py   PowerPC to C translation
      runtime/u360rt.h   guest memory, CR and CTR semantics, fault helpers
      pipeline.py        orchestration shared by GUI and CLI
      gui.py             PySide6 interface
      config.py          TOML settings
    docs/ROADMAP.md      status matrix and milestones

## Design notes

- Lifting is literal. Each guest register is a slot in `U360Ctx`, each branch is a
  goto or call, and unknown opcodes become runtime faults. Nothing is silently guessed.
- Guest memory is accessed through bounds-checked helpers that byte-swap for big-endian.
- Undiscovered call targets link to stubs that fault with the address, so partial
  coverage is visible instead of failing at link time.
- Progress reporting is a callback. The GUI and CLI share the same code path.

## Legal

This project contains no game content and no console keys. Only run it on discs and
key material you legally own.

## Standalone app

PyInstaller cannot cross-compile, so build on the OS you are targeting:

    packaging/build.sh          # Linux or macOS
    packaging\build.ps1         # Windows (PowerShell)

Or push a `v*` tag / run the "release" workflow in GitHub Actions to build
Windows, macOS, and Linux artifacts in CI. Output lands in `dist/`.
