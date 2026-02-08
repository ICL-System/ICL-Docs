# Implementation Guide

This guide is for developers who want to contribute to ICL Runtime or create alternative implementations.

## Conformance Requirements

Any ICL implementation **must**:

1. **Parse** all valid ICL contracts per the [BNF grammar](https://github.com/ICL-System/ICL-Spec/blob/main/grammar/icl.bnf)
2. **Reject** all invalid contracts with clear error messages
3. **Normalize** to the same canonical form as the reference implementation
4. **Produce identical semantic hashes** for equivalent contracts
5. **Pass all conformance tests** in [ICL-Spec/conformance/](https://github.com/ICL-System/ICL-Spec/tree/main/conformance)

---

## Determinism Checklist

Any ICL implementation must guarantee determinism. Here's what to check:

- [x] No random number generation (`rand`, `Math.random()`, etc.)
- [x] No direct system time access in core logic
- [x] `BTreeMap` / sorted maps instead of `HashMap` / unordered maps
- [x] IEEE 754 floating point operations only
- [x] No thread-dependent ordering
- [x] No external I/O in core (all I/O at boundaries)
- [x] 100-iteration determinism proof for every test path

---

## Building from Source

### Prerequisites

- Rust 1.93+ (`rustup install stable`)
- For Python binding: Python 3.8+, maturin
- For JS binding: wasm-pack, Node.js 16+
- For Go binding: Go 1.21+, cbindgen

### Build & Test

```bash
git clone https://github.com/ICL-System/ICL-Runtime.git
cd ICL-Runtime
cargo build --workspace
cargo test --workspace
```

### Build All Bindings

```bash
# Python
cd bindings/python && pip install -e . && pytest tests/

# JavaScript
cd bindings/javascript && wasm-pack build --target nodejs && node tests/test_icl.mjs

# Go
cd bindings/go/ffi && cargo build --release
cd bindings/go && LD_LIBRARY_PATH=../../target/release go test -v
```

---

## Project Structure

```
ICL-Runtime/
├── Cargo.toml              # Workspace root
├── crates/
│   ├── icl-core/           # Library crate (all logic)
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs      # Public types + re-exports
│   │       ├── parser/     # Tokenizer + AST + recursive descent
│   │       ├── normalizer.rs
│   │       ├── verifier.rs
│   │       ├── executor.rs
│   │       └── error.rs
│   └── icl-cli/            # Binary crate (CLI)
│       └── src/main.rs
├── bindings/
│   ├── python/             # PyO3 cdylib
│   ├── javascript/         # wasm-bindgen cdylib (excluded from workspace)
│   └── go/                 # cgo + cbindgen
│       ├── ffi/            # Rust FFI layer (staticlib + cdylib)
│       ├── icl.go          # Go wrapper
│       └── icl_test.go
└── tests/
    ├── integration/
    ├── conformance/
    └── determinism/
```

---

## Creating a New Language Binding

All bindings wrap the Rust core — they **must not** reimplement any logic.

### Steps

1. Create a new directory under `bindings/`
2. Add a thin FFI layer that calls `icl-core` functions
3. Expose 5 functions: parse, normalize, verify, execute, semantic_hash
4. Write tests that verify identical output to Rust
5. Add 100-iteration determinism proofs
6. Document installation and usage

### Function Signatures

Every binding should expose these function signatures (adapted to the target language):

```
parse_contract(icl_text: string) -> string   // Returns JSON
normalize(icl_text: string) -> string        // Returns canonical ICL text
verify(icl_text: string) -> string           // Returns JSON diagnostics
execute(icl_text: string, inputs: string) -> string  // Returns JSON result
semantic_hash(icl_text: string) -> string    // Returns hex hash
```

All functions take ICL text as a string and return strings. Errors are either returned as error types (Go, Rust) or raised as exceptions (Python) or thrown (JavaScript).

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Make changes with tests
4. Run the full test suite: `cargo test --workspace`
5. Verify determinism: every new test should include a 100-iteration proof
6. Submit a pull request

See [CONTRIBUTING.md](https://github.com/ICL-System/ICL-Runtime/blob/main/CONTRIBUTING.md) for detailed guidelines.
