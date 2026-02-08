# API Reference

> **Status**: The API is scaffolded but all functions are stubs. This documents the intended interface.

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

## Functions

### `parser::parse_contract`

```rust
pub fn parse_contract(input: &str) -> Result<Contract>
```

Parses ICL text into a `Contract` struct. Returns `ParseError` on invalid syntax.

### `normalizer::normalize`

```rust
pub fn normalize(icl: &str) -> Result<String>
```

Converts ICL text to canonical form. Idempotent: `normalize(normalize(x)) == normalize(x)`.

### `verifier::verify_contract`

```rust
pub fn verify_contract(contract: &Contract) -> Result<()>
```

Checks type consistency, invariant satisfaction, determinism, and coherence.

### `executor::execute_contract`

```rust
pub fn execute_contract(contract: &Contract, inputs: &str) -> Result<String>
```

Runs a contract with given inputs in a sandboxed environment.
