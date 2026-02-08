# Integration Overview

> **Status**: Integrations are not yet available. This documents the planned integration points.

## Language Bindings

ICL Runtime compiles to multiple targets from a single Rust codebase:

### Python
```python
# pip install icl-runtime  (Phase 7)
from icl_runtime import validate, verify, normalize

result = validate("path/to/contract.icl")
```

### JavaScript / Node.js
```javascript
// npm install icl-runtime  (Phase 7)
import { validate, verify, normalize } from 'icl-runtime';

const result = await validate('path/to/contract.icl');
```

### Go
```go
// go get github.com/ICL-System/ICL-Runtime/bindings/go  (Phase 7)
import "github.com/ICL-System/ICL-Runtime/bindings/go/icl"

result, err := icl.Validate("path/to/contract.icl")
```

### Rust (native)
```rust
// cargo add icl-core
use icl_core::parser::parse_contract;
use icl_core::verifier::verify_contract;

let contract = parse_contract(icl_text)?;
verify_contract(&contract)?;
```

## Editor Support (Planned)

- VS Code extension with syntax highlighting
- LSP server for inline diagnostics
- `.icl` file type recognition

## CI/CD Integration (Planned)

```yaml
# GitHub Actions example
- name: Validate ICL contracts
  run: icl validate contracts/*.icl

- name: Verify ICL contracts
  run: icl verify contracts/*.icl --json
```
