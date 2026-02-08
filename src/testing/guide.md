# Testing Guide

ICL has 268 tests across Rust, Python, JavaScript, and Go. Every test proves determinism.

## Running Tests

### Rust (core + CLI)

```bash
cd ICL-Runtime
cargo test --workspace
# 175 tests passing
```

### Python

```bash
cd ICL-Runtime/bindings/python
pip install -e .
pytest tests/
# 18 tests passing
```

### JavaScript

```bash
cd ICL-Runtime/bindings/javascript
wasm-pack build --target nodejs
node tests/test_icl.mjs
# 31 tests passing
```

### Go

```bash
cd ICL-Runtime/bindings/go
LD_LIBRARY_PATH=../../target/release go test -v
# 16 tests passing
```

---

## Test Categories

### 1. Unit Tests (in-crate)

Located alongside source code in `#[cfg(test)]` modules. Each module has its own tests.

```bash
cargo test -p icl-core              # All core tests
cargo test -p icl-core parser       # Parser tests only
cargo test -p icl-core normalizer   # Normalizer tests only
cargo test -p icl-core verifier     # Verifier tests only
cargo test -p icl-core executor     # Executor tests only
```

### 2. Integration Tests

Located in `tests/integration/`. Test cross-module behavior — parsing a contract, then normalizing, verifying, and executing it.

```bash
cargo test --test '*'
```

### 3. Conformance Tests

Located in `tests/conformance/`. Validate the runtime against the [ICL-Spec conformance suite](https://github.com/ICL-System/ICL-Spec/tree/main/conformance).

- `conformance/valid/` — contracts that must parse successfully
- `conformance/invalid/` — contracts that must be rejected with clear errors
- `conformance/normalization/` — input/expected pairs for canonical form

```bash
cargo test conformance
```

### 4. Determinism Tests

Located in `tests/determinism/`. Every test runs 100 iterations and verifies identical output.

```bash
cargo test determinism
```

---

## Writing Tests

### Rule 1: Every Test Must Prove Determinism

```rust
#[test]
fn test_my_feature() {
    let input = "...";
    let first_result = my_function(input);

    for _ in 0..100 {
        assert_eq!(my_function(input), first_result);
    }
}
```

### Rule 2: No Randomness, No System Time

Tests must not depend on:
- Random number generators
- Current system time
- File system ordering
- Thread scheduling

### Rule 3: Test Both Valid and Invalid Inputs

```rust
#[test]
fn test_rejects_invalid_confidence() {
    let input = r#"Contract {
      ...
      PurposeStatement { confidence_level: 1.5 }
      ...
    }"#;
    assert!(parse_contract(input).is_err());
}
```

### Rule 4: Conformance Fixtures

When adding parser/normalizer features, add matching fixtures to ICL-Spec:
- `conformance/valid/` — must parse
- `conformance/invalid/` — must be rejected
- `conformance/normalization/` — input→canonical pairs

---

## Cross-Language Determinism

All language bindings must produce identical results to the Rust core. The binding test suites include 100-iteration determinism proofs:

**Python:**
```python
def test_normalize_determinism():
    first = icl.normalize(contract_text)
    for _ in range(100):
        assert icl.normalize(contract_text) == first
```

**JavaScript:**
```javascript
const first = icl.normalize(contractText);
for (let i = 0; i < 100; i++) {
    assert.strictEqual(icl.normalize(contractText), first);
}
```

**Go:**
```go
first, _ := icl.Normalize(contract)
for i := 0; i < 100; i++ {
    result, _ := icl.Normalize(contract)
    if result != first {
        t.Fatalf("Non-determinism at iteration %d", i)
    }
}
```

---

## Test Counts by Component

| Component | Tests | Includes Determinism Proof |
|-----------|-------|---------------------------|
| Parser (tokenizer + AST + recursive descent) | ~80 | Yes |
| Normalizer (canonical form + hashing) | ~30 | Yes (idempotence + 100-iter) |
| Verifier (types + invariants + determinism + coherence) | ~40 | Yes |
| Executor (sandbox + provenance) | ~20 | Yes |
| CLI (all 9 commands) | ~15 | — |
| Python binding | 18 | Yes (100-iter) |
| JavaScript binding | 31 | Yes (100-iter) |
| Go binding | 16 | Yes (100-iter) |
| **Total** | **~268** | |
