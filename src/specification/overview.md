# Specification Overview

The ICL specification is maintained in the [ICL-Spec](https://github.com/ICL-System/ICL-Spec) repository. This page provides a summary.

## Core Sections

An ICL contract consists of 7 sections (6 required, 1 optional):

1. **Identity** — Stable ID, version, timestamp, owner, semantic hash
2. **PurposeStatement** — Narrative description, intent source, confidence level
3. **DataSemantics** — State definition with types and invariants
4. **BehavioralSemantics** — Operations with pre/postconditions
5. **ExecutionConstraints** — Resource limits, permissions, sandbox mode
6. **HumanMachineContract** — Commitments, refusals, user obligations
7. **Extensions** — Optional extension points for domain-specific needs

## Type System

### Primitives
| Type | Description | Example |
|------|-------------|---------|
| Integer | 64-bit signed | `42` |
| Float | 64-bit IEEE 754 | `3.14` |
| String | UTF-8 text | `"hello"` |
| Boolean | Logical | `true` |
| ISO8601 | Timestamp | `"2025-01-01T00:00:00Z"` |
| UUID | Unique ID | `"550e8400-..."` |

### Composites
| Type | Description | Syntax |
|------|-------------|--------|
| Object | Structured record | `Object { name: String, age: Integer }` |
| Enum | Tagged union | `Enum { Active, Inactive, Suspended }` |
| Array | Ordered collection | `Array<String>` |
| Map | Key-value | `Map<String, Integer>` |

## Determinism Requirements

Every ICL implementation MUST guarantee:
- Same input → same output (no randomness)
- No system time dependency in core logic
- BTreeMap not HashMap (ordered iteration)
- All floating point operations use IEEE 754 semantics

## Formal Grammar

The complete BNF grammar is available at:
[ICL-Spec/grammar/icl.bnf](https://github.com/ICL-System/ICL-Spec/blob/main/grammar/icl.bnf)

## Full Specification

Read the complete specification:
[CORE-SPECIFICATION.md](https://github.com/ICL-System/ICL-Spec/blob/main/spec/CORE-SPECIFICATION.md)
