# Knowledge Base Module Laws and Invariants

The laws the `knowledge-base` module adds to catalyst's ten (the
kernel's `INVARIANTS.md`). This file extends the kernel's and never
overrides it (`MODULE-SPECIFICATION.md` §6.4).

## The law

- **KB-L1 — Classification only rises.** Every document and pointer in a
  knowledge base is classified, at the higher of its own mark and its
  knowledge base's; nothing reaches a build above its ceiling, or an
  embedding backend above its allowance; a mark is lowered only by a
  person's journaled decision.
- **KB-L2 — Alone or beside any module.** The knowledge base module (KBM)
  is a criterion's only module or sits beside any other, so knowledge bases
  are built off any project. It never requires, names or assumes another
  module: its commands, documents and checks hold whichever module governs
  the project.

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
| INV-KB-8 alone or beside any module, naming none | KB-L2, L9 | catalyst's CI installs KBM alone, beside another module and with another beside it (the kernel's INV-30, modules compose) |
| INV-KB-9 a build is readable by any model, without catalyst | KB-L1, L9 | `GUIDE.md` "A build"; the engine's build writer and its tests (`catalyst-kb-engine`) |
