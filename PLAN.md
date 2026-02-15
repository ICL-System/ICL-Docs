# ICL Project — Master Plan

**Date:** 2026-02-08
**Organization:** [ICL-System](https://github.com/ICL-System)
**Goal:** Make ICL the universal standard for intent contracts across all AI systems.

---

## Vision

Intent Contract Language (ICL) is a formal, deterministic, language-agnostic specification language for intent contracts — like OpenAPI but for human intent and AI agent constraints. It will be adopted everywhere: AI agents, dev tools, robotics, workflow platforms, and more.

---

## Organization Structure

### GitHub: `ICL-System/`

| Repo | Purpose | Audience |
|------|---------|----------|
| **ICL-Spec** | The standard itself (spec + grammar + conformance tests + examples) | Standards bodies, alternative implementers, researchers |
| **ICL-Runtime** | Canonical Rust implementation + CLI + all language bindings | Developers using ICL |
| **ICL-Docs** | Documentation website (mdBook → iclstandard.org) | Everyone |

### Why 3 Repos?

- **Spec is separate from implementation** — required for standardization (IETF/W3C). Alt implementations reference the spec, not Rust code.
- **Runtime includes ALL bindings** — one codebase, one bug fix, all languages. No version drift.
- **Docs are deployable** — separate build/deploy pipeline for the website.

---

## Repository Architectures

### 1. ICL-Spec (NEW — to be created)

```
ICL-Spec/
├── spec/
│   └── CORE-SPECIFICATION.md        # The formal spec (BNF, type system, semantics)
├── grammar/
│   └── icl.bnf                       # Standalone formal grammar file
├── examples/
│   ├── db-write-validation.icl       # Reference example contracts
│   ├── api-rate-limiting.icl
│   ├── agent-action-verification.icl
│   ├── hello-world.icl
│   └── code-verification.icl
├── conformance/
│   ├── valid/                        # .icl files that MUST parse successfully
│   ├── invalid/                      # .icl files that MUST fail
│   └── normalization/                # input→expected canonical output pairs
├── roadmap/
│   └── standardization-roadmap.md
├── CHANGELOG.md
├── LICENSE                           # MIT
└── README.md
```

**Key principle:** This repo has ZERO code. It's the pure standard.

### 2. ICL-Runtime (RESTRUCTURED)

```
ICL-Runtime/
├── crates/
│   ├── icl-core/                     # Library crate — THE runtime
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs                # Public API
│   │       ├── parser/
│   │       │   ├── mod.rs
│   │       │   ├── tokenizer.rs      # BNF tokenization
│   │       │   └── ast.rs            # AST types
│   │       ├── normalizer.rs         # Canonical form
│   │       ├── verifier.rs           # Type checking + invariants + determinism
│   │       ├── executor.rs           # Sandboxed execution (pure Rust)
│   │       └── error.rs              # Error types
│   └── icl-cli/                      # Binary crate — CLI tool
│       ├── Cargo.toml
│       └── src/
│           └── main.rs
├── bindings/
│   ├── python/                       # PyO3 binding (pip: icl-runtime)
│   ├── javascript/                   # WASM binding (npm: icl-runtime)
│   └── go/                           # cgo binding
│       └── ffi/                      # Rust shared library for cgo
├── tests/
│   ├── integration/                  # End-to-end runtime tests
│   ├── conformance/                  # Tests against ICL-Spec conformance suite
│   └── determinism/                  # 100+ iteration determinism proofs
├── benches/
│   └── determinism.rs                # Performance benchmarks
├── Cargo.toml                        # Workspace manifest
├── CONTRIBUTING.md
├── CHANGELOG.md
├── LICENSE
└── README.md
```

**Key principle:** One Rust codebase compiles to all targets. Bindings are thin glue.

#### How One Runtime Serves All Languages

```
                    ┌─────────────────────────┐
                    │   icl-core (Rust lib)    │
                    │  parser │ normalizer     │
                    │  verifier │ executor     │
                    └──────────┬──────────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
         PyO3/maturin     wasm-pack          cgo
              │                │                │
         ┌────▼────┐    ┌─────▼─────┐    ┌────▼────┐
         │  PyPI   │    │    npm    │    │  go get │
         │   pip   │    │   wasm   │    │   mod   │
         └─────────┘    └──────────┘    └─────────┘
```

- Bug fix in `icl-core` → rebuild → all packages updated
- One test suite → all bindings verified
- One CI pipeline → publishes to PyPI + npm + crates.io simultaneously
- All packages always same version

#### CLI Commands

```bash
icl-cli init                            # Scaffold a new .icl contract
icl-cli validate <file.icl>             # Check syntax + types
icl-cli normalize <file.icl>            # Output canonical form
icl-cli verify <file.icl>               # Full verification (invariants, determinism)
icl-cli execute <file.icl> --input '{}' # Run contract with inputs
icl-cli hash <file.icl>                 # Compute semantic hash (SHA-256)
icl-cli diff <a.icl> <b.icl>            # Semantic diff between contracts
icl-cli fmt <file.icl>                  # Format to standard style
icl-cli version                         # Show version info
```

### 3. ICL-Docs (RESTRUCTURED)

```
ICL-Docs/
├── book.toml                         # mdBook configuration
├── src/
│   ├── SUMMARY.md                    # Table of contents
│   ├── introduction.md
│   ├── getting-started.md
│   ├── writing-contracts.md
│   ├── specification/
│   │   └── overview.md
│   ├── runtime/
│   │   ├── architecture.md
│   │   ├── api-reference.md
│   │   └── implementation-guide.md
│   ├── cli/
│   │   └── reference.md
│   ├── testing/
│   │   └── guide.md
│   └── integrations/
│       └── overview.md
├── LICENSE
└── README.md
```

**Key principle:** Builds to a static site deployable to `iclstandard.org` or GitHub Pages.

---

## What Moves Where

### From current ICL-Runtime → ICL-Spec (new repo)

| File | Destination |
|------|-------------|
| `spec/CORE-SPECIFICATION.md` | `ICL-Spec/spec/CORE-SPECIFICATION.md` |
| `docs/example-contracts.md` | Split into individual `.icl` files in `ICL-Spec/examples/` |
| `docs/standardization-roadmap.md` | `ICL-Spec/roadmap/standardization-roadmap.md` |

### From current ICL-Runtime → ICL-Docs

| File | Destination |
|------|-------------|
| `docs/getting-started.md` | `ICL-Docs/src/getting-started.md` |
| `docs/api-reference.md` | `ICL-Docs/src/api-reference.md` |
| `docs/implementation-guide.md` | `ICL-Docs/src/implementation-guide.md` |
| `docs/runtime-architecture.md` | `ICL-Docs/src/runtime-architecture.md` |
| `docs/testing.md` | `ICL-Docs/src/testing.md` |

### Stays in ICL-Runtime (restructured)

| File | New location |
|------|-------------|
| `runtime/src/*.rs` | `crates/icl-core/src/` |
| `runtime/Cargo.toml` | `crates/icl-core/Cargo.toml` |
| `CONTRIBUTING.md` | Stays at root (updated) |

### Deleted (during Phase 0, then recreated)

| What | Status |
|------|--------|
| `bindings/python/` (empty stubs) | Recreated with actual PyO3 code in Phase 6 |
| `bindings/javascript/` (empty stubs) | Recreated with actual wasm-pack code in Phase 6 |
| `bindings/go/` (empty stubs) | Recreated with actual cgo code in Phase 6 |
| `examples/` (empty) | Examples live in ICL-Spec |
| `tests/` (empty) | Recreated with actual tests |
| `docs/` (entire folder) | Content moved to ICL-Docs and ICL-Spec |

---

## Honesty Policy

The README and all documentation must reflect **actual state**, not aspirations:

- Status badges showing "Early Development" / "Alpha" / "Beta"
- Install commands only shown when packages actually exist
- Roadmap items marked as planned/in-progress/done
- No screenshots or examples of features that don't work yet

---

## Technology Stack

| Component | Technology | Why |
|-----------|-----------|-----|
| Core runtime | Rust | Deterministic, fast, compiles to WASM |
| CLI | Rust (clap) | Same codebase as core |
| Python binding | PyO3 + maturin | Compile Rust → native Python module |
| JS binding | wasm-pack | Compile Rust → WebAssembly for npm |
| Go binding | cgo FFI | Link to compiled Rust shared lib |
| Documentation | mdBook | Rust ecosystem standard, deploys as static site |
| CI/CD | GitHub Actions | Multi-target build + publish |
| Testing | Rust built-in + conformance suite | Determinism proofs |

---

## Future Repos (create when needed, NOT now)

| Repo | When | Purpose |
|------|------|---------|
| `icl-vscode` | When parser works | VS Code extension (syntax highlighting + validation) |
| `icl-action` | When CLI works | GitHub Action for CI contract validation |
| `.github` | Anytime | Org-level profile, issue templates, shared workflows |
| `icl-examples` | When ecosystem grows | Community-contributed example contracts |

---

## Development Priorities

> **Phases 0–9 are complete.** Phase 10 (Standardization) is next.

1. **Phase 0 — Restructure** ✅: Clean repos, move files, set up workspace
2. **Phase 1 — Parser** ✅: Tokenizer + parser + AST (make `.icl` files parseable)
3. **Phase 2 — Normalizer** ✅: Canonical form + idempotence proof
4. **Phase 3 — Verifier** ✅: Type checking + invariant verification + determinism checks
5. **Phase 4 — CLI** ✅: `icl-cli validate`, `icl-cli normalize`, `icl-cli fmt`, `icl-cli hash`
6. **Phase 5 — Executor** ✅: Sandboxed contract execution (pure Rust)
7. **Phase 6 — Bindings** ✅: Python (PyO3) + JS (wasm-pack) + Go (cgo)
8. **Phase 7 — Docs Site** ✅: mdBook build + deploy to GitHub Pages
9. **Phase 8 — CI/CD** ✅: Automated testing + multi-target publish
10. **Phase 9 — Conformance** ✅: Full conformance test suite
11. **Phase 10 — Standardization**: RFC, governance, iclstandard.org
