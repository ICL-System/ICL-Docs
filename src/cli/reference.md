# CLI Reference

The ICL CLI provides 9 commands for working with intent contracts.

## Installation

```bash
# From source
git clone https://github.com/ICL-System/ICL-Runtime.git
cd ICL-Runtime
cargo install --path crates/icl-cli
```

## Global Options

| Option | Description |
|--------|-------------|
| `--quiet` | Suppress non-error output (for CI usage) |
| `-h, --help` | Print help |
| `-V, --version` | Print version |

---

## `icl validate <file>`

Check syntax and types. Parses the contract and runs the verifier.

```bash
icl validate contract.icl
# ✓ contract.icl is valid

icl validate contract.icl --json
# {"file":"contract.icl","valid":true,"errors":0,"warnings":2,"diagnostics":[...]}
```

**Options:**

| Option | Description |
|--------|-------------|
| `--json` | Output results as JSON |
| `--quiet` | Exit code only, no output |

**Exit codes:** `0` = valid, `1` = invalid, `2` = file error

---

## `icl normalize <file>`

Output the canonical form. Sections are sorted alphabetically, fields are sorted within sections, comments are stripped, and the semantic hash is recomputed.

```bash
icl normalize contract.icl
# Prints canonical form to stdout
```

Normalization is idempotent: `normalize(normalize(x)) == normalize(x)`.

---

## `icl verify <file>`

Full verification — type checking, invariant consistency, determinism analysis, and coherence verification.

```bash
icl verify contract.icl
# ✓ contract.icl verified successfully

icl verify contract.icl --json
# {"file":"contract.icl","verified":true,...}
```

**Options:**

| Option | Description |
|--------|-------------|
| `--json` | Output results as JSON |

The verifier checks:
- All types are valid and consistent
- Invariants are satisfiable and non-contradictory
- Operations preserve invariants
- No randomness, system time, or external I/O in core logic
- Preconditions and postconditions are coherent
- Resource limits are feasible

---

## `icl fmt <file>`

Format a contract to standard style (equivalent to normalize, written back to file or stdout).

```bash
icl fmt contract.icl
# Prints formatted contract
```

---

## `icl hash <file>`

Compute the SHA-256 semantic hash of a contract.

```bash
icl hash contract.icl
# 1f7dcf67d92b813f3cc0402781f023ea33c76dd7c2b6963531fe68bf9c032cb8
```

The hash is computed from the canonical (normalized) form. Two contracts with the same semantics produce the same hash, regardless of formatting or comments.

---

## `icl diff <file_a> <file_b>`

Semantic diff between two contracts. Both contracts are normalized first, then compared field by field.

```bash
icl diff v1.icl v2.icl
# --- v1.icl (canonical)
# +++ v2.icl (canonical)
#   Contract {
#     Identity {
# -     stable_id: "ic-v1-001",
# +     stable_id: "ic-v2-001",
#       ...
```

---

## `icl init [name]`

Scaffold a new ICL contract with all required sections.

```bash
icl init my-contract
# ✓ created my-contract.icl

icl init
# ✓ created my-contract.icl  (default name)
```

The generated template includes all required sections with placeholder values — ready to fill in.

---

## `icl execute <file> --input <json>`

Run a contract with JSON inputs in a sandboxed environment.

```bash
icl execute contract.icl --input '{"operation":"echo","inputs":{"message":"Hello"}}'
# ✓ contract.icl executed successfully
#   Operations: 1
#   Provenance entries: 1
```

**Options:**

| Option | Description |
|--------|-------------|
| `--input <json>` | JSON input with `operation` and `inputs` fields (required) |
| `--json` | Output execution result as JSON |

The input JSON must contain:
- `"operation"` — name of the operation to execute
- `"inputs"` — parameters matching the operation's parameter spec

Execution enforces resource limits (memory, time, state size) and logs all state changes in the provenance log.

---

## `icl version`

Show version information.

```bash
icl version
# icl 0.1.0 (icl-core 0.1.0)
# Phases complete: Parser, Normalizer, Verifier, CLI
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Validation or verification failure |
| 2 | File not found, I/O error, or execution error |

---

## CI Integration

Use `--quiet` and `--json` for CI pipelines:

```bash
# Fail the build if any contract is invalid
icl validate contracts/*.icl --quiet

# Get machine-readable results
icl verify contract.icl --json | jq '.verified'
```
