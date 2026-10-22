# Best practices for building an AI knowledge base

What current practice (2024–2026) says about building a knowledge base that models retrieve from, gathered from
published research, vendor engineering posts and security guidance, and mapped to this module: knowledge bases
(`KB-`), their sources (`KSOURCE-`) and their builds (`KBUILD-`), as `GUIDE.md` lays them out. The last part covers a
special case: a knowledge base built from a catalyst criterion.

Most numbers below come from practitioner posts, not independent benchmarks. Treat them as starting points to test
against your own corpus, never as targets. The sources are listed at the end.

## 1. Decide whether you need retrieval at all

- **A small knowledge base can go straight into the prompt.** Anthropic's guidance: under about 200,000 tokens
  (roughly 500 pages), include the whole knowledge base in the prompt and use prompt caching; no retrieval is needed
  [1]. A knowledge base's repository is still worth keeping (classified, fresh, versioned), and a build may then be a
  single document instead of vectors.
- **Write down the questions first.** Who asks, what they ask, and what a good answer cites. These questions become
  the evaluation set (§9) and decide the chunking (§4) and whether a graph helps (§10). A knowledge base's Scope says
  them.

## 2. Choose and curate the sources

- **Authoritative sources only.** One owner per source; a document that two sources hold is taken from one. A
  `KSOURCE-` names where its documents come from, how they are read and what mark they get.
- **Provenance on everything.** Record who supplied each document, when, from where, and with what approval; keep a
  digest of each approved document (OWASP recommends a SHA-256 manifest, itself protected) [2]. In this module: each
  document's front matter (`source`, `imported`), and each build's manifest checksums.
- **Stage before indexing.** New content is reviewed (classified, scanned) before it reaches a build, never written
  straight into the index [2]. In this module: `/kb-import` writes into the repository; only `/kb-build` reaches the
  vectors.
- **Prefer pointers to copies for what changes elsewhere.** A pointer kept fresh is better than a copy that drifts;
  copy only what must be read when the source is unavailable.

## 3. Write documents for retrieval

A chunk is read out of context. Write so that each section stands on its own.

- One topic per document, one idea per section, under a descriptive heading that names its subject (not "Overview").
- Name things in full in each section: "the invoice approval threshold", not "it".
- State dates and versions in the text ("since 2026-03"), so a retrieved passage carries its validity.
- Define terms once, in a glossary document, and use them consistently.
- Keep a curated index: a top-level markdown file listing the important documents with a line each. This is the idea
  behind `llms.txt` [3], an emerging convention, not a ratified standard. This module's `KB.md` and each directory's
  `README.md` play that role.
- Strip what is not content (navigation, boilerplate, repeated footers) before import.

## 4. Chunk deliberately, and test it

- **Start from a recursive baseline.** Split by structure (headings, then paragraphs), around 300–512 tokens, with
  10–15% overlap [4][5]. In one 2026 comparison, recursive 512-token splitting beat semantic chunking by 15 points on
  the same corpus [5]; other studies favour structure-aware chunks for contracts and technical documents [4][6].
  The lesson is the same: measure on your own corpus (§9), don't assume.
- **Respect the document's structure.** A chunk never spans two sections; a table or a list stays whole with its
  heading.
- **Parent and child.** Retrieve by small chunks, but hand the model the larger section they belong to [7].
- **Give each chunk its context.** Anthropic's *contextual retrieval* prepends 50–100 tokens situating each chunk in
  its document before embedding and keyword-indexing it. It cut the top-20 retrieval failure rate from 5.7% to 3.7%
  (embeddings), to 2.9% (with contextual BM25), and to 1.9% with reranking: a 67% reduction [1]. With prompt caching
  the one-time cost was estimated at about $1.02 per million document tokens [1].

## 5. Retrieve with more than one signal

- **Hybrid search:** combine embedding similarity with keyword search (BM25), which finds exact identifiers, error
  codes and names that embeddings blur; merge the two result lists by rank fusion [1].
- **Rerank:** retrieve broadly (Anthropic took the top 150), rerank, and pass the best 20 to the model; 20 did better
  than 10 or 5 in their tests [1].
- **Filter before searching:** apply classification and permission filters before similarity search, not after [2]
  (§8).
- **Limit what reaches the model:** a bounded number of chunks and tokens limits both cost and the surface for
  injected instructions [2].

## 6. Treat embeddings as versioned artifacts

- **Pin the model and record it.** Vectors from two models live in incompatible spaces; similarity across them is
  meaningless. Record the model (name, version, dimensions) on every vector and in the build's manifest [8].
- **Never mix two models in one index.** The failure is silent: a store accepts vectors of the right dimension from
  any model, including a provider's quietly updated one [8][9].
- **Re-embed side by side.** Build the new index next to the old, validate recall on the evaluation set, then switch
  [8]. In this module: a new embedding model or chunking is a new **major** build version (`GUIDE.md`, "A build"), and
  the old build is `Superseded` only once the new one is `Published`.

## 7. Keep it fresh

- **Detect change by content hash, not by timestamp.** Re-process only what changed; pair change events with a
  periodic hash sweep that catches missed events [10].
- **Handle updates and deletions, not only additions.** That is where freshness fails [11]: a deleted or superseded
  document must leave every build that follows, and a reconciliation pass checks that no stale chunk is still
  retrievable [12].
- **Give each document a stable ID and a version,** so an older revision never overwrites a newer one [10].
- **Set a freshness interval per source,** tighter for what changes fast. In this module: each source's `Next check`
  (its knowledge base's `Freshness` after its last check), and `catalyst inbox` lists a source past it as overdue.

## 8. Classify and protect

OWASP's RAG security cheat sheet [2] is the most authoritative of the sources found; the rest agree with it.

- **Classification and permissions on every chunk,** carried from the document, never lost in chunking [2]. In this
  module: every document and pointer is marked, the effective classification is the higher of its own and its
  knowledge base's (KB-L1), and each chunk in `chunks.jsonl` carries it.
- **Enforce at retrieval time, before similarity search;** filtering in the interface after retrieval is not secure
  [2][13].
- **Separate what a single index cannot filter reliably:** distinct collections per classification or tenant [2]. In
  this module: one build per ceiling (`Ceiling`), never one build mixing levels a reader may not all see.
- **Inherit the source's permissions,** and add document rules only for exceptions [13].
- **Retrieved text is data, not instructions.** Delimit it, scan it for injection markers and invisible Unicode, and
  restate the system's instructions after it [2].
- **Embeddings are sensitive:** they can leak their text. Encrypt them at rest, restrict write access to the build
  pipeline, rate-limit queries, and don't return similarity scores [2].
- **Propagate deletions** to chunks, embeddings and caches; scope caches by permission [2].
- **Mind where text is embedded.** A hosted embedding service receives the text. In this module: `Hosted embedding up
  to` caps what may leave the machine; above it, only a local model.
- **Fail closed:** when retrieval fails, don't let the model answer from its own memory as if it had sources [2].

## 9. Evaluate before and after every build

- **A golden set:** 50–100 questions with their verified answers and the passages that support them, written or
  checked by people who know the domain [4][14].
- **Measure retrieval and generation separately.** Retrieval: Recall@k (are the right passages in at all?),
  Precision@k, MRR or nDCG (are they near the top?) [14][15]. Generation: faithfulness (grounded in what was
  retrieved), answer relevancy, context precision and recall, as RAGAS defines them [4][14].
- **Calibrate model judges against people:** they prefer longer answers, are swayed by position and favour their own
  outputs [14].
- **Faithful is not correct:** a stale document faithfully summarised gives an answer that is grounded and wrong [15].
  Freshness (§7) is part of quality.
- **Run the set on every build, and compare.** A build whose scores drop is not published. In this module: the
  results go into the build's Verification, which `Published` requires.

## 10. Add a graph only where questions follow links

- Graph-based retrieval helps questions that follow relationships: multi-hop ("which customers of X also buy from
  Y"), cross-entity, whole-corpus [16]. For single-fact lookups, manuals and FAQs, plain retrieval is cheaper and
  usually as good [16].
- Recent benchmarks report graph retrieval often under-performing plain retrieval on many real tasks, and helping
  mainly on multi-hop questions [16]. Its quality follows the graph's: model entities and relations first [16].

## 11. Operate it

- Log each request with the document IDs retrieved and the access decisions, without the raw text by default [2].
- Alert on unusual retrieval patterns, repeated injection attempts and access violations [2].
- Red-team the pipeline: poisoned documents, cross-classification leakage, stale permissions [2].
- Name an owner and a schedule: a knowledge base nobody maintains decays. In this module: its curator, its
  Maintenance section, and, if adopted, a workflow governing its builds (`GUIDE.md`, "Governance").

## 12. Checklist

| Practice | Where this module carries it |
|---|---|
| Questions and scope written first | `KB-` Scope; the evaluation set in Verification |
| Authoritative, traceable sources | `KSOURCE-`; front matter `source`, `imported`; manifest checksums |
| Documents written to stand alone | `GUIDE.md` "A document"; `KB.md` and directory READMEs as the index |
| Chunking tested, not assumed | the build's `chunking` in `manifest.json`; Verification |
| Hybrid retrieval, reranking | the consumer's choice; `chunks.jsonl` keeps text for keyword search |
| Embedding model pinned, never mixed | `Embedding model`; a new model is a new major build |
| Freshness by hash, deletions handled | `/kb-refresh`, `Next check`, overdue in the inbox; builds superseded |
| Classification on every chunk, enforced before search | KB-L1; `Ceiling`; one build per ceiling |
| Embedding never leaves the machine above its allowance | `Hosted embedding up to` |
| Evaluation on every build | the `Published` guard: Verification written |
| Owner and schedule | `Curator`, Maintenance, the governing workflow |

## 13. Special case: a knowledge base built from a catalyst criterion

A criterion is unusually good material: every entity has a permanent ID (never reused, L7), typed fields, a status
from a declared workflow, explicit links to the rules it rests on, and a journal saying who changed what, when and
why. Most of what §2–§8 ask for is already there; the work is not to lose it.

### Read through catalyst, not the files

Use catalyst's machine-readable views, whose shapes are described by JSON Schemas (catalyst's `spec/`), rather than
parsing markdown:

| What | Command |
|---|---|
| Every rule, domain, artifact and type, with fields and links | `catalyst graph --json` |
| One entity: its fields, sections, links, what links to it, its journal | `catalyst view <id> --json` |
| One type's entities | `catalyst list <type> --json` |
| One entity's history, with contents | `catalyst journal show --artifact <id> --content --json` |
| What rests on a rule, transitively | `catalyst impact <id> --json` |

These stay stable across kernel releases (a change to a schema is a release note), where the files' layout may not.

### One entity, one parent; its sections, the children

- **The unit is the entity:** one rule (its heading and text), one artifact. Its sections (Description, Acceptance,
  Root cause…) are the child chunks; the entity is the parent handed to the model (§4).
- **The context comes from the fields, without a model.** Prepend to each chunk a line built from its metadata:
  `REQ-000084 · Requirement · Completed · domain STRUCTURE · targets fw-STRUCTURE-000012: <title>`. It is contextual
  retrieval at no generation cost, and the ID makes keyword search exact.
- **Metadata on every chunk:** ID, type, status and whether it is closed, domain, a rule's strength and whether it is
  retired, the IDs it targets and those that cite it, its file, the criterion's commit and its kernel version.

### Keep what is retired, and label it

Retired rules and closed artifacts stay in the criterion (L7); keep them in the knowledge base too, with their status
in metadata, and filter to the live ones by default. "Why was this rule retired?" is a question a criterion answers
well.

### Links are a graph already

`links` and `mentions` (from `catalyst graph`) are the edges: rules to domains, work to the rules it targets, steps to
their requirements. Multi-hop questions ("which open work rests on a rule of this domain?") follow them directly; for
"what would a change reach", `catalyst impact` already answers. A graph store is worth it only for questions catalyst's
views don't answer (§10).

### Leave out what is not knowledge, or not yours to share

- **Generated files:** indexes (`rules.md`, `*s.md` type indexes), backlogs and composed documents
  (`CODE-OF-CONDUCT.md`, `Rules-of-Rules.md` are composed from the kernel and the modules): they duplicate entities and
  would be retrieved twice. Take the kernel's and modules' documents from their own repositories instead.
- **Templates and catalogs:** they describe the shape of entities, not knowledge.
- **People:** `IAM/users/users.json` holds names and git usernames. Leave it out, or treat it as `Restricted`; an
  entity's `Signed-off-by` is metadata, kept only where the knowledge base's classification allows it.
- **History with care:** past versions may hold what was later redacted (`catalyst journal redact`); never re-import
  a redacted blob, and prefer current versions unless the questions are about change.
- **The runtime:** `.venv`, `bin/`, `.criterion` internals.

### Classify the criterion as a source

A criterion repository is usually private: start from `Internal`. Reconciliation cases and code analyses may quote
code and findings: `Confidential` is often right. Record one `KSOURCE-` per criterion (or per area of it when their
classifications differ), with the mark its entities get.

### Refresh from the journal

The journal is a change log with content hashes: every change names the files it touched with their blobs before and
after. Rebuild incrementally from the entries since the last build's content commit (the build's `Content commit` is
the criterion's commit), instead of re-reading everything; a deleted or renamed entity shows up there too. A shared
criterion's `catalyst share status` (or a server's change counter) tells cheaply whether anything changed.

### Cite IDs

Answers cite entity IDs and rule IDs. They never change and never get reused, so a citation stays valid; a reader
checks it with `catalyst view <id>`.

### Reflective knowledge bases

A reflective knowledge base (the Cantica ecosystem: the kernel, its modules, its plugins) takes the same approach
with the repositories and their releases as sources, plus catalyst's own criterion where its classification allows:
rebuilt at each release (rr-KB-005), each build naming the kernel version it describes.

## Sources

1. Anthropic, *Introducing Contextual Retrieval* (September 2024): https://www.anthropic.com/news/contextual-retrieval
2. OWASP, *RAG Security Cheat Sheet*: https://cheatsheetseries.owasp.org/cheatsheets/RAG_Security_Cheat_Sheet.html
3. Analytics Vidhya, *LLMs.txt Explained*: https://www.analyticsvidhya.com/blog/2025/03/llms-txt/ (see also ZenML,
   https://www.zenml.io/blog/llms-txt)
4. Prem AI, *Building Production RAG: Architecture, Chunking, Evaluation & Monitoring (2026)*:
   https://www.premai.io/blog/building-production-rag-architecture-chunking-evaluation-monitoring-2026-guide/
5. Future AGI, *Evaluating RAG Chunking Strategies in 2026*: https://futureagi.com/blog/evaluating-rag-chunking-strategies-2026/
6. *Evaluating Chunking Strategies for RAG in Oil and Gas Enterprise Documents* (arXiv 2603.24556):
   https://arxiv.org/pdf/2603.24556; and Atlan, *Chunking Strategies for RAG*: https://atlan.com/know/chunking-strategies-rag/
7. *RAG in Practice: Strategies & Production Best Practices* (DEV Community):
   https://dev.to/ifnodoraemon/retrieval-augmented-generation-rag-in-practice-strategies-production-best-practices-2ccn
8. *Embedding Models in Production: Selection, Versioning, and the Index Drift Problem*:
   https://tianpan.co/blog/2026-04-09-embedding-models-production-versioning-index-drift
9. *Embedding Model Churn: When Your Provider Silently Invalidates Your Entire Vector Index*:
   https://tianpan.co/blog/2026-05-07-embedding-model-churn-vector-index-invalidation
10. *Incremental Indexing and Freshness Strategies for Enterprise Search and RAG Corpora*:
    https://www.geosurge.ai/corpus/gen-4165/indexed-search/incremental-indexing-and-freshness-strategies-for-enterprise-search-and-rag-corpora.html
11. *RAG Knowledge Base Freshness: The Staleness Problem Teams Solve Last*:
    https://tianpan.co/blog/2026-04-20-rag-knowledge-base-freshness-index-rot
12. Oracle, *How to detect RAG index drift: deleted docs, stale chunks and duplicate embeddings*:
    https://blogs.oracle.com/developers/how-to-detect-rag-index-drift-deleted-docs-stale-chunks-and-duplicate-embeddings
13. Kiteworks, *RAG Pipeline Security Best Practices for 2026*:
    https://www.kiteworks.com/cybersecurity-risk-management/rag-pipeline-security-best-practices/
14. Anyscale, *RAG evaluation*: https://docs.anyscale.com/rag/evaluation; and Prem AI, *RAG Evaluation: Metrics,
    Frameworks & Testing (2026)*: https://blog.premai.io/rag-evaluation-metrics-frameworks-testing-2026
15. Future AGI, *How to Evaluate RAG Systems*: https://futureagi.substack.com/p/how-to-evaluate-rag-systems-the-complete
16. *When to use Graphs in RAG* (ICLR 2026): https://www.iclr.cc/virtual/2026/poster/10007992; and Cognee, *GraphRAG vs
    RAG*: https://www.cognee.ai/blog/deep-dives/graphrag-vs-rag
