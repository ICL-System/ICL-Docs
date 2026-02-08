# Writing Contracts

This guide covers the full ICL contract syntax, all sections, and the type system.

## Contract Structure

Every ICL contract follows this structure:

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

All sections except `Extensions` are required. Fields within a section are separated by commas.

---

## Identity

Uniquely identifies the contract. Every field is required.

```
Identity {
  stable_id: "ic-my-contract-001",
  version: 1,
  created_timestamp: 2026-02-01T10:00:00Z,
  owner: "your-name",
  semantic_hash: "placeholder"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `stable_id` | String | Unique identifier, never changes across versions |
| `version` | Integer | Monotonically increasing version number |
| `created_timestamp` | ISO8601 | When this version was created |
| `owner` | String | Author or maintainer |
| `semantic_hash` | String | SHA-256 hash of canonical form (auto-computed by `icl normalize`) |

> The `semantic_hash` is automatically recomputed when you run `icl normalize`. You can use a placeholder during authoring.

---

## PurposeStatement

Human-readable description of the contract's intent.

```
PurposeStatement {
  narrative: "Manages user preferences with full audit logging",
  intent_source: "human_authored",
  confidence_level: 0.95
}
```

| Field | Type | Description |
|-------|------|-------------|
| `narrative` | String | What this contract does, in plain language |
| `intent_source` | String | Who/what authored it (e.g., `"human_authored"`, `"ai_generated"`, `"developer_specification"`) |
| `confidence_level` | Float | 0.0 to 1.0 — how confident the author is in the contract's correctness |

---

## DataSemantics

Defines the contract's state and invariants.

```
DataSemantics {
  state: {
    count: Integer = 0,
    name: String,
    items: Array<String>,
    active: Boolean = true
  },
  invariants: [
    "count >= 0",
    "name.length <= 255"
  ]
}
```

### State Fields

Each state field has a name, a type, and an optional default value:

```
field_name: Type = default_value
```

### Primitive Types

| Type | Description | Example Values |
|------|-------------|----------------|
| `Integer` | 64-bit signed integer | `0`, `42`, `-1` |
| `Float` | 64-bit IEEE 754 floating point | `3.14`, `0.0`, `-1.5` |
| `String` | UTF-8 text | `"hello"`, `""` |
| `Boolean` | Logical value | `true`, `false` |
| `ISO8601` | Timestamp | `2026-02-01T10:00:00Z` |
| `UUID` | Universally unique identifier | `"550e8400-e29b-41d4-a716-446655440000"` |

### Composite Types

| Type | Syntax | Example |
|------|--------|---------|
| `Object` | `Object { field: Type, ... }` | `Object { name: String, age: Integer }` |
| `Enum` | `Enum { Variant1, Variant2, ... }` | `Enum { Active, Inactive, Suspended }` |
| `Array` | `Array<T>` | `Array<String>`, `Array<Integer>` |
| `Map` | `Map<K, V>` | `Map<String, Integer>` |

### Invariants

Invariants are string expressions that must always hold for the contract's state. The verifier checks that:
- Invariants are consistent (no contradictions)
- Invariants can be satisfied by initial state
- Operations preserve invariants

---

## BehavioralSemantics

Defines operations — the things the contract can do.

```
BehavioralSemantics {
  operations: [
    {
      name: "increment",
      precondition: "count < 1000",
      parameters: {},
      postcondition: "count == old.count + 1",
      side_effects: [],
      idempotence: "not_idempotent"
    },
    {
      name: "set_name",
      precondition: "true",
      parameters: { new_name: String },
      postcondition: "name == new_name",
      side_effects: ["log_change"],
      idempotence: "idempotent"
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `name` | String | Operation identifier, must be unique within the contract |
| `precondition` | String | Condition that must be true before execution |
| `parameters` | Object | Named parameters with types |
| `postcondition` | String | Condition that must be true after execution |
| `side_effects` | Array\<String\> | Declared side effects (for transparency) |
| `idempotence` | String | `"idempotent"` or `"not_idempotent"` |

---

## ExecutionConstraints

Bounds for execution — resource limits and sandbox configuration.

```
ExecutionConstraints {
  trigger_types: ["api_call", "scheduled"],
  resource_limits: {
    max_memory_bytes: 10485760,
    computation_timeout_ms: 5000,
    max_state_size_bytes: 1048576
  },
  external_permissions: ["network:api.example.com"],
  sandbox_mode: "full_isolation"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `trigger_types` | Array\<String\> | How the contract can be invoked (`"manual"`, `"api_call"`, `"scheduled"`) |
| `resource_limits` | Object | Memory, time, and state size bounds |
| `external_permissions` | Array\<String\> | Declared external access (empty = fully sandboxed) |
| `sandbox_mode` | String | `"full_isolation"`, `"strict"`, or `"relaxed"` |

### Resource Limits

| Field | Type | Description |
|-------|------|-------------|
| `max_memory_bytes` | Integer | Maximum memory usage in bytes |
| `computation_timeout_ms` | Integer | Maximum execution time in milliseconds |
| `max_state_size_bytes` | Integer | Maximum state size in bytes |

---

## HumanMachineContract

The social contract between the system and the user.

```
HumanMachineContract {
  system_commitments: [
    "All state changes are logged",
    "No data leaves the sandbox"
  ],
  system_refusals: [
    "Will not delete data without confirmation"
  ],
  user_obligations: [
    "Must provide valid input format"
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `system_commitments` | Array\<String\> | What the system promises to do |
| `system_refusals` | Array\<String\> | What the system explicitly will not do |
| `user_obligations` | Array\<String\> | What the user is expected to do |

---

## Extensions (Optional)

Domain-specific extensions with namespaced key-value pairs.

```
Extensions {
  monitoring: {
    alert_threshold: 0.95,
    dashboard_url: "https://monitor.example.com"
  },
  compliance: {
    framework: "SOC2",
    last_audit: 2025-12-01T00:00:00Z
  }
}
```

Extension namespaces are isolated — they cannot affect core contract semantics.

---

## Comments

ICL supports single-line comments:

```
// This is a comment
Contract {
  // Comments can appear anywhere
  Identity {
    stable_id: "example",  // inline comments too
    ...
  }
}
```

Comments are stripped during normalization and do not affect the semantic hash.

---

## Complete Example

Here is a full, working contract for a database write validator:

```
Contract {
  Identity {
    stable_id: "ic-db-write-001",
    version: 1,
    created_timestamp: 2026-01-31T10:00:00Z,
    owner: "developer-team",
    semantic_hash: "auto"
  }
  PurposeStatement {
    narrative: "Validate database writes before execution",
    intent_source: "developer_specification",
    confidence_level: 1.0
  }
  DataSemantics {
    state: {
      query: String,
      table: String,
      approved: Boolean = false
    },
    invariants: [
      "query is not empty",
      "table is not empty"
    ]
  }
  BehavioralSemantics {
    operations: [
      {
        name: "validate_write",
        precondition: "query is not empty",
        parameters: { query: String, table: String },
        postcondition: "approved == true",
        side_effects: ["log_validation"],
        idempotence: "idempotent"
      }
    ]
  }
  ExecutionConstraints {
    trigger_types: ["api_call"],
    resource_limits: {
      max_memory_bytes: 10485760,
      computation_timeout_ms: 5000,
      max_state_size_bytes: 1048576
    },
    external_permissions: [],
    sandbox_mode: "full_isolation"
  }
  HumanMachineContract {
    system_commitments: [
      "All writes are validated before execution",
      "Invalid writes are rejected with explanation"
    ],
    system_refusals: [
      "Will not execute unvalidated writes",
      "Will not bypass validation"
    ],
    user_obligations: [
      "Must provide SQL query and target table"
    ]
  }
}
```

Validate it:

```bash
icl validate db-write.icl
# ✓ db-write.icl is valid

icl verify db-write.icl
# ✓ db-write.icl verified successfully
```

## Next Steps

- See more [example contracts](https://github.com/ICL-System/ICL-Spec/tree/main/examples)
- Read the [full specification](specification/overview.md)
- Explore the [CLI Reference](cli/reference.md)
