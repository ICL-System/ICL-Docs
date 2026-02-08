# Quick Start

> **Note**: ICL is in early development. The CLI and runtime are scaffolded but not yet functional. This guide shows where things are headed.

## Prerequisites

- [Rust 1.75+](https://www.rust-lang.org/tools/install) (for building from source)

## Install the CLI (from source)

```bash
git clone https://github.com/ICL-System/ICL-Runtime.git
cd ICL-Runtime
cargo install --path crates/icl-cli
```

## Your First Contract

Create a file called `hello.icl`:

```
Contract {
  Identity {
    stable_id: "hello-world-001"
    version: 1
    created_timestamp: "2025-01-01T00:00:00Z"
    owner: "your-name"
    semantic_hash: "placeholder"
  }
  PurposeStatement {
    narrative: "A simple greeting contract"
    intent_source: "human_authored"
    confidence_level: 1.0
  }
  DataSemantics {
    state: {
      greeting: String = "Hello, World!"
    }
    invariants: [
      "greeting.length > 0"
    ]
  }
  BehavioralSemantics {
    operations: []
  }
  ExecutionConstraints {
    trigger_types: ["manual"]
    resource_limits: {
      max_memory_bytes: 1048576
      computation_timeout_ms: 1000
      max_state_size_bytes: 65536
    }
    external_permissions: []
    sandbox_mode: "strict"
  }
  HumanMachineContract {
    system_commitments: ["Always produce a greeting"]
    system_refusals: ["Will not produce empty output"]
    user_obligations: []
  }
}
```

## Validate (once implemented)

```bash
icl validate hello.icl
# ✓ hello.icl is valid

icl verify hello.icl
# ✓ Types OK
# ✓ Invariants consistent
# ✓ Determinism verified
```

## Next Steps

- [Writing Contracts](writing-contracts.md) — learn the full ICL syntax
- [Specification Overview](specification/overview.md) — understand the formal model
- [CLI Reference](cli/reference.md) — all available commands
