# Runtime Architecture

The ICL Runtime uses a pipeline architecture:

```
ICL Text → Parser → AST → Normalizer → Canonical Form
                           ↓
                        Verifier → Type Check + Invariants + Determinism
                           ↓
                        Executor → Sandboxed Execution
```

## Components

### Parser (`icl-core::parser`)
Converts ICL text into an Abstract Syntax Tree (AST).
- **Tokenizer**: Character-by-character scanning → token stream
- **Parser**: Recursive descent → AST nodes
- **Status**: Not yet implemented (Phase 1)

### Normalizer (`icl-core::normalizer`)
Transforms contracts into canonical form for hashing and comparison.
- Section sorting (alphabetical)
- Field sorting within sections
- Whitespace normalization
- Comment removal
- SHA-256 semantic hashing
- **Status**: Stub — passes input through (Phase 2)

### Verifier (`icl-core::verifier`)
Validates contracts against the specification.
- Type checking
- Invariant consistency
- Determinism verification
- Precondition/postcondition coherence
- Resource limit feasibility
- **Status**: Stub — returns Ok for anything (Phase 3)

### Executor (`icl-core::executor`)
Runs contracts in a sandboxed environment.
- Precondition evaluation
- WASM sandbox isolation
- Postcondition verification
- Resource limit enforcement
- Provenance logging
- **Status**: Stub — returns empty string (Phase 5)

## Crate Structure

```
crates/
├── icl-core/    # Library crate (all logic)
└── icl-cli/     # Binary crate (CLI interface)
```

## Determinism Architecture

The runtime enforces determinism at every level:
- **No randomness**: No `rand`, no UUIDs generated at runtime
- **No system time**: Timestamps are inputs, never generated
- **Ordered collections**: `BTreeMap` everywhere, never `HashMap`
- **IEEE 754**: Strict floating point semantics
- **100-iteration proof**: Every test runs 100 times, all outputs must match

## Language Bindings

All bindings wrap the same Rust core:

| Language | Technology | Package |
|----------|-----------|---------|
| Python | PyO3 + maturin | `pip install icl-runtime` |
| JavaScript | wasm-pack (WASM) | `npm install icl-runtime` |
| Go | cgo + cbindgen | `go get github.com/ICL-System/ICL-Runtime/bindings/go` |
