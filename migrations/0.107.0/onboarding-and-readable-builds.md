# Migration 0.107.0 (module 0.5.0): onboarding, and builds any model can read

> Applies when a deployment of this module is synchronized to module `0.5.0` (`0.4.x` → `0.5.0`); the module needs
> kernel `0.107.0` or later. Never re-run once applied.

## What changed

1. **`KB-` gains three optional fields** that onboarding sets: `Basis` (`Criterion` / `Documents`), `Document base`
   (`Static` / `Dynamic`) and `Target` (`Folders` / `Vector database`).
2. **`KBUILD-` gains `Target`**, and `Embedding model` is required only of a vector database build.
3. **`/kb-setup` onboards**: basis and sources (the criterion plus enrichment, or gathering), location and storage,
   target, document base; the Knowledge Curator role is catalyst-users' (kernel 0.106.0), never `catalyst role add`.
4. **`rr-KB-009`, INV-KB-9**: a build, of either target, is readable by any model without catalyst (`GUIDE.md`,
   "A build": a `format` and `target` in its manifest, a `README.md`, the documents beside the chunks).

## Steps

1. **Refresh the module** the ordinary sync way (`catalyst module upgrade knowledge-base`) and recompose
   `CODE-OF-CONDUCT.md`, `Rules-of-Rules.md` and `Taskfile.common.yml`.
2. **Templates.** `catalyst migrate --apply --template KB` and `--template KBUILD` add the generated templates with
   the new fields as the types' next versions (optional; a frozen template is never rewritten, INV-23).
3. **Judgment, optional:** for each existing knowledge base, with its curator's agreement, record what onboarding
   would have decided: `Basis` (`Criterion` if it draws on this deployment's criterion, else `Documents`),
   `Document base` (`Dynamic` unless it is meant to stay as defined), `Target` (`Vector database` for one with
   vector builds). Leaving them out changes nothing.
4. **Version.** Record module `0.5.0` in `DEPLOYMENT.md`; journal the files touched (`--action sync`, intent naming
   this migration).
