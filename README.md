# ICL Documentation

> **Status: Phases 0–9 Complete**
>
> Documentation website for the [Intent Contract Language (ICL)](https://github.com/ICL-System/ICL-Spec) ecosystem. Built with [mdBook](https://rust-lang.github.io/mdBook/).

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
