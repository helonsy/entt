# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EnTT is a header-only C++20 entity-component-system (ECS) library (v4.0.0). All source lives under `src/entt/`. It requires no compilation for use — consumers just include the headers.

## Build & Test Commands

```bash
# Configure with tests enabled (from repo root)
cmake -B build -DENTT_BUILD_TESTING=ON

# Build
cmake --build build -j4

# Run all tests
ctest --test-dir build -C Debug -j4

# Run a single test (by name regex)
ctest --test-dir build -R "registry"

# Build with sanitizers
cmake -B build -DENTT_BUILD_TESTING=ON -DENTT_USE_SANITIZER=ON
cmake --build build

# Build with clang-tidy
cmake -B build -DENTT_BUILD_TESTING=ON -DENTT_USE_CLANG_TIDY=ON
cmake --build build
```

Key CMake options: `ENTT_BUILD_BENCHMARK`, `ENTT_BUILD_EXAMPLE`, `ENTT_BUILD_LIB` (shared library tests), `ENTT_BUILD_DOCS` (Doxygen), `ENTT_ID_TYPE` (entity ID type, default `std::uint32_t`).

## Architecture

The library is organized into independent modules under `src/entt/`:

- **entity/** — Core ECS: `registry` (central store), `entity` (ID with version bits), `sparse_set` (underlying data structure), `storage` (typed component containers), `view`/`group` (queries over components), `organizer` (execution graph builder)
- **meta/** — Non-intrusive runtime reflection system with factory-based type registration, supporting properties, methods, constructors, conversions, and container introspection
- **signal/** — Event system: `delegate` (type-erased callable), `sigh` (signal handler / callback list), `dispatcher` (queued event dispatch), `emitter` (event emitter base)
- **core/** — Utilities: `any` (type-erased container), `type_info` (built-in RTTI replacement), `hashed_string` (compile-time string hashing), `compressed_pair`, memory utilities, algorithms
- **container/** — Custom containers: `dense_map`, `dense_set` (sparse-set-based hash containers), `table`
- **graph/** — `adjacency_matrix`, `flow` (task graph), `dot` (Graphviz export)
- **resource/** — Resource cache and loader system
- **process/** — Cooperative task scheduler
- **locator/** — Service locator pattern
- **poly/** — Static polymorphism via type erasure

Key design patterns:
- **Sparse set** is the foundational data structure — both ECS storage and hash containers build on it.
- **Mixins** layer behavior onto storage (e.g., `sigh_mixin` adds signals to component changes, `reactive_mixin` for reactive queries). Controlled via `ENTT_NO_MIXIN`.
- **Configurable via macros** in `config/config.h`: entity ID type (`ENTT_ID_TYPE`), page sizes (`ENTT_SPARSE_PAGE`, `ENTT_PACKED_PAGE`), exception handling, atomics (`ENTT_USE_ATOMIC`), assertions, DLL export.

## Code Style

- Formatting: `.clang-format` (LLVM-based, 4-space indent, no column limit, pointer right-aligned `int *ptr`)
- Static analysis: `.clang-tidy` configured
- C++20 minimum; CI tests C++20 and C++23
- Tested compilers: GCC 12–14, Clang 16–18, MSVC v143, AppleClang

## Test Structure

Tests use GoogleTest (fetched automatically via CMake FetchContent). Test files mirror the module structure under `test/entt/`. Additional test categories:
- `test/benchmark/` — performance benchmarks
- `test/lib/` — cross-DLL boundary tests
- `test/snapshot/` — serialization tests (Cereal integration)
- `test/example/` — usage examples as tests
