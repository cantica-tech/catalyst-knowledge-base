# catalyst-knowledge-base

A catalyst process module with one purpose: building and maintaining **knowledge bases**: a project's, from any
information outside catalyst, or a **reflective** one of the Cantica ecosystem (the kernel, its modules, its
plugins). A knowledge base is a classified repository of markdown documents and fresh pointers to outside documents,
from which versioned vector databases are built for models to ingest.

- **`module.yaml`**: the module in catalyst's module format 3: its tiers (`chore`, `kb`, `build`), the namespace `KB`
  of its meta-rules (`rr-KB-NNN`) and invariants (`INV-KB-n`), its three types, knowledge bases (`KB-`), their
  sources (`KSOURCE-`) and their builds (`KBUILD-`), and its commands.
- **`GUIDE.md`**: how a knowledge base is established and maintained; its classification; the layout of its
  repository, of a document, of a pointer and of a build.
- **`docs/best-practices.md`**: current practice for AI knowledge bases (sources, chunking, retrieval, embeddings,
  freshness, classification, evaluation), with sources, mapped to this module; and how to build one from a catalyst
  criterion.
- **`rules-of-rules.module.md`**: its meta-rules, composed into the deployed `Rules-of-Rules.md`.
- **`code-of-conduct.module.md`**: its document types and commands (`/kb-setup`, `/kb-import`, `/kb-classify`,
  `/kb-refresh`, `/kb-build`, `/kb-audit`), composed into the deployed `CODE-OF-CONDUCT.md`.
- **`INVARIANTS.module.md`**: the law it adds, KB-L1: classification only rises.

**Classification.** Every document and pointer is `Public`, `Internal`, `Confidential` or `Restricted`, effectively the
higher of its own mark and its knowledge base's; a policy per knowledge base marks what is new, imported or
unmarked. A build contains nothing above its ceiling, and a document reaches a hosted embedding service only up to
what its knowledge base allows.

Install it with `catalyst module install <this repository>`, beside another module or as a deployment's first, or
with `catalyst init --module knowledge-base`. It needs kernel 0.104.0 or later: its guards hold field conditions, its sources fall due, and its builds can be governed by a workflow and a conditional gate. It is catalyst's second
production module, the proof in catalyst's CI that the kernel names no module type.
