# Language Bindings

All language bindings wrap the same Rust core (`icl-core`). They never reimplement any logic — only type conversion and FFI glue.

Every binding exposes the same 5 functions:

| Function | Python | JavaScript | Go |
|----------|--------|------------|-----|
| Parse | `parse_contract(text)` | `parseContract(text)` | `ParseContract(text)` |
| Normalize | `normalize(text)` | `normalize(text)` | `Normalize(text)` |
| Verify | `verify(text)` | `verify(text)` | `Verify(text)` |
| Execute | `execute(text, input)` | `execute(text, input)` | `Execute(text, input)` |
| Hash | `semantic_hash(text)` | `semanticHash(text)` | `SemanticHash(text)` |

---

## Python (PyO3 + maturin)

**Package:** `icl-runtime` on [test.pypi.org](https://test.pypi.org/project/icl-runtime/)

### Install

```bash
# From test.pypi.org (current)
pip install --index-url https://test.pypi.org/simple/ icl-runtime

# From local build
cd ICL-Runtime/bindings/python
pip install -e .
```

### Usage

```python
import icl

# Parse a contract
result = icl.parse_contract("""
Contract {
  Identity { stable_id: "test", version: 1, ... }
  ...
}
""")
print(result)  # JSON string

# Normalize to canonical form
canonical = icl.normalize(contract_text)

# Verify (returns JSON diagnostics)
diagnostics = icl.verify(contract_text)

# Execute with inputs
output = icl.execute(contract_text, '{"operation":"echo","inputs":{"message":"Hi"}}')

# Compute semantic hash
hash_value = icl.semantic_hash(contract_text)
print(hash_value)  # "1f7dcf67..."
```

### Type Stubs

Python type stubs (`.pyi`) are included for IDE autocompletion:

```python
def parse_contract(icl_text: str) -> str: ...
def normalize(icl_text: str) -> str: ...
def verify(icl_text: str) -> str: ...
def execute(icl_text: str, inputs_json: str) -> str: ...
def semantic_hash(icl_text: str) -> str: ...
```

---

## JavaScript / TypeScript (wasm-pack)

**Package:** `icl-wasm`

### Install

```bash
cd ICL-Runtime/bindings/javascript
wasm-pack build --target nodejs
```

The built package is in `pkg/`.

### Usage (Node.js)

```javascript
const icl = require('./pkg/icl_wasm.js');

// Parse
const parsed = icl.parseContract(contractText);

// Normalize
const canonical = icl.normalize(contractText);

// Verify
const diagnostics = icl.verify(contractText);

// Execute
const result = icl.execute(contractText, '{"operation":"echo","inputs":{"message":"Hi"}}');

// Hash
const hash = icl.semanticHash(contractText);
```

### TypeScript

TypeScript definitions (`.d.ts`) are generated automatically:

```typescript
export function parseContract(icl_text: string): string;
export function normalize(icl_text: string): string;
export function verify(icl_text: string): string;
export function execute(icl_text: string, inputs_json: string): string;
export function semanticHash(icl_text: string): string;
```

### Browser Usage

Build for the browser with:

```bash
wasm-pack build --target web
```

---

## Go (cgo + cbindgen)

**Module:** `github.com/ICL-System/ICL-Runtime/bindings/go`

### Install

The Go binding requires the Rust FFI library to be built first:

```bash
cd ICL-Runtime
cargo build --release -p icl-ffi
```

### Usage

```go
package main

import (
    icl "github.com/ICL-System/ICL-Runtime/bindings/go"
    "fmt"
    "log"
)

func main() {
    contract := `Contract { ... }`

    // Parse
    result, err := icl.ParseContract(contract)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(result)

    // Normalize
    canonical, err := icl.Normalize(contract)
    if err != nil { log.Fatal(err) }

    // Verify
    diagnostics, err := icl.Verify(contract)
    if err != nil { log.Fatal(err) }

    // Execute
    output, err := icl.Execute(contract, `{"operation":"echo","inputs":{"message":"Hi"}}`)
    if err != nil { log.Fatal(err) }

    // Hash
    hash, err := icl.SemanticHash(contract)
    if err != nil { log.Fatal(err) }

    fmt.Println(canonical, diagnostics, output, hash)
}
```

### Running Tests

```bash
cd ICL-Runtime/bindings/go
LD_LIBRARY_PATH=../../target/release go test -v
```

---

## Rust (Native)

Use `icl-core` directly as a Rust dependency:

```toml
# Cargo.toml
[dependencies]
icl-core = { path = "crates/icl-core" }
```

```rust
use icl_core::parser::parse_contract;
use icl_core::normalizer;
use icl_core::verifier::verify_contract;
use icl_core::executor::execute_contract;

let contract = parse_contract(&icl_text)?;
verify_contract(&contract)?;
let canonical = normalizer::normalize(&icl_text)?;
let hash = normalizer::semantic_hash(&icl_text)?;
```

---

## Cross-Language Determinism

All bindings are tested with 100-iteration determinism proofs. Given the same ICL text:

- Python `normalize()` == Rust `normalize()` == JS `normalize()` == Go `Normalize()`
- Python `semantic_hash()` == Rust `semantic_hash()` == JS `semanticHash()` == Go `SemanticHash()`
- Python `execute()` == Rust `execute_contract()` == JS `execute()` == Go `Execute()`

This is guaranteed because all bindings call through to the same compiled Rust code.
