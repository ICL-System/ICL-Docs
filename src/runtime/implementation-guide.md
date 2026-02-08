# Implementation Guide

This guide is for developers who want to contribute to ICL Runtime or create alternative implementations.

## Conformance Requirements

Any ICL implementation MUST:

1. **Parse** all valid ICL contracts per the BNF grammar
2. **Reject** all invalid contracts with clear error messages
3. **Normalize** to the same canonical form as the reference implementation
4. **Produce identical semantic hashes** for equivalent contracts
5. **Pass all conformance tests** in [ICL-Spec/conformance/](https://github.com/ICL-System/ICL-Spec/tree/main/conformance)

## Determinism Checklist

- [ ] No `rand` crate or random number generation
- [ ] No direct system time access in core logic
- [ ] `BTreeMap` instead of `HashMap` everywhere
- [ ] IEEE 754 floating point operations only
- [ ] No thread-dependent ordering
- [ ] No external I/O in core (all I/O at boundaries)
- [ ] 100-iteration determinism proof for every test

## Building from Source

```bash
git clone https://github.com/ICL-System/ICL-Runtime.git
cd ICL-Runtime
cargo build
cargo test
```

## Running Conformance Tests

```bash
# Once implemented (Phase 6):
cargo test --test conformance
```

Conformance test fixtures are in [ICL-Spec/conformance/](https://github.com/ICL-System/ICL-Spec/tree/main/conformance).

## Creating a New Language Binding

All bindings wrap the Rust core — they MUST NOT reimplement any logic:

1. **Python**: Use PyO3 + maturin in `bindings/python/`
2. **JavaScript**: Use wasm-pack in `bindings/javascript/`
3. **Go**: Use cgo + cbindgen in `bindings/go/`

Each binding should:
- Expose the same public API as the Rust core
- Pass the same conformance test suite
- Add no additional logic beyond type conversion
