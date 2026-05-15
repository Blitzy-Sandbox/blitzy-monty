# Monty

A minimal, secure Python interpreter written in Rust for use by AI.

> **Experimental** - This project is still in development, and not ready for the prime time.

Monty avoids the cost, latency, complexity and general faff of using a full container based sandbox for running LLM generated code. Instead, it lets you safely run Python code written by an LLM embedded in your agent, with startup times measured in single digit microseconds rather than hundreds of milliseconds.

## What Monty can do

- Run a reasonable subset of Python code - enough for your agent to express what it wants to do.
- Completely block access to the host environment: filesystem, environment variables, and network access are all implemented via external function calls the developer can control.
- Call functions on the host - only those functions you have explicitly granted access to.
- Run typechecking - Monty supports modern Python type hints and ships with [ty](https://docs.astral.sh/ty/) bundled in a single binary.
- Be snapshotted to bytes at external function calls, so you can store interpreter state in a file or database and resume later.
- Start up extremely fast (<1μs from code to execution result) with runtime performance similar to CPython (generally between 5x faster and 5x slower).
- Be called from Rust, Python, or JavaScript - because Monty has no dependency on CPython, it runs anywhere you can run Rust.
- Control resource usage - track memory usage, allocations, stack depth, and execution time, cancelling execution when preset limits are exceeded.
- Collect stdout and stderr and return them to the caller.
- Run async or sync code on the host via async or sync code in the guest.
- Use a small subset of the standard library: `sys`, `os`, `typing`, `asyncio`, `re`, `datetime`, `json`, and `dataclasses` (soon).

## What Monty cannot do

- Use the rest of the standard library.
- Use third party libraries (like Pydantic); support for external Python libraries is not a goal.
- Define classes (support coming soon).
- Use `match` statements (support coming soon).

## A single use case

In short, Monty is extremely limited and designed for **one** use case:

**To run code written by agents.**

## Motivation

For more context on why running agent-written code matters, see:

- [Codemode](https://blog.cloudflare.com/code-mode/) from Cloudflare
- [Programmatic Tool Calling](https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling) from Anthropic
- [Code Execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) from Anthropic
- [Smol Agents](https://github.com/huggingface/smolagents) from Hugging Face

The shared idea behind these efforts is that LLMs can work faster, cheaper, and more reliably when asked to write Python (or JavaScript) code rather than relying on traditional tool calling. Monty makes that possible without the complexity of a sandbox or the risk of running LLM-written code directly on the host.

## Pydantic AI codemode

Monty will (soon) be used to implement `codemode` in [Pydantic AI](https://github.com/pydantic/pydantic-ai). Instead of issuing sequential tool calls, the LLM writes Python code that calls your tools as ordinary functions, and Monty executes it safely.
