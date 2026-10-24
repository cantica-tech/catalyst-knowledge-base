# Migration 0.107.0 (module 0.5.1): `Local only`, not `None`

> Applies when a deployment of this module is synchronized to module `0.5.1`. Never re-run once applied.

## What changed

`KB`'s `Hosted embedding up to` offers `Local only` where it offered `None` (nothing goes to a hosted embedding
service). catalyst reads `none` as an empty field, so a knowledge base set to `None` failed `catalyst check` with
`required field … is missing or empty` (`BUG-000001-I3JORbi9`).

## Steps

1. **Refresh the module** (`catalyst module upgrade knowledge-base`); `catalyst migrate --apply --template KB` adds
   the generated template with the new value as the type's next version (optional, INV-23).
2. **Each knowledge base whose `Hosted embedding up to` is `None`**: set it to `Local only`, journaled
   (`--action sync`, intent naming this migration). It meant the same.
3. **Version.** Record module `0.5.1` in `DEPLOYMENT.md`.
