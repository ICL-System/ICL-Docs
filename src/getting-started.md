# Quick Start

Get up and running with ICL in under 5 minutes.

## Install the CLI

### From Source (Rust)

```bash
git clone https://github.com/ICL-System/ICL-Runtime.git
cd ICL-Runtime
cargo install --path crates/icl-cli
```

### Python

```bash
pip install icl-runtime
```

> The Python package is published on [test.pypi.org](https://test.pypi.org/project/icl-runtime/) during development.

### JavaScript / Node.js

```bash
npm install icl-runtime
```

> Not yet published to npm. During development, build from source:
>
> ```bash
> cd ICL-Runtime/bindings/javascript
> wasm-pack build --target nodejs
> ```

### Go

```bash
cd ICL-Runtime/bindings/go/ffi
cargo build --release
```

Then import the Go module from `bindings/go/`.

## Create Your First Contract

Scaffold a new contract:

```bash
icl-cli init hello
# ✓ created hello.icl
```

Or create `hello.icl` manually:

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
    state: {
      message: String,
      count: Integer = 0
    },
    invariants: [
      "message is not empty",
      "count >= 0"
    ]
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

## Validate

Check that the contract parses correctly:

```bash
icl-cli validate hello.icl
# ✓ hello.icl is valid
```

Get machine-readable output with `--json`:

```bash
icl-cli validate hello.icl --json
# {"file":"hello.icl","valid":true,"errors":0,"warnings":2,"diagnostics":[...]}
```

## Verify

Run full verification — types, invariants, determinism, and coherence:

```bash
icl-cli verify hello.icl
# ✓ hello.icl verified successfully
```

## Normalize

Get the canonical form (sorted sections, sorted fields, computed hash):

```bash
icl-cli normalize hello.icl
```

The normalizer produces a deterministic output — running it twice always gives the same result.

## Compute Hash

Get the SHA-256 semantic hash:

```bash
icl-cli hash hello.icl
# 1f7dcf67d92b813f3cc0402781f023ea33c76dd7c2b6963531fe68bf9c032cb8
```

Two contracts with the same semantics always produce the same hash, regardless of formatting or comment differences.

## Execute

Run a contract with inputs:

```bash
icl-cli execute hello.icl --input '{"operation":"echo","inputs":{"message":"Hello"}}'
# ✓ hello.icl executed successfully
#   Operations: 1
#   Provenance entries: 1
```

## Compare Contracts

Semantic diff between two contracts:

```bash
icl-cli diff v1.icl v2.icl
# Shows field-by-field differences in canonical form
```

## Using from Python

```python
import icl

# Parse
result = icl.parse_contract(open("hello.icl").read())
print(result)  # JSON string of the parsed contract

# Normalize
canonical = icl.normalize(open("hello.icl").read())

# Verify
issues = icl.verify(open("hello.icl").read())

# Execute
output = icl.execute(
    open("hello.icl").read(),
    '{"operation":"echo","inputs":{"message":"Hello"}}'
)

# Semantic hash
hash_val = icl.semantic_hash(open("hello.icl").read())
```

## Using from JavaScript

```javascript
const icl = require('./path-to/pkg/icl_runtime.js');

// Parse
const result = icl.parseContract(contractText);

// Normalize
const canonical = icl.normalize(contractText);

// Verify
const issues = icl.verify(contractText);

// Execute
const output = icl.execute(contractText, '{"operation":"echo","inputs":{"message":"Hello"}}');

// Semantic hash
const hash = icl.semanticHash(contractText);
```

## Using from Go

```go
package main

import (
    icl "github.com/ICL-System/ICL-Runtime/bindings/go"
    "fmt"
)

func main() {
    contract := `Contract { ... }` // your ICL text

    result, err := icl.ParseContract(contract)
    if err != nil { panic(err) }
    fmt.Println(result)

    canonical, _ := icl.Normalize(contract)
    hash, _ := icl.SemanticHash(contract)
    fmt.Println(canonical, hash)
}
```

## Next Steps

- [Writing Contracts](writing-contracts.md) — learn the full ICL syntax and type system
- [CLI Reference](cli/reference.md) — all 9 CLI commands
- [Specification Overview](specification/overview.md) — the formal model
- [Architecture](runtime/architecture.md) — how the runtime works
