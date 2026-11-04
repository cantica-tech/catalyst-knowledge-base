# Migration 0.124.0 (module 0.8.0): only a curator creates a knowledge base or a build

> Applies when a deployment of this module is synchronized to module `0.8.0` (`0.7.x` → `0.8.0`). The module still
> needs kernel `0.107.0` or later; the change takes effect on kernel `0.124.0` or later (`created_by_exclusive`,
> fw-STRUCTURE-000026), and an older kernel ignores the field. Never re-run once applied.

## What changed

1. **`created_by_exclusive: true`** is declared on `KB` (created by `/kb-setup`) and `KBUILD` (created by
   `/kb-build`).
2. On kernel `0.124.0` or later, `catalyst new KB` checks the `cmd:kb-setup` grant alone and `catalyst new KBUILD`
   the `cmd:kb-build` grant alone. The general `cmd:create`, a `create-<type>` alias and `create:<TYPE>` no longer
   let a signer create them.
3. **Who may create them from now on:** a person whose role holds `cmd:kb-setup` (a knowledge base) or
   `cmd:kb-build` (a build): the `Knowledge Curator`, and an `Admin` (`cmd:*`). A role that holds `/create` only, as
   another module's roles may, can no longer create either (`rr-KB-006`).
4. `KSOURCE` is unchanged: it keeps the generic create.

## Steps

1. **Refresh the module** the ordinary sync way (`/sync-framework`).
2. **Check the provider's roles.** The `Knowledge Curator` role in catalyst-users must hold `cmd:kb-setup` and
   `cmd:kb-build` (with `cmd:kb-import` and `cmd:kb-classify`). If it does not, ask the provider's admin to add them
   (`catalyst-users roles provision`, with the user's assent); until then only an `Admin` creates a knowledge base or
   a build. Who holds them: `catalyst identity status` for yourself, the provider's admin for others.
3. **Version.** Record module `0.8.0` in `DEPLOYMENT.md`; journal the files touched.
