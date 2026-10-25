# Migration 0.107.0 (module 0.6.0): a source's kind and visibility

> Applies when a deployment of this module is synchronized to module `0.6.0`. Never re-run once applied.

## What changed

1. **`KSOURCE-` gains two optional fields**: `Kind` (`Repository` / `Folder` / `Site` / `Criterion`) and
   `Visibility` (`Public` / `Private`). A `Criterion` source is a catalyst project's criterion, `Location`
   `criterion:<project name>`, read only through catalyst's CLI (`kbm-MODEL-000002-I3JORbi9`).
2. **A private source is a floor** (`rr-KB-002`): every document read from a `Private` source is at least `Internal`,
   whatever its own mark. A build refuses to run when a source's `Visibility` and its host disagree.
3. **`/kb-setup`, `/create knowledge-source` and `/kb-import`** set, check and apply them.

## Steps

1. **Refresh the module** (`catalyst module upgrade knowledge-base`) and recompose `CODE-OF-CONDUCT.md`,
   `Rules-of-Rules.md` and `Taskfile.common.yml`; `catalyst migrate --apply --template KSOURCE` adds the generated
   template with the new fields as the type's next version (optional; a frozen template is never rewritten, INV-23).
2. **Judgment, optional:** for each existing source, with its curator's agreement, record its `Kind` and its
   `Visibility` as its host shows it (a source whose `Location` is `criterion:<name>` is `Criterion`, and `Private`
   unless its criterion repository is public), journaled (`--action sync`, intent naming this migration). Leaving
   them out changes nothing, but a source left without `Visibility` gets no floor: record it for every private one.
3. **Re-mark what a private source brought in:** a document imported from a source now recorded `Private` and marked
   `Public` is raised to `Internal` (`/kb-classify`), and a build that holds one is superseded by the next build.
4. **Version.** Record module `0.6.0` in `DEPLOYMENT.md`.
