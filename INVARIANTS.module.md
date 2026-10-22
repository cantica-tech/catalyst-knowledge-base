# Knowledge Base Module Laws and Invariants

The law the `knowledge-base` module adds to catalyst's ten (the
kernel's `INVARIANTS.md`). This file extends the kernel's and never
overrides it (`MODULE-SPECIFICATION.md` §6.4).

## The law

- **KB-L1 — Classification only rises.** Every document and pointer in a
  knowledge base is classified, at the higher of its own mark and its
  knowledge base's; nothing reaches a build above its ceiling, or an
  embedding backend above its allowance; a mark is lowered only by a
  person's journaled decision.

<!-- catalyst: end of the session brief -->

## The invariants

| Invariant | Law | Enforced by |
|---|---|---|
| INV-5 chain (module part) | L2, KB-L1 | `validate`: a knowledge base's `Serves` is required; sources and builds inherit it |
| INV-KB-1 sources and builds cite their knowledge base back | L6 | `backref`, `check` |
| INV-KB-2 an active knowledge base has its policy | KB-L1 | the `Active` guard (`section_written: Policy`, `Maintenance`) |
| INV-KB-3 a published build has its verification | KB-L1 | the `Published` guard (`section_written: Verification`) |
| INV-KB-4 nothing unmarked, nothing over its ceiling | KB-L1 | `/kb-audit`; `/kb-build` refuses an unmarked document |
| INV-KB-5 an active knowledge base has its curator; a published build its classification | KB-L1 | the `Active` and `Published` guards (`holds`) |
| INV-KB-6 a source past its next check is overdue | L1 | `due: "Next check"`: `catalyst inbox`, `catalyst check` |
| INV-KB-7 builds move as their workflow says; a sensitive build needs two curators | L4 | a governing `WORKFLOW-`; a gate with `when` (kernel 0.104.0) |
