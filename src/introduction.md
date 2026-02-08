# Intent Contract Language (ICL)

**ICL** is a universal, language-agnostic specification language for intent contracts — like OpenAPI but for human intent and AI agent constraints.

## What is an Intent Contract?

An intent contract formally specifies:

- **What** a system should do (purpose and operations)
- **How** it should behave (preconditions, postconditions, invariants)
- **What it must not do** (constraints, refusals, resource limits)
- **What it guarantees** (commitments to the user)

## Why ICL?

Every AI system, API, and automation tool needs contracts. Today, these are scattered across:
- Natural language documentation (ambiguous)
- Type definitions (incomplete)
- Test cases (partial)
- Informal agreements (unverifiable)

ICL replaces all of these with a **single, machine-checkable, deterministic** format.

## The ICL Ecosystem

| Repository | What It Contains |
|------------|-----------------|
| [ICL-Spec](https://github.com/ICL-System/ICL-Spec) | The standard — BNF grammar, specification, conformance tests |
| [ICL-Runtime](https://github.com/ICL-System/ICL-Runtime) | Canonical Rust implementation, CLI, language bindings |
| ICL-Docs (this site) | Documentation you're reading now |

## Current Status

ICL is in **early development** (Phase 0). The specification is defined, the project structure is established, and the parser is being implemented.

> **Honest status**: The runtime compiles and has 20 passing tests, but all core components (parser, normalizer, verifier, executor) are stubs. Real implementation starts in Phase 1.

## Getting Started

Ready to learn more? Start with the [Quick Start Guide](getting-started.md).
