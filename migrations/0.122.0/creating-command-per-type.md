# Migration 0.122.0 (module 0.7.0): the knowledge base and its builds name their command

> Applies when a deployment of this module is synchronized to module `0.7.0` (`0.6.x` → `0.7.0`). The module still
> needs kernel `0.107.0` or later; the declaration takes effect on kernel `0.122.0` or later (`created_by`,
> fw-STRUCTURE-000026), and an older kernel ignores it and keeps the generic create. Never re-run once applied.

## What changed

1. **`created_by: /<command>`** is declared on two types:
   `KB` → `/kb-setup` (onboarding, which sets the basis, sources, location and target), and
   `KBUILD` → `/kb-build`.
2. For these types a kernel `0.122.0` or later composes **no generic `/create <type>` part**, and `catalyst new
   <PREFIX>` refuses unless it runs with `--command` naming the declared command. Both commands already pass
   `--command`, so their behaviour does not change.
3. `KSOURCE` (`/create knowledge-source`) keeps the generic create: it is not in the module's `commands:` catalogue,
   so it is not declared yet.

## Steps

1. **Refresh the module** the ordinary sync way (`/sync-framework`) and recompose `CODE-OF-CONDUCT.md`.
2. **Check the declared commands are the ones the deployment runs** (kernel `0.122.0` or later). `catalyst spec
   create <type>` gives the declared command's procedure; a knowledge base is created by `/kb-setup`, never by
   `catalyst new KB` alone.
3. **Version.** Record module `0.7.0` in `DEPLOYMENT.md`; journal the files touched.
