# Knowledge bases: establishing and maintaining them

How a knowledge base is established, laid out, kept fresh and built into vector databases. The rules behind each step
are the module's meta-rules (`rr-KB-001` to `rr-KB-007`); the commands that carry them out are in
`CODE-OF-CONDUCT.md` §4.

## Two kinds

| Kind | What it draws on | Typical classification |
|---|---|---|
| **Project** | Any conglomerate of information outside catalyst: repositories, wikis, drives, sites, exports | Chosen per project |
| **Reflective** | Only the Cantica ecosystem: the catalyst kernel, its modules and its plugins (`rr-KB-005`) | `Internal`, `Public` for what is published |

## Classification

Four levels, lowest to highest: `Public` < `Internal` < `Confidential` < `Restricted`.

- **The knowledge base's own** (`Classification`) is the floor of everything in it.
- **A document's or a pointer's own mark** is its `classification` in its front matter.
- **Its effective classification** is the higher of the two. A public document in an internal knowledge base is
  internal; a restricted document in a public knowledge base stays restricted.
- **The policy** (`Default classification`) marks every document or pointer that is new, imported or unmarked, when
  it is written, unless its source gives a mark (`KSOURCE` `Classification`). Nothing stays unmarked: `/kb-build`
  refuses while anything is.
- **A build** contains only what is at or below its `Ceiling`, and is classified at the highest effective
  classification it contains.
- **Embedding** goes to a hosted service only up to `Hosted embedding up to`; anything above is embedded by a local
  model and never leaves the machine.

## Establishing a knowledge base

1. **Decide** what it must make known, for whom, its kind and its classification. Name the rule it serves; propose
   one if none says it.
2. **Record it:** `/kb-setup project|reflective <title>` creates the `KB-` (`Proposed`), with its policy fields.
3. **Name a curator:** a `Knowledge Curator` (the role `/kb-setup` offers to add), or an `Admin`.
4. **Create the repository** (below), with `KB.md` naming the `KB-` ID. Write the Scope and the Structure: the
   knowledge base is `Established`.
5. **Record its sources:** one `KSOURCE-` for each place it draws on, with how its documents come in (Import) and
   the mark they get.
6. **Write the Policy and the Maintenance:** it is `Active`.
7. **Import:** `/kb-import <KSOURCE-id>` for each source. Every document and pointer is marked as it is written.
8. **Audit, then build:** `/kb-audit` finds nothing unmarked; `/kb-build <KB-id>` makes the first vector database.

## Maintaining it

| When | What |
|---|---|
| Within each `Freshness` | `/kb-refresh <KB-id>`: every pointer and source checked; moved and broken ones marked |
| When a source changes | `/kb-import` again for what is new; changed documents updated, journaled |
| When a mark is wrong | `/kb-classify <path> <level>`; lowering one is a person's decision |
| Before each build | `/kb-audit`: nothing unmarked, nothing stale, nothing over a published build's ceiling |
| When content has moved on | `/kb-build`: a new version; the previous one `Superseded` once the new one is `Published` |
| A reflective knowledge base | Refreshed and rebuilt at each release of the kernel or a module |
| When it is no longer needed | `Frozen` (kept, not changed), then `Retired`; its builds withdrawn or superseded, never deleted |

## Governance

Catalyst enforces what this section sets up (kernel 0.104.0, `CLI.md` "Workflow governance"). `/kb-setup` offers each
part; none is added without the owner's assent.

- **Freshness:** each source's `Next check` is its last check plus its knowledge base's `Freshness`; past it, the
  source is `overdue` in `catalyst inbox`, escalated to the role its workflow names.
- **A workflow governing builds:** a `WORKFLOW-` with `Governs: KBUILD`, `Escalates to: Knowledge Curator`, made
  `Active`, its Transitions table:

  | From | To | Role | When |
  |---|---|---|---|
  | Planned | Built | Knowledge Curator | |
  | Built | Published | Knowledge Curator | Classification at most Internal |
  | Built | Published | Admin | |
  | Published | Superseded | Knowledge Curator | |
  | * | Withdrawn | Knowledge Curator | |

  A curator publishes a public or internal build; a confidential or restricted one is published by an `Admin`, after
  the gate below. Catalyst refuses any other move, and `--force` does not pass it.
- **A gate on sensitive builds**, in `IAM/gates.json`:

  ```json
  {"id": "kb-sensitive-build", "covers": {"type": "KBUILD", "to": "Published"},
   "when": "Classification at least Confidential", "approvals": 2, "role": "Knowledge Curator"}
  ```

  Two curators approve the build's exact version, neither its author (L4).
- **A workflow governing knowledge bases** (optional): `Governs: KB`, its moves `Proposed → Established →
  Active → Frozen → Retired` each by a `Knowledge Curator`, `Retired` by an `Admin`.

## The repository

A knowledge base is its own repository, a catalyst project like any other (with its `catalyst.toml`, governed by
this module), so every document added, changed or reclassified is journaled.

```
<knowledge base>/
├── catalyst.toml
├── KB.md                 the KB- ID, its title, classification, default classification, and a contents tree
├── docs/                 the documents: hierarchical markdown, one topic per directory
│   ├── README.md         what this level holds
│   └── <topic>/
│       ├── README.md
│       └── <document>.md
├── pointers/             pointers to outside documents, mirroring docs/'s hierarchy
│   └── <topic>/<pointer>.md
└── builds/               one directory per build version: its manifest only (the build itself is kept elsewhere)
    └── <version>/manifest.json
```

Each directory's `README.md` says what it holds and lists its documents and pointers. Names are descriptive,
lowercase, hyphenated.

### A document

```markdown
---
title: How invoices are approved
classification: Internal
source: KSOURCE-000002-Ab3xR9pQ
imported: 2026-10-22
updated: 2026-10-22
---

# How invoices are approved

…
```

`classification` is required; `source` and `imported` say where it came from (none for a document written in the
knowledge base); `updated` is its last change.

### A pointer

```markdown
---
title: The finance team's approval matrix
url: https://drive.example.com/finance/approval-matrix
classification: Confidential
source: KSOURCE-000003-Ab3xR9pQ
checked: 2026-10-22
status: ok
etag: "a1b2c3"
---

What it holds and why it matters, in a sentence or two: this text is what a build embeds for a pointer.
```

`status` is `ok`, `moved` (with the new `url`) or `broken`, set by `/kb-refresh`; `etag` (or a content hash) tells it
whether the target changed. A pointer is never deleted silently.

## A build

A build is a versioned vector database in portable files, loadable by any vector store (FAISS, LanceDB, pgvector,
Chroma) and by any model:

```
<version>/
├── manifest.json
├── chunks.jsonl
└── vectors.parquet
```

- **`manifest.json`**: what it is.

  ```json
  {
    "knowledge_base": "KB-000001-Ab3xR9pQ",
    "build": "KBUILD-000004-Ab3xR9pQ",
    "version": "1.2.0",
    "classification": "Internal",
    "ceiling": "Internal",
    "content_commit": "0123456789abcdef0123456789abcdef01234567",
    "embedding": {"model": "nomic-embed-text-v1.5", "dimensions": 768, "where": "local"},
    "chunking": {"max_tokens": 512, "overlap": 64, "by": "heading"},
    "counts": {"documents": 412, "pointers": 57, "chunks": 3120, "left_out": 9},
    "files": {"chunks.jsonl": "sha256:…", "vectors.parquet": "sha256:…"},
    "built": "2026-10-22T10:00:00Z"
  }
  ```

- **`chunks.jsonl`**: one line per chunk: `{"id", "path", "title", "heading", "text", "classification", "source",
  "url"}`, `url` for a pointer's chunk.
- **`vectors.parquet`**: one row per chunk: `id` and `vector` (a fixed-size list of floats, the manifest's
  `dimensions`), in `chunks.jsonl`'s order.

The repository keeps only `builds/<version>/manifest.json`; the build's files are kept where its `KBUILD-` says
(`Location`): a release asset, a bucket, a path. A build's classification travels with it: it is stored and shared
only where that classification allows.

A version is `MAJOR.MINOR.PATCH`: a new major when the embedding model or the chunking changes (the vectors are not
comparable), a new minor when content is added or removed, a patch for corrections.
