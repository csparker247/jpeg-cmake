# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`jpeg-cmake` adds CMake build support to IJG's libjpeg. The actual libjpeg C source lives in `libjpeg/` (not tracked in git — populated by copying from a libjpeg release). All CMake files are maintained in `resources/` and copied into `libjpeg/` at configure time via `configure_file()` in the root `CMakeLists.txt`.

## Build Commands

```shell
# Configure and build
cmake -S . -B build/
cmake --build build/

# Run tests (requires BUILD_EXECUTABLES=ON, the default)
cmake --build build/
cmake --build build/ --target test
# or
cd build && ctest
```

## Architecture

### Two-level CMake structure
- **Root `CMakeLists.txt`**: Copies all files from `resources/` into `libjpeg/` using `configure_file(...COPYONLY)`, then calls `add_subdirectory(libjpeg)`.
- **`resources/CMakeLists.txt`** (the real build logic): Defines all targets — `jpeg_objs` (OBJECT library), `jpeg` (shared), `jpeg_static` (static), and the utility executables (`cjpeg`, `djpeg`, `jpegtran`, `rdjpgcom`, `wrjpgcom`). This file gets copied to `libjpeg/CMakeLists.txt`.

### `jconfig.h` generation
`resources/ConfigureJConfig.cmake` detects system capabilities and exposes CMake options, then feeds them into `resources/jconfig.h.in` (a `#cmakedefine`-based template) to produce `libjpeg/jconfig.h`. This is included by `resources/CMakeLists.txt`.

### Key CMake options
| Flag | Default | Description |
|------|---------|-------------|
| `BUILD_SHARED_LIBS` | ON | Build shared library |
| `BUILD_STATIC_LIBS` | ON | Build static library |
| `BUILD_EXECUTABLES` | ON | Build cjpeg/djpeg/jpegtran/etc. |
| `BUILD_TESTS` | ON | Generate CTest targets |
| `LINK_STATIC` | OFF | Link executables against static lib |
| `BUILD_ALT_UI` | OFF | Alternate CLI for cjpeg/djpeg |
| `DEFAULT_FMT` | (none) | Default output format for djpeg |

### Versioning
The project version in `resources/CMakeLists.txt` is read dynamically from `libjpeg/configure.ac` at configure time. The root `CMakeLists.txt` version should match the libjpeg release version (e.g., `10.0.0`).

## Updating libjpeg source

```shell
rm -rf libjpeg/*
cp -a ~/jpeg-10/ libjpeg/
cmake -S . -B build/
cmake --build build/
```

## Standalone use

The files in `resources/` can be copied directly into any libjpeg source directory for standalone CMake support without this wrapper repo.
