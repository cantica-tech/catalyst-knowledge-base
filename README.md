# catalyst-knowledge-base

A catalyst process module with one purpose: building and maintaining **knowledge bases**: a project's, from any
information outside catalyst, or a **reflective** one of the Cantica ecosystem (the kernel, its modules, its
plugins). A knowledge base is a classified repository of markdown documents and fresh pointers to outside documents,
from which versioned builds are made for models to read: organized markdown folders or vector databases, in open
formats any model can consume without catalyst.

- **`module.yaml`**: the module in catalyst's module format 3: its tiers (`chore`, `kb`, `build`), the namespace `KB`
  of its meta-rules (`rr-KB-NNN`) and invariants (`INV-KB-n`), its three types, knowledge bases (`KB-`), their
  sources (`KSOURCE-`) and their builds (`KBUILD-`), and its commands.
- **`GUIDE.md`**: how a knowledge base is established and maintained; its classification; the layout of its
  repository, of a document, of a pointer and of a build.
- **`docs/best-practices.md`**: current practice for AI knowledge bases (sources, chunking, retrieval, embeddings,
  freshness, classification, evaluation), with sources, mapped to this module; and how to build one from a catalyst
  criterion.
- **`docs/engine-and-onboarding.md`**: the engine and onboarding design (criterion-based or documents-based, static or
  dynamic, folders or vector database) and the open-source Python technologies that could run it (research, not built).
- **`rules-of-rules.module.md`**: its meta-rules, composed into the deployed `Rules-of-Rules.md`.
- **`code-of-conduct.module.md`**: its document types and commands (`/kb-setup`, `/kb-import`, `/kb-classify`,
  `/kb-refresh`, `/kb-build`, `/kb-audit`), composed into the deployed `CODE-OF-CONDUCT.md`.
- **`INVARIANTS.module.md`**: the laws it adds, KB-L1 (classification only rises) and KB-L2 (alone or beside any
  module).

**Classification.** Every document and pointer is `Public`, `Internal`, `Confidential` or `Restricted`, effectively the
higher of its own mark and its knowledge base's; a policy per knowledge base marks what is new, imported or
unmarked. A build contains nothing above its ceiling, and a document reaches a hosted embedding service only up to
what its knowledge base allows.

**Alone or beside any module** (KB-L2): the knowledge base module (KBM) is a criterion's only module, or sits beside
any other, so knowledge bases are built off any project. Install it with `catalyst module install <this repository>`
beside the module that already governs a project, or with `catalyst init --module knowledge-base` as a criterion's
only module. It needs kernel 0.107.0 or later: its guards hold field conditions, its sources fall due, its builds can
be governed by a workflow and a conditional gate, and a module's own catalyst project file never ships. It is
catalyst's second production module, the proof in catalyst's CI that the kernel names no module type.

**Branches.** Work lands on `development` (the default branch), where CI runs on every push; `main` takes it by a
pull request from `development`, and each release is tagged on `main`.
