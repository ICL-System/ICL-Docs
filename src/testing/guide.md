# Testing Guide

## Test Categories

ICL Runtime has three categories of tests:

### 1. Unit Tests (in-crate)
Located alongside the source code in `#[cfg(test)]` modules.

```bash
cargo test                      # Run all
cargo test -p icl-core          # Core library only
cargo test -p icl-core parser   # Parser tests only
```

### 2. Integration Tests
Located in `tests/integration/`. Test cross-module behavior.

```bash
cargo test --test '*'    # All integration tests
```

### 3. Conformance Tests
Located in `tests/conformance/`. Validate against the ICL specification using fixtures from [ICL-Spec/conformance/](https://github.com/ICL-System/ICL-Spec/tree/main/conformance).

```bash
cargo test --test conformance
```

### 4. Determinism Tests
Located in `tests/determinism/`. Every test runs 100 iterations and verifies identical output.

```bash
cargo test --test determinism
```

## Writing Tests

### Rule: Every Test Must Prove Determinism

```rust
#[test]
fn test_my_feature() {
    let input = "...";
    let first_result = my_function(input);

    // 100-iteration determinism proof
    for _ in 0..100 {
        assert_eq!(my_function(input), first_result);
    }
}
```

### Rule: No Randomness, No System Time

Tests must not depend on:
- Random number generators
- Current system time
- File system ordering
- Thread scheduling

### Rule: Conformance Fixtures

When adding parser/normalizer features, add corresponding fixtures to ICL-Spec:
- `conformance/valid/` — contracts that MUST parse
- `conformance/invalid/` — contracts that MUST be rejected
- `conformance/normalization/` — input→canonical pairs

## Benchmarks

Performance benchmarks use [Criterion](https://bheisler.github.io/criterion.rs/book/):

```bash
cargo bench                    # Run all benchmarks
cargo bench --bench determinism  # Determinism benchmarks only
```
