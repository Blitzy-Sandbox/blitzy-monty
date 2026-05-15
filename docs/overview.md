# Overview

This page summarises how the Monty repository is laid out, how it integrates with Pydantic AI, and how it compares to alternative approaches for running LLM-written code.

## Repository structure

The root of the repository contains the following top-level items:

- `crates/` - the Rust workspace where the interpreter, type checker integration, and language bindings live.
- `pyproject.toml` - Python package metadata for `pydantic-monty`, the PyO3-backed binding distributed on PyPI.
- `Cargo.toml` / `Cargo.lock` - Rust workspace manifest and lockfile.
- `examples/` - runnable example programs demonstrating Python, Rust, and Pydantic AI usage.
- `scripts/` - helper scripts, including `startup_performance.py` used to produce the latency numbers in the alternatives table below.
- `Makefile` - aggregated developer commands for building, testing, linting, and benchmarking across the Rust and Python toolchains.
- `.github/` - CI workflows, issue templates, and other GitHub configuration.

## PydanticAI integration

Monty will power code-mode in [Pydantic AI](https://github.com/pydantic/pydantic-ai). Rather than emitting sequential tool calls, the LLM writes a Python program that calls your tools as ordinary functions, and Monty executes that program safely. See the upstream README for a complete example using a `CodeModeToolset` and `FunctionToolset` with `pydantic_ai.Agent`.

## Alternatives

People generally have one of two reactions when they first encounter Monty: either "this solves so many problems, I want it", or "why not X?" - where X is some other technology.

The table below summarises Monty against the most common alternatives. None of the listed projects are bad - most were never designed to be an LLM sandbox.

| Tech               | Language completeness | Security     | Start latency   | FOSS       | Setup complexity | File mounting  | Snapshotting |
| ------------------ | --------------------- | ------------ | --------------- | ---------- | ---------------- | -------------- | ------------ |
| Monty              | partial               | strict       | 0.06 ms         | free / OSS | easy             | easy           | easy         |
| Docker             | full                  | good         | 195 ms          | free / OSS | intermediate     | easy           | intermediate |
| Pyodide            | full                  | poor         | 2800 ms         | free / OSS | intermediate     | easy           | hard         |
| starlark-rust      | very limited          | good         | 1.7 ms          | free / OSS | easy             | not available? | impossible?  |
| WASI / Wasmer      | partial, almost full  | strict       | 66 ms           | free \*    | intermediate     | easy           | intermediate |
| sandboxing service | full                  | strict       | 1033 ms         | not free   | intermediate     | hard           | intermediate |
| YOLO Python        | full                  | non-existent | 0.1 ms / 30 ms  | free / OSS | easy             | easy / scary   | hard         |

Startup numbers come from `scripts/startup_performance.py` in the repository.

### Monty
- Partial language support; strict security via explicit external function calls; microsecond startup.
- `pip install pydantic-monty`, ~4.5MB download.
- Snapshotting is trivial via `dump()`/`load()`.

### Docker
- Full CPython + any library; process and FS isolation; ~195ms cold start.
- Requires Docker daemon and images.

### Pyodide
- Full CPython compiled to WASM; ~2800ms WASM load.
- Not designed for server-side isolation.

### starlark-rust
- Configuration language, not Python — no classes, exceptions, or async.
- Deterministic and hermetic by design.

### WASI / Wasmer
- Full CPython via WebAssembly; strong sandboxing guarantees in principle.
- Wasmer Python package is unmaintained; called as subprocess with ~66ms startup.

### Sandboxing services
- Daytona, E2B, Modal — full CPython, professionally-managed container isolation.
- ~1s cold start including network; warm containers can be faster.

### YOLO Python
- `exec()` directly. ~0.1ms (or ~30ms via subprocess).
- Zero security.

## Trade-off summary

Monty trades language completeness for tight security and microsecond startup. For the specific job of running short snippets of Python authored by an LLM inside an agent loop, Monty's combination of microsecond startup, strict host isolation, snapshotting, and a single small dependency is what motivates building a new interpreter rather than picking one off the shelf.
