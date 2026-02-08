# Architecture

The ICL Runtime processes contracts through a four-stage pipeline.

## Pipeline

```
ICL Text → Parser → AST → Normalizer → Canonical Form
                            ↓
                         Verifier → Type Check + Invariants + Determinism + Coherence
                            ↓
                         Executor → Sandboxed Execution + Provenance Logging
```

Each stage is independent and can be used standalone.

---

## Components

### Parser (`icl-core::parser`)

Converts ICL text into a typed Abstract Syntax Tree (AST).

- **Tokenizer** (`parser::tokenizer`): Character-by-character scanning produces a stream of typed tokens — keywords, identifiers, strings, integers, floats, ISO8601 timestamps, UUIDs, operators, and delimiters. Reports errors with line/column numbers.
- **AST** (`parser::ast`): Defines all AST node types matching the BNF grammar — `ContractNode`, `IdentityNode`, `TypeExpression`, `OperationNode`, etc.
- **Parser** (`parser::parse_contract`): Recursive descent parser. Handles all 7 contract sections, type expressions (primitives, composites, collections), and error recovery.

### Normalizer (`icl-core::normalizer`)

Transforms contracts into a deterministic canonical form.

- Sorts sections alphabetically
- Sorts fields within each section
- Strips all comments and normalizes whitespace
- Expands type shorthands to full forms
- Fills in defaults
- Computes SHA-256 semantic hash of the canonical output
- Guarantees idempotence: `normalize(normalize(x)) == normalize(x)`

### Verifier (`icl-core::verifier`)

Validates contracts against the ICL specification. Four verification passes:

1. **Type Checker** — Validates all types (primitives, composites, collections), checks parameter/postcondition consistency, enforces no implicit coercion
2. **Invariant Verifier** — Checks invariants are satisfiable, consistent (no contradictions), and preserved by all operations
3. **Determinism Checker** — Detects randomness, system time access, external I/O, floating-point non-determinism, and hash iteration order dependencies
4. **Coherence Verifier** — Checks precondition/postcondition consistency, detects circular dependencies, verifies resource limits are feasible

### Executor (`icl-core::executor`)

Runs contracts in a sandboxed environment:

- Evaluates preconditions before execution
- Applies operation with given inputs
- Verifies postconditions after execution
- Enforces resource limits (memory bytes, computation timeout, state size)
- Records all state changes in a provenance log
- No WASM needed — ICL is declarative, execution is simulated

---

## Crate Structure

```
ICL-Runtime/
├── crates/
│   ├── icl-core/        # Library crate — all logic
│   │   └── src/
│   │       ├── lib.rs           # Public types (Contract, Identity, etc.)
│   │       ├── parser/
│   │       │   ├── mod.rs       # parse_contract()
│   │       │   ├── tokenizer.rs # Token scanning
│   │       │   └── ast.rs       # AST node types
│   │       ├── normalizer.rs    # normalize()
│   │       ├── verifier.rs      # verify_contract()
│   │       ├── executor.rs      # execute_contract()
│   │       └── error.rs         # Error types
│   └── icl-cli/         # Binary crate — CLI interface
│       └── src/
│           └── main.rs          # clap commands
├── bindings/
│   ├── python/          # PyO3 + maturin
│   ├── javascript/      # wasm-bindgen + wasm-pack
│   └── go/              # cgo + cbindgen
└── tests/
    ├── integration/     # Cross-module tests
    ├── conformance/     # Spec compliance
    └── determinism/     # 100-iteration proofs
```

---

## Determinism Architecture

Determinism is enforced at every level:

| Guarantee | How |
|-----------|-----|
| No randomness | No `rand` crate, no UUID generation at runtime |
| No system time | All timestamps are inputs, never generated |
| Ordered collections | `BTreeMap` everywhere, never `HashMap` |
| Stable floating point | IEEE 754 strict semantics |
| Deterministic ordering | All iteration is sorted |
| 100-iteration proof | Every test runs 100 times, all outputs must match byte-for-byte |

---

## Language Bindings Architecture

All bindings wrap the same Rust core — they never reimplement logic:

```
                    ┌──────────────┐
                    │   icl-core   │  ← Single Rust library
                    └──────┬───────┘
            ┌──────────────┼──────────────┐
            │              │              │
     ┌──────┴──────┐ ┌────┴────┐ ┌───────┴──────┐
     │ PyO3 (FFI)  │ │  WASM   │ │  cgo (FFI)   │
     │  maturin    │ │wasm-pack│ │  cbindgen    │
     └──────┬──────┘ └────┬────┘ └───────┬──────┘
            │              │              │
     ┌──────┴──────┐ ┌────┴────┐ ┌───────┴──────┐
     │   Python    │ │  JS/TS  │ │     Go       │
     │ pip install │ │npm inst.│ │   go get     │
     └─────────────┘ └─────────┘ └──────────────┘
```

| Language | Technology | Package Name |
|----------|-----------|--------------|
| Python | PyO3 + maturin | `icl-runtime` on [PyPI](https://pypi.org/project/icl-runtime/) (published on `v*` tag) |
| JavaScript | wasm-bindgen + wasm-pack | `icl-runtime` on [npm](https://www.npmjs.com/package/icl-runtime) |
| Go | cgo + cbindgen | `github.com/ICL-System/ICL-Runtime/bindings/go` |

All three expose the same 5 functions: parse, normalize, verify, execute, semantic hash.
