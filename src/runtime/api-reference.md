# API Reference

The `icl-core` library exposes a clean Rust API. All language bindings (Python, JS, Go) wrap these same functions.

---

## Core Functions

### `parser::parse_contract`

```rust
pub fn parse_contract(input: &str) -> Result<Contract>
```

Parses ICL text into a `Contract` struct. Returns `Error::ParseError` on invalid syntax.

The parser uses recursive descent and handles all 7 contract sections, type expressions, and error recovery.

### `normalizer::normalize`

```rust
pub fn normalize(icl: &str) -> Result<String>
```

Converts ICL text to canonical form. Returns the normalized string.

Guarantees:
- Idempotent: `normalize(normalize(x)) == normalize(x)`
- Deterministic: same input → same output, always
- Semantic hash is embedded in the `Identity.semantic_hash` field

### `normalizer::semantic_hash`

```rust
pub fn semantic_hash(icl: &str) -> Result<String>
```

Computes the SHA-256 hash of the canonical form. Returns a hex string.

### `verifier::verify_contract`

```rust
pub fn verify_contract(contract: &Contract) -> Result<()>
```

Runs all verification passes: type checking, invariant verification, determinism analysis, and coherence checking. Returns `Ok(())` if the contract passes, or `Error` with diagnostics.

### `executor::execute_contract`

```rust
pub fn execute_contract(contract: &Contract, inputs: &str) -> Result<String>
```

Executes a contract operation in a sandboxed environment. The `inputs` parameter is a JSON string with `"operation"` and `"inputs"` fields. Returns a JSON string with execution results and provenance log.

---

## Core Types

### `Contract`

```rust
pub struct Contract {
    pub identity: Identity,
    pub purpose_statement: PurposeStatement,
    pub data_semantics: DataSemantics,
    pub behavioral_semantics: BehavioralSemantics,
    pub execution_constraints: ExecutionConstraints,
    pub human_machine_contract: HumanMachineContract,
}
```

### `Identity`

```rust
pub struct Identity {
    pub stable_id: String,
    pub version: u32,
    pub created_timestamp: String,  // ISO8601
    pub owner: String,
    pub semantic_hash: String,
}
```

### `PurposeStatement`

```rust
pub struct PurposeStatement {
    pub narrative: String,
    pub intent_source: String,
    pub confidence_level: f64,
}
```

### `DataSemantics`

```rust
pub struct DataSemantics {
    pub state: serde_json::Value,
    pub invariants: Vec<String>,
}
```

### `BehavioralSemantics`

```rust
pub struct BehavioralSemantics {
    pub operations: Vec<Operation>,
}
```

### `Operation`

```rust
pub struct Operation {
    pub name: String,
    pub precondition: String,
    pub parameters: serde_json::Value,
    pub postcondition: String,
    pub side_effects: Vec<String>,
    pub idempotence: String,
}
```

### `ExecutionConstraints`

```rust
pub struct ExecutionConstraints {
    pub trigger_types: Vec<String>,
    pub resource_limits: ResourceLimits,
    pub external_permissions: Vec<String>,
    pub sandbox_mode: String,
}
```

### `ResourceLimits`

```rust
pub struct ResourceLimits {
    pub max_memory_bytes: u64,
    pub computation_timeout_ms: u64,
    pub max_state_size_bytes: u64,
}
```

### `HumanMachineContract`

```rust
pub struct HumanMachineContract {
    pub system_commitments: Vec<String>,
    pub system_refusals: Vec<String>,
    pub user_obligations: Vec<String>,
}
```

### `Error`

```rust
pub enum Error {
    ParseError(String),
    TypeError { expected: String, found: String },
    DeterminismViolation(String),
    ContractViolation { commitment: String, violation: String },
    ValidationError(String),
    ExecutionError(String),
    NormalizationError(String),
}
```

All errors implement `Display` and `std::error::Error`.

---

## Usage Example

```rust
use icl_core::parser::parse_contract;
use icl_core::normalizer;
use icl_core::verifier::verify_contract;
use icl_core::executor::execute_contract;

fn main() -> icl_core::Result<()> {
    let icl_text = std::fs::read_to_string("contract.icl")?;

    // Parse
    let contract = parse_contract(&icl_text)?;

    // Verify
    verify_contract(&contract)?;

    // Normalize
    let canonical = normalizer::normalize(&icl_text)?;
    let hash = normalizer::semantic_hash(&icl_text)?;
    println!("Hash: {}", hash);

    // Execute
    let result = execute_contract(
        &contract,
        r#"{"operation":"echo","inputs":{"message":"Hello"}}"#,
    )?;
    println!("Result: {}", result);

    Ok(())
}
```

---

## Serialization

All core types derive `serde::Serialize` and `serde::Deserialize`, so they can be serialized to/from JSON:

```rust
let contract = parse_contract(&icl_text)?;
let json = serde_json::to_string_pretty(&contract)?;
let roundtrip: Contract = serde_json::from_str(&json)?;
assert_eq!(contract, roundtrip);
```
