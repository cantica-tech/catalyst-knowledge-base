# Module migrations index

One-time migrations for this module's entity shapes. `From`/`Target` name
catalyst kernel versions; `/sync-framework` applies these together with the
kernel's own migrations, in version order.

| Migration | From | Target | What |
|---|---|---|---|
| [`0.107.0/onboarding-and-readable-builds.md`](0.107.0/onboarding-and-readable-builds.md) | `0.106.1` | `0.107.0` | Module `0.5.0`. `KB-` gains `Basis`, `Document base`, `Target` (optional, set by onboarding); `KBUILD-` gains `Target`, `Embedding model` required only of vectors; `/kb-setup` onboards, its curator role from catalyst-users; `rr-KB-009`/INV-KB-9: a build is readable by any model without catalyst. Requires kernel `0.107.0`. |
| [`0.107.0/local-only-embedding.md`](0.107.0/local-only-embedding.md) | `0.107.0` | `0.107.0` | Module `0.5.1`. `Hosted embedding up to`'s value `None` becomes `Local only`: catalyst reads `none` as an empty field, so a knowledge base embedded only locally failed `catalyst check` (BUG-000001-I3JORbi9). |
| [`0.107.0/source-kind-and-visibility.md`](0.107.0/source-kind-and-visibility.md) | `0.107.0` | `0.107.0` | Module `0.6.0`. `KSOURCE-` gains `Kind` (`Repository` / `Folder` / `Site` / `Criterion`: a catalyst criterion read through catalyst's CLI) and `Visibility` (`Public` / `Private`); a private source's documents are at least `Internal`, and a build refuses when the record and the host disagree (`REQ-000002-I3JORbi9`). |
