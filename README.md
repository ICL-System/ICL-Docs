# ICL Documentation

> **Status: Phases 0–10 Complete**
>
> Documentation website for the [Intent Contract Language (ICL)](https://github.com/ICL-System/ICL-Spec) ecosystem. Built with [mdBook](https://rust-lang.github.io/mdBook/).

## How is ICL Different from Guardrails?

Guardrails and system prompts are *suggestions* an LLM can ignore or misinterpret. ICL contracts are **mathematically enforced walls** — verified before execution, impossible to bypass.

| | System Prompts | Guardrails | **ICL** |
|---|---|---|---|
| **What it is** | Natural language instructions | Runtime filters | Formal, verified contracts |
| **Enforcement** | LLM interprets (may ignore) | Probabilistic | **Mathematical proof** |
| **Analogy** | "Please don't" | Smoke detector | **Fireproof wall** |

**Where ICL is used:** Trading agents that CANNOT exceed limits · Surgical robots that MUST stop if sensors fail · Drones that CANNOT enter restricted airspace · Code deploy agents that CANNOT ship without passing tests

> See the full [ICL vs Guardrails — 50+ real-world examples](https://icl-system.github.io/ICL-Docs/use-cases.html) for when (and when not) to use ICL.

## Building Locally

```bash
# Install mdBook
cargo install mdbook

# Build and serve
mdbook serve --open
```

The site will be available at `http://localhost:3000`.

## Structure

```
ICL-Docs/
├── book.toml          # mdBook configuration
├── src/
│   ├── SUMMARY.md     # Table of contents (defines sidebar)
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
└── README.md
```

## Related Repositories

| Repo | Purpose |
|------|---------|
| [ICL-Spec](https://github.com/ICL-System/ICL-Spec) | The standard: BNF grammar, specification, conformance tests |
| [ICL-Runtime](https://github.com/ICL-System/ICL-Runtime) | Canonical Rust implementation + CLI |

## License

MIT — See [LICENSE](LICENSE) for details.
