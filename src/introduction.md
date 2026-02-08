# Intent Contract Language (ICL)

**ICL** is a deterministic, language-agnostic specification language for intent contracts — formal documents that describe what a system should do, how it should behave, and what it guarantees.

## What is an Intent Contract?

An intent contract formally specifies:

- **What** a system should do (purpose and operations)
- **How** it should behave (preconditions, postconditions, invariants)
- **What it must not do** (constraints, refusals, resource limits)
- **What it guarantees** (commitments to the user)

Think of it like OpenAPI for human intent and AI agent constraints — but machine-checkable and deterministic.

## Why ICL?

Every AI system, API, and automation tool has implicit contracts. Today, these live in:
- Natural language documentation (ambiguous)
- Type definitions (incomplete)
- Test cases (partial)
- Informal agreements (unverifiable)

ICL replaces all of these with a **single, machine-checkable, deterministic** format. Given the same contract and the same input, every ICL implementation produces identical output — always.

## Quick Example

```
Contract {
  Identity {
    stable_id: "ic-hello-001",
    version: 1,
    created_timestamp: 2026-02-01T10:00:00Z,
    owner: "developer",
    semantic_hash: "e5f6a7b8c9d0"
  }
  PurposeStatement {
    narrative: "Simple contract that echoes input messages",
    intent_source: "tutorial",
    confidence_level: 1.0
  }
  DataSemantics {
    state: { message: String, count: Integer = 0 },
    invariants: ["message is not empty", "count >= 0"]
  }
  BehavioralSemantics {
    operations: [{
      name: "echo",
      precondition: "input_provided",
      parameters: { message: String },
      postcondition: "state_updated_with_message",
      side_effects: ["log_operation"],
      idempotence: "idempotent"
    }]
  }
  ExecutionConstraints {
    trigger_types: ["manual"],
    resource_limits: {
      max_memory_bytes: 1048576,
      computation_timeout_ms: 100,
      max_state_size_bytes: 1048576
    },
    external_permissions: [],
    sandbox_mode: "full_isolation"
  }
  HumanMachineContract {
    system_commitments: ["All messages are echoed"],
    system_refusals: ["Will not modify past messages"],
    user_obligations: ["May provide new messages"]
  }
}
```

## The ICL Ecosystem

| Repository | What It Contains |
|------------|-----------------|
| [ICL-Spec](https://github.com/ICL-System/ICL-Spec) | The standard — BNF grammar, core specification, conformance tests |
| [ICL-Runtime](https://github.com/ICL-System/ICL-Runtime) | Canonical Rust implementation, CLI, and language bindings |
| ICL-Docs (this site) | Documentation you're reading now |

## Core Guarantees

- **Deterministic**: Same input always produces identical output — no randomness, no system time dependency
- **Verifiable**: All contract properties are machine-checkable (types, invariants, coherence)
- **Bounded**: All execution is bounded in memory, time, and state size
- **Canonical**: One normalized form per contract, with SHA-256 semantic hashing
- **Language-agnostic**: One Rust core compiled to Python, JavaScript, and Go

## Current Status

ICL is in **active development**. The core pipeline is fully implemented:

| Component | Status | Tests |
|-----------|--------|-------|
| Parser (tokenizer + recursive descent) | Complete | 80+ tests |
| Normalizer (canonical form + hashing) | Complete | 30+ tests |
| Verifier (types, invariants, determinism, coherence) | Complete | 40+ tests |
| Executor (sandbox + provenance logging) | Complete | 20+ tests |
| CLI (9 commands) | Complete | 28 tests |
| Python binding (PyO3) | Complete | 18 tests |
| JavaScript binding (WASM) | Complete | 31 tests |
| Go binding (cgo) | Complete | 16 tests |

> **Total: 268 tests passing** — including 100-iteration determinism proofs across all components and all language bindings.

## Getting Started

Ready to try ICL? Start with the [Quick Start Guide](getting-started.md).
