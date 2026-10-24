# Code of Conduct — `knowledge-base` module contribution

> Composed into each deployment's `CODE-OF-CONDUCT.md` (`MODULE-SPECIFICATION.md`
> §6.2): the `## 3.` section below is inserted at the end of the deployed §3,
> and the `## 4.` section at the end of the deployed §4, each under a
> `### From module knowledge-base` heading.

## 3. Standard document types

| Type | Folder | ID prefix | Grounded in |
|---|---|---|---|
| Knowledge base | `knowledge-bases/` | `KB-` | the rules it serves (`Serves`) |
| Knowledge source | `kb-sources/` | `KSOURCE-` | its knowledge base (`Knowledge base`) |
| Knowledge build | `kb-builds/` | `KBUILD-` | its knowledge base (`Knowledge base`) |

What each needs and when it closes: `Rules-of-Rules.md`, the module's part
(KB-L1, KB-L2, `rr-KB-001` to `rr-KB-009`). How a knowledge base's repository, its
documents, its pointers and its builds are laid out: the module's `GUIDE.md`
(`modules/knowledge-base/GUIDE.md` in the criterion).

## 4. Slash-command entry points

- `/kb-setup project|reflective <title>` — onboard a knowledge base: its basis
  and sources, its location and storage, its target, its document base, then its
  `KB-` record, its repository, its policy and its curator.
- `/create knowledge-source <title>` (`/create-knowledge-source`) — record a source a knowledge
  base draws from.
- `/kb-import <KSOURCE-id>` — bring a source's documents into the knowledge
  base, each marked by its policy.
- `/kb-classify <path> <level>` — mark one document or pointer.
- `/kb-refresh <KB-id>` — check its pointers and sources, and mark what is
  stale or broken.
- `/kb-build <KB-id> [<ceiling>]` — build a versioned vector database within a
  ceiling, recorded as a `KBUILD-`.
- `/kb-audit [<KB-id>]` — report what is unmarked, stale or over its ceiling.
  Changes nothing.

A classification is one of `Public` < `Internal` < `Confidential` <
`Restricted`. A document's or pointer's **effective classification** is the
higher of its own mark and its knowledge base's `Classification`; an unmarked
one takes the knowledge base's `Default classification` first (`rr-KB-002`).
Never lower a mark to fit a build or a backend: leave the document out.

Only a `Knowledge Curator` or an `Admin` runs `/kb-setup`, `/kb-import`,
`/kb-classify` and `/kb-build`; anyone may run `/kb-refresh` and `/kb-audit`
(`rr-KB-006`).

When the user enters `/kb-setup <kind> <title>`, onboard it (`GUIDE.md`, "Onboarding"). Onboarding always
happens, whether this module is alone or beside others; every choice is the user's, proposed, never assumed:

1. **What it serves.** Ask what it must make known and for whom, and its
   classification; if no rule says it, propose one (`catalyst id next-rule`)
   and add it only with the user's assent. A reflective knowledge base draws
   only on the Cantica ecosystem (`rr-KB-005`): say so.
2. **Its basis and sources.** Propose `Criterion` when another module governs
   this deployment (`catalyst module list`), else `Documents`:
   - *Criterion*: the project's criterion is its first source, read through
     catalyst's CLI (`list`, `view`, `graph`), whichever modules wrote it; ask
     which other sources enrich it.
   - *Documents*: gather its sources with the user: what they are, where they
     live, how they are reached, and their classification.
3. **Its location and storage strategy**: where its repository lives, and how
   its documents are organized in folders (`GUIDE.md`, "The repository"); the
   folder plan goes into its Structure.
4. **Its target**: `Folders` (organized markdown documents) or
   `Vector database` (chunks and vectors); both are readable by any model
   (`rr-KB-009`).
5. **Its document base**: `Dynamic` (grows and shrinks; the default for a
   criterion-based knowledge base, a criterion always being in flux) or
   `Static` (defined once; it takes no new source after its first build).
6. Run `catalyst new KB --title "<title>" --field Serves=<rule IDs>
   --field Kind=<Project|Reflective> --field Classification=<level>
   --field "Default classification=<level>" --field "Hosted embedding up to=<level|None>"
   --field Repository=<url or path> --field Freshness=<interval>
   --field Basis=<Criterion|Documents> --field "Document base=<Static|Dynamic>"
   --field Target=<Folders|Vector database>
   [--field Curator=<user>] --command /kb-setup --tier kb --as <signer>`.
   Propose `Default classification` no lower than `Classification`, and
   `Hosted embedding up to` no higher than `Internal`, unless the user decides
   otherwise. Then record each source of step 2 (`/create knowledge-source`),
   the criterion first for a criterion-based one (`Location` `criterion:<the
   project's name>`).
7. **Its curator.** People and roles are catalyst-users' (kernel 0.106.0): if
   its provider has no Knowledge Curator role, offer its admin to add one
   (`catalyst-users roles provision <a roles document naming Knowledge Curator
   and the commands /kb-setup, /kb-import, /kb-classify, /kb-build,
   /kb-refresh, /kb-audit, /create, /status, /journal>`) and to put the
   curator in it — each only with the user's assent; `catalyst identity
   whoami` shows the result.
8. Create the repository as `GUIDE.md` lays it out (`KB.md`, `docs/`,
   `pointers/`, `builds/`), with `KB.md` naming the `KB-` ID, its
   classification, its policy, its basis, document base and target. A new
   repository is the user's to create or push: ask before any push (L3).
9. Write the Scope and Structure, then set it `Established`. Its Policy and
   Maintenance are written next, and, with its curator named, it becomes
   `Active` (`rr-KB-001`).
10. Offer the governance `GUIDE.md` gives ("Governance"), each only with the
    user's assent (`rr-KB-007`): a `WORKFLOW-` governing `KBUILD` (and one
    governing `KB`), from its Transitions table, made `Active`; and the gate
    that asks two curators to publish a build classified `Confidential` or
    above, added to `IAM/gates.json`, journaled.

When the user enters `/create knowledge-source <title>` or `/create-knowledge-source <title>`, run
`catalyst new KSOURCE --title "<t>" --field "Knowledge base=<KB-id>"
--field Location=<url or path> [--field Classification=<level|Unmarked>]
[--field Access=<how>] --command /create-knowledge-source --tier kb --as <signer>`
(it cites the source back in the knowledge base's `Sources`), then write its
Description and Import. Refuse a source outside the Cantica ecosystem for a
reflective knowledge base, and a new source for a `Static` document base once
its first build exists (a static base is defined once).

When the user enters `/kb-import <KSOURCE-id>`, read the source and its
knowledge base (`catalyst view <id> --json`). For each document taken in,
as its Import section says — copied as markdown, summarised, or kept as a
pointer — write it under `docs/` or `pointers/` in the knowledge base's
repository with its front matter (`GUIDE.md`, "A document" and "A pointer"):
`classification` the source's mark, else the knowledge base's
`Default classification`; `source` the `KSOURCE-` ID; `imported` today. Never
copy a document the source's access does not allow, and never write a secret.
Then report what was added, by directory and by classification.

When the user enters `/kb-classify <path> <level>`, set the `classification`
of that document's or pointer's front matter, and report its effective
classification. Lowering a mark is a decision: say what it was, ask, and
name the builds that contain the document (they keep their classification;
a new build is needed for the change to reach a model).

When the user enters `/kb-refresh <KB-id>`, for each pointer in its
repository, check that its target still answers and whether its content
changed (an HTTP request, or the file's or the repository's state); update
`checked`, and set `status` to `ok`, `moved` (with the new location) or
`broken`. For each `KSOURCE-` of it, set `Checked` to today and `Next check`
to today plus the knowledge base's `Freshness`, journaled, and, with
`catalyst status set`, `Stale` or `Broken` when it is, or back to `Active`.
A source past its `Next check` is listed by `catalyst inbox` as `overdue`. Report what changed;
never delete a pointer — a broken one stays, marked, until a person removes
or replaces it.

When the user enters `/kb-build <KB-id> [<ceiling>]`:

1. Refuse unless the knowledge base is `Active` and `/kb-audit` finds no
   unmarked document or pointer.
2. Take every document and pointer whose effective classification is at or
   below the ceiling (default: the knowledge base's `Classification`); list
   what the ceiling leaves out. A criterion-based knowledge base first reads
   the criterion's current records into its documents; a `Dynamic` one
   processes only what changed since its last build.
3. For a `Vector database` target, embed each with the backend its effective
   classification allows: a hosted service only up to `Hosted embedding up
   to`, a local model above it. Never send a document to a backend above its
   allowance; if no allowed backend is available, stop and say so. A
   `Folders` target embeds nothing.
4. Write the build as `GUIDE.md` lays it out for its target ("A build"), with
   its manifest (format version, version, classification — the highest
   effective classification it contains — and the repository's commit) and
   its `README.md`: readable by any model, without catalyst (`rr-KB-009`).
5. Record it: `catalyst new KBUILD --title "<KB title> <version>"
   --field "Knowledge base=<KB-id>" --field Version=<version>
   --field Target=<Folders|Vector database>
   --field Ceiling=<level> --field Classification=<level>
   [--field "Embedding model=<model, where it ran>"] --field "Content commit=<sha>"
   --field Location=<where> --command /kb-build --tier build --as <signer>`
   (`Embedding model` required of a vector database build),
   set it `Built`, and write its Contents.
6. Publishing is a separate step: write its Verification, then set it
   `Published`, and the build it replaces `Superseded`. Publishing anywhere
   outside this machine needs the user's yes (L3).

When the user enters `/kb-audit [<KB-id>]`, report for each knowledge base:
documents and pointers with no `classification`; pointers not checked within
its `Freshness`, `moved` or `broken`; sources `Stale` or `Broken`; documents
whose effective classification is above a `Published` build's ceiling but
inside it; builds older than the repository's latest commit. Change nothing.
