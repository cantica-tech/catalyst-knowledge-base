# Rules of Rules — cantica-knowledge-base module

> Module contribution (`MODULE-SPECIFICATION.md` §6.1): appended to the
> deployed `rules/Rules-of-Rules.md`, after the kernel's handbook, under
> `### From module cantica-knowledge-base`. It never replaces kernel content.
> Its meta-rules are numbered in the module's namespace, `rr-KB-NNN`.

This module has one purpose: building and maintaining knowledge bases. A
knowledge base (`KB-`) is a repository of markdown documents and pointers to
outside documents, kept classified and fresh, from which versioned vector
databases (`KBUILD-`) are built for models to ingest. Its sources
(`KSOURCE-`) say where its documents come from. There are two kinds: a
**project** knowledge base, from any information outside catalyst, and a
**reflective** one, of the Cantica ecosystem only. How the repository and a
build are laid out is the module's `GUIDE.md`.

## The tier decides the artifact (KB-L1)

A chore changes no rule's behaviour and has no artifact. Establishing,
reclassifying or retiring a knowledge base is a `KB-` change; producing a
vector database is a `KBUILD-`. A source is opened when a knowledge base
starts drawing on it, never as a tier of its own.

## 1. `rr-KB-001` A knowledge base is established before it is filled, and active only with its policy

A `KB-` names the rules it serves (`Serves`). It is `Established` once its
Structure says how its repository is laid out, and `Active` only once its
Policy and Maintenance are written. Nothing is imported into a knowledge base
before it is `Established`, and nothing is built from one that is not
`Active`. A `Frozen` one is kept as it is: no import, no build; a `Retired`
one is retired in place, its builds withdrawn or superseded.

## 2. `rr-KB-002` Classification only rises

Every document and pointer carries a classification: `Public`, `Internal`,
`Confidential` or `Restricted`, in that order. One that is new, imported or
unmarked gets its source's mark, else its knowledge base's `Default
classification`, when it is written — nothing stays unmarked. Its
**effective classification** is the higher of its own mark and its knowledge
base's `Classification`: a public document in an internal knowledge base is
internal; a restricted document in a public knowledge base stays restricted.
A mark is lowered only by a person's decision, journaled with why.

## 3. `rr-KB-003` Pointers are kept fresh, never silently dropped

A pointer names an outside document and when it was last checked. Within
each knowledge base's `Freshness`, `/kb-refresh` checks every pointer and
source and marks what moved or broke. A broken pointer stays, marked, until
a person replaces or removes it, journaled; a source is retired in place.

## 4. `rr-KB-004` A build stays within its ceiling and its embedding allowance

A `KBUILD-` contains only documents and pointers whose effective
classification is at or below its `Ceiling`, and is itself classified at the
highest effective classification it contains. A document is embedded only by
a backend its knowledge base allows for its classification: a hosted service
up to `Hosted embedding up to`, a local model above it. A build is versioned,
records the model and the repository commit it came from, is `Published` only
with its Verification written, and is superseded or withdrawn, never deleted.

## 5. `rr-KB-005` A reflective knowledge base draws only on the Cantica ecosystem

A reflective knowledge base's sources are the Cantica ecosystem's own
repositories and releases: the kernel, its modules and its plugins. It is
refreshed and rebuilt when one of them is released.

## 6. `rr-KB-006` A curator decides what goes in, at what classification

Importing, classifying and building are a `Knowledge Curator`'s or an
`Admin`'s; anyone may refresh pointers and audit. An agent proposes a
classification and never lowers one on its own (L4).
