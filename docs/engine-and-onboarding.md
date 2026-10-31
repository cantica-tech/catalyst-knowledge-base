# The KBM engine and onboarding: design and technology research

Status: **research and proposal** (2026-10-24). Decided the same day: the engine is its own repository,
`catalyst-kb-engine` (Apache-2.0, governed by catalyst), on Pydantic AI; the KB record gains `Basis`,
`Document base` and `Target`; a build is readable by any model without catalyst (KBM 0.5.0, `rr-KB-009`). It records the owner's design for the knowledge base
module's (KBM) engine and onboarding, what catalyst requires of it, and the open-source Python technologies that could
run it, checked online on 2026-10-24.

## 1. The owner's design

- **An engine** builds a knowledge base's output by running a **workflow**. The output, the **target**, is either an
  **organized markdown folder structure** or a **vector database**.
- **Onboarding always happens**, whether KBM is a criterion's only module or sits beside other modules (KB-L2).
- **Two kinds of document base:**

  | KBM                                        | Its document base                                                     | Default |
  | ------------------------------------------ | --------------------------------------------------------------------- | ------- |
  | **Criterion-based** (beside other modules) | the current project's criterion, optionally enriched by other sources | dynamic |
  | **Documents-based** (alone)                | the sources gathered during onboarding                                | chosen  |

- **Static or dynamic:** a static document base is defined once and never changes; a dynamic one grows and shrinks at
  will. Dynamic is the default for a criterion-based knowledge base, because a criterion is always in flux.

### Onboarding

| Step | Criterion-based                                                                                  | Documents-based                             |
| ---- | ------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| 1    | select document sources that **enrich** the criterion                                            | select the document sources (**gathering**) |
| 2    | define the location and storage strategy: the knowledge base's folder organization for documents | same                                        |
| 3    | define the target: organized documents in folders, or a vector database                          | same                                        |

Both kinds also record whether the document base is **static or dynamic** (dynamic by default for criterion-based).

## 2. What catalyst requires of the engine

These come from catalyst's laws and KBM's own, and narrow the technology choice more than any feature list does.

| Requirement                                               | From                      | Consequence for the engine                                                                                                    |
| --------------------------------------------------------- | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Every change is journaled as it happens                   | L1                        | each step writes files the journal records; no hidden state only the engine knows                                             |
| A person decides; the agent proposes                      | L4, `rr-KB-006`           | onboarding's choices (sources, layout, target) and every classification are **human-approved** pauses, which may last days    |
| Model-agnostic, and no module knows an agent              | L9                        | the engine never assumes one model provider or one coding agent                                                               |
| Classification only rises; embedding within the allowance | KB-L1, `rr-KB-004`        | `Restricted`/`Confidential` documents are embedded by a **local** model; hosted embedding only up to `Hosted embedding up to` |
| Alone or beside any module                                | KB-L2, INV-30             | the engine reads a criterion as a source without knowing which module wrote it                                                |
| A module carries no code today                            | `MODULE-SPECIFICATION.md` | the engine is code, so it lives in its own package (as catalyst-users does), which KBM's commands call                        |
| Local-first, single machine                               | catalyst's CLI            | no server to run for the common case; a durable-execution server is optional, never required                                  |
| Dynamic document bases                                    | the owner's design        | builds are **incremental**: re-process only what changed since the last build (content hashes)                                |

The decisive point: **onboarding is a long, human-paced process**, and a build is a pipeline with a few judgment
steps. What must survive a pause is the **state of the workflow**, and in catalyst that state belongs in the criterion
(the `KB-`, `KSOURCE-` and `KBUILD-` records and a `WORKFLOW-`), journaled, not in a framework's private store.

## 3. The candidates

Checked on GitHub on 2026-10-24 (license, stars, last push). "Fit" is this document's judgment against §2.

### Agent and workflow frameworks

| Project                                                                        | License                        | Stars                                                                      | Active              | What it is                                                                                                                                                                                                                                                  | Fit                                                    |
| ------------------------------------------------------------------------------ | ------------------------------ | -------------------------------------------------------------------------- | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| [Pydantic AI](https://github.com/pydantic/pydantic-ai) (with `pydantic-graph`) | MIT                            | 20.8k                                                                      | v2.61.0, 2026-10-24 | typed agents, structured outputs, model-agnostic, embeddings, MCP, deferred tools (human approval); `pydantic-graph` is a typed state machine with **no dependency on pydantic-ai or an LLM**; durable execution through Temporal, DBOS, Prefect and others | **high**                                               |
| [LangGraph](https://github.com/langchain-ai/langgraph)                         | MIT                            | 43.8k                                                                      | 1.2.17, 2026-10-24  | graph of stateful nodes; checkpointers (in-memory, SQLite, Postgres) run locally without its platform; interrupts for human-in-the-loop                                                                                                                     | high, heavier (LangChain ecosystem)                    |
| [Apache Burr](https://github.com/apache/burr)                                  | Apache-2.0                     | 2.6k                                                                       | 2026-10-21          | explicit state machine with persistence, human-in-the-loop, a tracking UI                                                                                                                                                                                   | medium (small community)                               |
| [LlamaIndex Workflows](https://github.com/run-llama/llama-agents)              | MIT                            | 0.5k (part of [LlamaIndex](https://github.com/run-llama/llama_index), MIT) | 2026-10-23          | event-driven, async step workflows; strong for retrieval                                                                                                                                                                                                    | medium                                                 |
| [Haystack](https://github.com/deepset-ai/haystack)                             | Apache-2.0                     | 26.8k                                                                      | 2026-10-24          | pipelines of components plus agents; strong ingestion and RAG                                                                                                                                                                                               | medium (a pipeline framework more than a workflow one) |
| [Microsoft Agent Framework](https://github.com/microsoft/agent-framework)      | MIT                            | 14.3k                                                                      | 2026-10-24          | AutoGen and Semantic Kernel's successor, 1.0; graph workflows, multi-provider, MCP and A2A                                                                                                                                                                  | medium (enterprise-weight)                             |
| [CrewAI](https://github.com/crewAIInc/crewAI)                                  | MIT                            | 60.1k                                                                      | 2026-10-24          | role-based crews of agents, plus Flows                                                                                                                                                                                                                      | low (multi-agent roles we don't need)                  |
| [Agno](https://github.com/agno-agi/agno)                                       | Apache-2.0                     | 42.9k                                                                      | 2026-10-24          | lightweight agents with memory and storage                                                                                                                                                                                                                  | low–medium                                             |
| [Google ADK](https://github.com/google/adk-python)                             | Apache-2.0                     | 22.0k                                                                      | 2026-10-24          | agent development kit, workflow agents                                                                                                                                                                                                                      | low–medium (Google-leaning)                            |
| [OpenAI Agents SDK](https://github.com/openai/openai-agents-python)            | MIT                            | 30.3k                                                                      | 2026-10-22          | agent loop, handoffs, guardrails                                                                                                                                                                                                                            | low (OpenAI-leaning)                                   |
| [Strands Agents](https://github.com/strands-agents/harness-sdk)                | Apache-2.0                     | 8.9k                                                                       | 2026-10-24          | model-driven agents (AWS)                                                                                                                                                                                                                                   | low                                                    |
| [smolagents](https://github.com/huggingface/smolagents)                        | Apache-2.0                     | 30.0k                                                                      | 2026-10-22          | minimal code-writing agents                                                                                                                                                                                                                                 | low (agents, not workflows)                            |
| [DSPy](https://github.com/stanfordnlp/dspy)                                    | MIT                            | 38.8k                                                                      | 2026-10-23          | programming and optimizing LM prompts                                                                                                                                                                                                                       | complement (optimizing a step), not an orchestrator    |
| [BeeAI Framework](https://github.com/i-am-bee/beeai-framework)                 | Apache-2.0                     | 3.4k                                                                       | 2026-10-23          | agents and workflows (Linux Foundation)                                                                                                                                                                                                                     | low                                                    |
| [CAMEL](https://github.com/camel-ai/camel)                                     | Apache-2.0                     | 18.0k                                                                      | 2026-10-19          | multi-agent research framework                                                                                                                                                                                                                              | low                                                    |
| [Letta](https://github.com/letta-ai/letta)                                     | Apache-2.0                     | 25.2k                                                                      | 2026-10-18          | stateful agents with long-term memory, a server                                                                                                                                                                                                             | low                                                    |
| [Mirascope](https://github.com/Mirascope/mirascope)                            | MIT                            | 1.5k                                                                       | 2026-10-24          | thin LLM call toolkit                                                                                                                                                                                                                                       | low                                                    |
| [Atomic Agents](https://github.com/Eigenwise/atomic-agents)                    | MIT                            | 6.3k                                                                       | 2026-10-24          | small, schema-driven agents                                                                                                                                                                                                                                 | low                                                    |
| [AutoGen](https://github.com/microsoft/autogen)                                | code MIT, repository CC-BY-4.0 | 61.5k                                                                      | moving to archived  | its README advises against production use and points to Microsoft Agent Framework                                                                                                                                                                           | **excluded**                                           |

### Durable execution (optional, for long pauses outside catalyst's records)

| Project                                                                                                     | License                                           | Note                                                                           |
| ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------ |
| [Temporal](https://github.com/temporalio/temporal) ([Python SDK](https://github.com/temporalio/sdk-python)) | MIT                                               | needs a Temporal server; integrated with Pydantic AI and the OpenAI Agents SDK |
| [DBOS Transact](https://github.com/dbos-inc/dbos-transact-py)                                               | MIT                                               | a library over a database; integrated with Pydantic AI                         |
| [Prefect](https://github.com/PrefectHQ/prefect)                                                             | Apache-2.0                                        | Python flows, optional server; integrated with Pydantic AI                     |
| [Restate](https://github.com/restatedev/restate)                                                            | **server: Business Source License 1.1** (SDK MIT) | **not open source** in the OSI sense: excluded                                 |

### Building the targets

| Need                                               | Candidates (license)                                                                                                                                                                                                                                  |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Parsing documents (PDF, Office, HTML, tables, OCR) | [Docling](https://github.com/docling-project/docling) (MIT, LF AI & Data, 69.5k stars); [Unstructured](https://github.com/Unstructured-IO/unstructured) (Apache-2.0)                                                                                  |
| Vector store, embedded and local                   | [LanceDB](https://github.com/lancedb/lancedb) (Apache-2.0); [Chroma](https://github.com/chroma-core/chroma) (Apache-2.0); [FAISS](https://github.com/facebookresearch/faiss) (MIT); [Qdrant](https://github.com/qdrant/qdrant) (Apache-2.0, a server) |
| The portable build                                 | KBM's own format (`manifest.json`, `chunks.jsonl`, `vectors.parquet`, `GUIDE.md`); LanceDB reads and writes Arrow/Parquet natively                                                                                                                    |
| Embeddings                                         | hosted through the model-agnostic agent layer up to the allowance; local models for anything above it (KB-L1)                                                                                                                                         |

## 4. Is Pydantic AI a good choice?

**Yes, for the agentic steps, with `pydantic-graph` for the workflow, and catalyst's own records as the durable
state.**

- **Why it fits.** MIT; model-agnostic, with local models, which KB-L1 needs for classified documents; typed,
  validated structured outputs, so a proposed classification, folder layout or source list is data a person approves,
  not free text; deferred tools for human approval (L4); embeddings and MCP built in; very active (v2.61.0 the day
  this was written).
- **The workflow.** `pydantic-graph` is a typed state machine that needs neither pydantic-ai nor an LLM, so the
  deterministic steps (gather, parse, chunk, write the folders, embed, write the build) and the agentic ones (propose
  sources, layout, classification) sit in one graph.
- **What it does not give for free.** Pausing for days and resuming in another process. Its durable execution relies
  on an external engine (Temporal, DBOS, Prefect), which is too heavy to require for a local CLI, and the current
  `pydantic-graph` documentation does not describe persisting a run. **Proposed answer:** make each step idempotent and
  file-producing, and keep the workflow's position in the criterion: the onboarding's answers in the `KB-` record and
  its `WORKFLOW-`, each source as a `KSOURCE-`, each build as a `KBUILD-`, all journaled. Resuming is reading those
  records, which catalyst already does. A durable engine stays an option for hosted, multi-machine builds.
- **The alternative.** LangGraph, if built-in pause and resume (SQLite checkpointer, interrupts) is wanted inside the
  framework. It is mature and MIT, but brings the LangChain ecosystem's weight, and its checkpoints are a second store
  of state beside the criterion, which L1 would then have to journal.

## 5. Proposed shape

```text
KBM commands (/kb-setup, /kb-import, /kb-build, …)        module: prose, types, workflow
        │ call
catalyst-kb-engine (new package, Apache-2.0)                      code, its own repository and releases
  ├─ onboarding graph (pydantic-graph)
  │    select sources → layout and storage → target → static|dynamic   (each a person's approval)
  ├─ build graph (pydantic-graph)
  │    diff the document base → parse (Docling) → classify (Pydantic AI, approved)
  │    → organize folders  ─or─  chunk → embed (local or hosted, per KB-L1) → write the build (Parquet; LanceDB)
  └─ state: the criterion's KB-, KSOURCE-, KBUILD- and WORKFLOW- records, journaled through catalyst's CLI
```

- **Criterion-based**: the criterion itself is the first source, read through catalyst's CLI (`list`, `view`,
  `graph`), never by knowing which module wrote it; enrichment adds `KSOURCE-`s.
- **Documents-based**: onboarding's first step gathers the sources.
- **Dynamic**: every build re-reads the sources and processes only what changed (content hashes); **static**: the
  document base is frozen after onboarding and a build is a pure function of it.

## 6. Decided (formerly the open questions for the owner)

1. **Where the engine lives**: its own repository, `cantica-tech/catalyst-kb-engine` (Apache-2.0, governed by
   catalyst), as catalyst-users is for people; modules still carry no code.
2. **Static or dynamic as a field**: yes, the KB record's `Document base: Static | Dynamic` field, since KBM 0.5.0
   (with `Basis` and `Target`).
3. **Which models by default**: a local embedding model, `nomic-embed-text` (the owner, 2026-10-25); a hosted one
   only when configured, up to the allowance (catalyst-kb-engine's design `01` §10.3).
4. **Which framework**: Pydantic AI, with catalyst's records (the criterion) as the durable state.

## Sources

- Licenses, stars and last activity: each repository's GitHub API record, read 2026-10-24.
- Pydantic AI: [repository](https://github.com/pydantic/pydantic-ai), [graph documentation](https://pydantic.dev/docs/ai/graph/graph/)
  (pydantic-graph has no dependency on pydantic-ai), [v1 announcement](https://pydantic.dev/articles/pydantic-ai-v1),
  [durable execution in production](https://cadenya.com/other-content/pydantic-ai-agents-in-production).
- LangGraph: [persistence](https://docs.langchain.com/oss/python/langgraph/persistence).
- AutoGen's status: its [README](https://github.com/microsoft/autogen).
- Restate's license: its [LICENSE](https://github.com/restatedev/restate) (Business Source License 1.1).
- Overviews (secondary, version figures unreliable): [Langfuse](https://langfuse.com/blog/2025-03-19-ai-agent-comparison),
  [Arize](https://arize.com/guides/ai-agent-handbook/agent-frameworks), [Turing Post on RAG licenses](https://www.turingpost.com/p/rag-tools),
  [Prefect](https://www.prefect.io/).
