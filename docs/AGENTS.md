# FlowState Agent Instructions

## Canonical Documentation

The Product Design Specification located at:

docs/PRODUCT_DESIGN_SPECIFICATION.md

is the canonical source of truth.

If implementation differs from the PDS, explain the conflict instead of guessing.

---

## General Rules

- Never invent product features.
- Never change product philosophy.
- Never rename established terminology.
- Never introduce parallel systems that duplicate existing functionality.
- Reuse existing components and utilities whenever appropriate.
- Preserve consistency with the PDS.

---

## Scope

Implement only the subsystem requested.

Avoid modifying unrelated files.

Avoid unrelated refactors.

---

## Implementation

Prefer simple, maintainable solutions.

Keep code modular.

Document major architectural decisions.

If a requirement is ambiguous, ask for clarification instead of making assumptions.
