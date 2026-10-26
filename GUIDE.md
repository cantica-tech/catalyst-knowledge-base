# Knowledge bases: establishing and maintaining them

How a knowledge base is established, laid out, kept fresh and built into what a model reads (organized folders or a vector database). The rules behind each step
are the module's meta-rules (`rr-KB-001` to `rr-KB-009`); the commands that carry them out are in
`CODE-OF-CONDUCT.md` §4.

Why each step is what it is, with the research behind it: [`docs/best-practices.md`](docs/best-practices.md).

## Alone or beside any module

The knowledge base module (KBM) works in two setups (KB-L2, `rr-KB-008`):

- **Alone**: a criterion whose only module is KBM, made for its knowledge bases (`catalyst init --module
  knowledge-base`).
- **Beside another module**: added to the criterion of an existing project, beside the module that already governs it
  (`catalyst module install <KBM release>`), so its knowledge bases are built off that project. Either can be
  installed first.

Nothing below depends on which: KBM names no other module, and its commands and checks hold in both.

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
- **A private source is a floor.** Every document read from a source whose `Visibility` is `Private` is at least
  `Internal`, whatever its own mark. A source's visibility changes when its host's does: each build re-reads it, and
  refuses to run when the `KSOURCE-` record and the host (such as GitHub) disagree. A source made private leaves the
  next `Public` build; a source made public joins it.
- **The policy** (`Default classification`) marks every document or pointer that is new, imported or unmarked, when
  it is written, unless its source gives a mark (`KSOURCE` `Classification`). Nothing stays unmarked: `/kb-build`
  refuses while anything is.
- **A build** contains only what is at or below its `Ceiling`, and is classified at the highest effective
  classification it contains.
- **Embedding** goes to a hosted service only up to `Hosted embedding up to`; anything above is embedded by a local
  model and never leaves the machine.

## Onboarding

Every knowledge base is onboarded before its first build, whether KBM is alone or beside other modules
(`/kb-setup`). Every choice is the curator's; the engine or an agent proposes.

| Step | Criterion-based (KBM beside other modules) | Documents-based (KBM alone) |
| --- | --- | --- |
| 1. Basis and sources | the project's criterion is the first source, read through catalyst's CLI; select the sources that **enrich** it | **gather** the sources: what, where, how reached, how classified |
| 2. Location and storage | where its repository lives, and how its documents are organized in folders | the same |
| 3. Target | `Folders` (organized markdown documents) or `Vector database` (chunks and vectors) | the same |
| 4. Document base | `Dynamic` by default (a criterion is always in flux), or `Static` | `Static` or `Dynamic` |

- **Static**: defined once, never changed; it takes no new source after its first build, and a build is a function
  of it alone.
- **Dynamic**: grows and shrinks at will; each build re-reads its sources and processes only what changed.

The `KB-` record keeps what onboarding decided: `Basis`, `Document base` and `Target`.

Each source is a `KSOURCE-` with its `Kind` and its `Visibility`:

| Kind | What it is | `Location` |
| --- | --- | --- |
| `Repository` | a git repository of documents | its URL, or a path |
| `Folder` | a directory or a drive folder | its path or URL |
| `Site` | a website or a wiki space | its URL |
| `Criterion` | a catalyst project's criterion: this project's, or another's | `criterion:<project name>` |

A `Criterion` source is read only through catalyst's CLI (`catalyst list`, `view --json`), never by reading its files
and never by knowing which modules wrote it. Each entity becomes a document citing its ID, its criterion and its
status, with its own history from the journal. A criterion is private unless its repository is public, so its
documents are at least `Internal`.

## Establishing a knowledge base

1. **Onboard it** (above): what it must make known, for whom, its kind and classification, the rule it serves (propose
   one if none says it), its basis and sources, its location and storage, its target and its document base.
2. **Record it:** `/kb-setup project|reflective <title>` creates the `KB-` (`Proposed`), with its policy and
   onboarding fields, and one `KSOURCE-` per source (the criterion first, for a criterion-based one).
3. **Name a curator:** a person holding the Knowledge Curator role in catalyst-users (which `/kb-setup` offers to
   provision), or an `Admin`.
4. **Create the repository** (below), with `KB.md` naming the `KB-` ID. Write the Scope and the Structure: the
   knowledge base is `Established`.
5. **Write each source's Import:** how its documents come in, and the mark they get.
6. **Write the Policy and the Maintenance:** it is `Active`.
7. **Import:** `/kb-import <KSOURCE-id>` for each source. Every document and pointer is marked as it is written.
8. **Audit, then build:** `/kb-audit` finds nothing unmarked; `/kb-build <KB-id>` makes the first build of its target.

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

A build is what a model reads: **organized markdown folders** or a **vector database**, as the knowledge base's
`Target` says. Either is readable by any model, without catalyst, an account or a network (`rr-KB-009`): open formats
only, a manifest and a README that say what it is and how to load it, and every document and chunk carrying its
effective classification.

### Folders

```
<version>/
├── manifest.json
├── README.md          what it is, its classification, how to read it
├── index.md           every document: its path, title and one-line summary, linked relatively
└── docs/              the documents at or below the ceiling, organized as the knowledge base's folders
    └── <topic>/<document>.md
```

Each document keeps its front matter (`title`, `classification`, `source`, `updated`), with `hash` (its content's
SHA-256) added; pointers become documents holding their summary and `url`.

### Vector database

```
<version>/
├── manifest.json
├── README.md          what it is, its embedding model, how to load it (any vector store, any language)
├── chunks.jsonl
├── vectors.parquet
└── docs/              the documents the chunks came from, as for Folders
```

- **`manifest.json`**: what it is.

  ```json
  {
    "format": "kbm-build/1",
    "target": "vector-database",
    "knowledge_base": "KB-000001-Ab3xR9pQ",
    "build": "KBUILD-000004-Ab3xR9pQ",
    "version": "1.2.0",
    "classification": "Internal",
    "ceiling": "Internal",
    "content_commit": "0123456789abcdef0123456789abcdef01234567",
    "embedding": {"model": "nomic-embed-text-v1.5", "dimensions": 768, "where": "local",
                  "normalized": true, "distance": "cosine", "input": "context",
                  "document_prefix": "search_document: ", "query_prefix": "search_query: "},
    "chunking": {"max_tokens": 512, "overlap": 64, "by": "heading"},
    "incremental": {"previous": "1.1.0", "reused": 3080, "embedded": 40},
    "counts": {"documents": 412, "pointers": 57, "chunks": 3120, "left_out": 9},
    "files": {"chunks.jsonl": "sha256:…", "vectors.parquet": "sha256:…"},
    "built": "2026-10-22T10:00:00Z"
  }
  ```

  A folders build's manifest has `"target": "folders"`, no `embedding` or `chunking`, and lists `index.md` and the
  documents under `files`.

  `embedding.input` says what each chunk was embedded as: `text`, its text alone, or `context`, its `title`, a newline,
  its `heading` (when it has one), a blank line and its `text`. `document_prefix` came before that, and a question is
  embedded with `query_prefix` before it, as some models expect. A manifest without them means `text` and no
  prefixes. `incremental`, when present, names the previous build whose vectors were reused for unchanged chunks, and
  counts the reused and the newly embedded ones.

- **`chunks.jsonl`**: one line per chunk: `{"id", "path", "title", "heading", "text", "classification", "source",
  "url"}`, `url` for a pointer's chunk.
- **`vectors.parquet`**: one row per chunk: `id` and `vector` (a fixed-size list of floats, the manifest's
  `dimensions`), in `chunks.jsonl`'s order. A consumer queries with the same embedding model, or re-embeds the chunks
  (or the documents) with its own. A vector store (FAISS, LanceDB, pgvector, Chroma) loads them directly; one built
  from them is a convenience, never the only copy.

### Where builds are kept

The repository keeps only `builds/<version>/manifest.json`; the build's files are kept where its `KBUILD-` says
(`Location`): a release asset, a bucket, a path. A build's classification travels with it: it is stored and shared
only where that classification allows.

## The engine

`catalyst-kb-engine` (its own repository, built on Pydantic AI) carries onboarding and builds out as workflows: it
proposes, a person decides, and its state is this criterion's `KB-`, `KSOURCE-`, `KBUILD-` and `WORKFLOW-` records,
journaled. Until it is released, the commands above are carried out by the agent following them.
