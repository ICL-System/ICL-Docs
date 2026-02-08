# CLI Reference

> **Status**: All commands are scaffolded but print "not yet implemented". Implementation starts in Phase 4.

## Installation

```bash
# From source
cargo install --path crates/icl-cli

# Or build locally
cargo build -p icl-cli
./target/debug/icl --help
```

## Commands

### `icl validate <file>`
Check syntax and types.
```bash
icl validate contract.icl
icl validate contract.icl --json
```

### `icl normalize <file>`
Output canonical form.
```bash
icl normalize contract.icl > canonical.icl
```

### `icl verify <file>`
Full verification (types + invariants + determinism + coherence).
```bash
icl verify contract.icl
icl verify contract.icl --json
```

### `icl fmt <file>`
Format to standard style.
```bash
icl fmt contract.icl
```

### `icl hash <file>`
Compute SHA-256 semantic hash.
```bash
icl hash contract.icl
# → sha256:a1b2c3d4e5f6...
```

### `icl diff <file_a> <file_b>`
Semantic diff between two contracts.
```bash
icl diff v1.icl v2.icl
```

### `icl init [name]`
Scaffold a new contract.
```bash
icl init my-contract
# Creates my-contract.icl with template
```

### `icl execute <file> --input <json>`
Run a contract with inputs.
```bash
icl execute contract.icl --input '{"key": "value"}'
icl execute contract.icl --input '{}' --json
```

### `icl version`
Show version info.
```bash
icl version
# icl 0.1.0 (icl-core 0.1.0)
# Status: Early Development — Phase 0
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Validation/verification failure or not implemented |
| 2 | File not found or I/O error |
