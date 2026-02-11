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

**Package:** `icl-runtime` on [PyPI](https://pypi.org/project/icl-runtime/)

### Install

```bash
pip install icl-runtime
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

**Package:** `icl-runtime` on [npm](https://www.npmjs.com/package/icl-runtime)

Works out of the box in **Node.js**, **bundlers** (Vite, Webpack, Rollup), and **plain browsers** — zero manual WASM copying. The correct target is selected automatically via [conditional exports](https://nodejs.org/api/packages.html#conditional-exports).

### Install

```bash
npm install icl-runtime
```

### Usage (Node.js — CommonJS)

```javascript
const icl = require('icl-runtime');

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

### Usage (Node.js — ES Modules)

```javascript
import { parseContract, normalize, verify, execute, semanticHash } from 'icl-runtime';

const ast = JSON.parse(parseContract(contractText));
const hash = semanticHash(contractText);
```

### Usage (Bundlers — Vite, Webpack, Rollup)

```javascript
import { parseContract, normalize, verify, execute, semanticHash } from 'icl-runtime';

// Same API — bundler target is selected automatically via "exports" → "import"
const ast = JSON.parse(parseContract(contractText));
```

> **Vite users**: Add `vite-plugin-wasm` to your Vite config for WASM ESM integration support.

### Usage (Browser — Script Tag)

```html
<script type="module">
  import init, { parseContract } from 'icl-runtime/web';

  // Must call init() first — loads the WASM binary via fetch()
  await init();

  const ast = JSON.parse(parseContract(contractText));
  console.log(ast);
</script>
```

### TypeScript

TypeScript definitions (`.d.ts`) are included — no `@types` package needed:

```typescript
export function parseContract(icl_text: string): string;
export function normalize(icl_text: string): string;
export function verify(icl_text: string): string;
export function execute(icl_text: string, inputs_json: string): string;
export function semanticHash(icl_text: string): string;
```

### How Target Selection Works

The npm package includes 3 pre-built WASM targets:

| Target | Loaded When | WASM Loading |
|--------|-------------|--------------|
| `dist/nodejs/` | `require()` or `import` in Node.js | `fs.readFileSync` (sync) |
| `dist/bundler/` | `import` in Vite/Webpack/Rollup | Bundler handles `.wasm` |
| `dist/web/` | `import from 'icl-runtime/web'` | `fetch()` + `WebAssembly.instantiate` |

No manual copying of `.wasm` files is needed — each target's glue code loads the WASM binary from the correct relative path.

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
icl-core = "0.1"
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
