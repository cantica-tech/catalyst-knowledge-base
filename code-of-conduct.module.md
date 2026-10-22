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
(KB-L1, `rr-KB-001` to `rr-KB-006`). How a knowledge base's repository, its
documents, its pointers and its builds are laid out: the module's `GUIDE.md`
(`modules/knowledge-base/GUIDE.md` in the criterion).

## 4. Slash-command entry points

- `/kb-setup project|reflective <title>` — establish a knowledge base: its
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

When the user enters `/kb-setup <kind> <title>`:

1. Ask what it must make known and for whom, and its classification; if no
   rule says it, propose one (`catalyst id next-rule`) and add it only with
   the user's assent. A reflective knowledge base draws only on the Cantica
   ecosystem (`rr-KB-005`): say so.
2. Run `catalyst new KB --title "<title>" --field Serves=<rule IDs>
   --field Kind=<Project|Reflective> --field Classification=<level>
   --field "Default classification=<level>" --field "Hosted embedding up to=<level|None>"
   --field Repository=<url or path> --field Freshness=<interval>
   [--field Curator=<user>] --command /kb-setup --tier kb --as <signer>`.
   Propose `Default classification` no lower than `Classification`, and
   `Hosted embedding up to` no higher than `Internal`, unless the user decides
   otherwise.
3. If the deployment has no `Knowledge Curator` role, offer to add it:
   `catalyst role add "Knowledge Curator" --action /kb-setup --action /kb-import
   --action /kb-classify --action /kb-build --action /kb-refresh --action /kb-audit
   --action /create --action /status --action /journal`, then
   `catalyst user assign-role <user> "Knowledge Curator"` — each only with the
   user's assent.
4. Create the repository as `GUIDE.md` lays it out (`KB.md`, `docs/`,
   `pointers/`, `builds/`), with `KB.md` naming the `KB-` ID, its
   classification and its policy. A new repository is the user's to create or
   push: ask before any push (L3).
5. Write the Scope and Structure, then set it `Established`. Its Policy and
   Maintenance are written next, and it becomes `Active` (`rr-KB-001`).

When the user enters `/create knowledge-source <title>` or `/create-knowledge-source <title>`, run
`catalyst new KSOURCE --title "<t>" --field "Knowledge base=<KB-id>"
--field Location=<url or path> [--field Classification=<level|Unmarked>]
[--field Access=<how>] --command /create-knowledge-source --tier kb --as <signer>`
(it cites the source back in the knowledge base's `Sources`), then write its
Description and Import. Refuse a source outside the Cantica ecosystem for a
reflective knowledge base.

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
`broken`. For each `KSOURCE-` of it, set `Checked` and, with `catalyst status
set`, `Stale` or `Broken` when it is, or back to `Active`. Report what changed;
never delete a pointer — a broken one stays, marked, until a person removes
or replaces it.

When the user enters `/kb-build <KB-id> [<ceiling>]`:

1. Refuse unless the knowledge base is `Active` and `/kb-audit` finds no
   unmarked document or pointer.
2. Take every document and pointer whose effective classification is at or
   below the ceiling (default: the knowledge base's `Classification`); list
   what the ceiling leaves out.
3. Embed each with the backend its effective classification allows:
   a hosted service only up to `Hosted embedding up to`, a local model above
   it. Never send a document to a backend above its allowance; if no allowed
   backend is available, stop and say so.
4. Write the build as `GUIDE.md` lays it out (`manifest.json`,
   `chunks.jsonl`, `vectors.parquet`) with its version, its classification
   (the highest effective classification it contains) and the repository's
   commit.
5. Record it: `catalyst new KBUILD --title "<KB title> <version>"
   --field "Knowledge base=<KB-id>" --field Version=<version>
   --field Ceiling=<level> --field Classification=<level>
   --field "Embedding model=<model, where it ran>" --field "Content commit=<sha>"
   --field Location=<where> --command /kb-build --tier build --as <signer>`,
   set it `Built`, and write its Contents.
6. Publishing is a separate step: write its Verification, then set it
   `Published`, and the build it replaces `Superseded`. Publishing anywhere
   outside this machine needs the user's yes (L3).

When the user enters `/kb-audit [<KB-id>]`, report for each knowledge base:
documents and pointers with no `classification`; pointers not checked within
its `Freshness`, `moved` or `broken`; sources `Stale` or `Broken`; documents
whose effective classification is above a `Published` build's ceiling but
inside it; builds older than the repository's latest commit. Change nothing.
