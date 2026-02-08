# Writing Your First Contract

> **Status**: This guide describes the ICL contract syntax. The parser is not yet implemented — use this as a reference for what's coming.

## Contract Structure

Every ICL contract has the following sections:

```
Contract {
  Identity { ... }
  PurposeStatement { ... }
  DataSemantics { ... }
  BehavioralSemantics { ... }
  ExecutionConstraints { ... }
  HumanMachineContract { ... }
  Extensions { ... }     // optional
}
```

## Identity

Uniquely identifies the contract:

```
Identity {
  stable_id: "my-contract-001"
  version: 1
  created_timestamp: "2025-01-01T00:00:00Z"
  owner: "your-name"
  semantic_hash: "auto-computed"
}
```

## PurposeStatement

Human-readable description of intent:

```
PurposeStatement {
  narrative: "Manages user preferences with full audit logging"
  intent_source: "human_authored"
  confidence_level: 0.95
}
```

## DataSemantics

Defines state and invariants:

```
DataSemantics {
  state: {
    count: Integer = 0
    name: String
    items: Array<String>
  }
  invariants: [
    "count >= 0"
    "name.length <= 255"
  ]
}
```

### Primitive Types
- `Integer` — 64-bit signed integer
- `Float` — 64-bit floating point
- `String` — UTF-8 text
- `Boolean` — true or false
- `ISO8601` — timestamp
- `UUID` — universally unique identifier

### Composite Types
- `Array<T>` — ordered collection
- `Map<K, V>` — key-value mapping
- `Object { ... }` — structured record
- `Enum { ... }` — tagged union

## BehavioralSemantics

Defines operations with pre/postconditions:

```
BehavioralSemantics {
  operations: [
    {
      name: "increment"
      precondition: "count < 1000"
      parameters: {}
      postcondition: "count == old.count + 1"
      side_effects: []
      idempotence: "not_idempotent"
    }
  ]
}
```

## ExecutionConstraints

Bounds for execution:

```
ExecutionConstraints {
  trigger_types: ["api_call", "scheduled"]
  resource_limits: {
    max_memory_bytes: 10485760
    computation_timeout_ms: 5000
    max_state_size_bytes: 1048576
  }
  external_permissions: ["network:api.example.com"]
  sandbox_mode: "strict"
}
```

## HumanMachineContract

Commitments and refusals:

```
HumanMachineContract {
  system_commitments: [
    "All state changes are logged"
    "No data leaves the sandbox"
  ]
  system_refusals: [
    "Will not delete data without confirmation"
  ]
  user_obligations: [
    "Must provide valid input format"
  ]
}
```

## Next Steps

- See [example contracts](https://github.com/ICL-System/ICL-Spec/tree/main/examples) in the spec repo
- Read the [full specification](specification/overview.md)
