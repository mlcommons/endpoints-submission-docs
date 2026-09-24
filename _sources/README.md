# Provenance snapshots

Read-only copies of the upstream documents these docs were written from. They exist so a later
writer can `diff` a snapshot against current upstream and see exactly what changed since a page was
last verified. **Never edit these, and never link readers here** — link the upstream repository
instead.

| Snapshot | Upstream | Ref | Captured |
|---|---|---|---|
| `policies-*.md`, `policies-seedset.yaml` | mlcommons/endpoints_policies | `v1.0_rules_dev@6b0b1ef` | 2026-09-24 |
| `submission-cli-*.md` | mlcommons/endpoints-submission-cli | `main@f25f71e` (tag `v1.0.1.0`) | 2026-09-24 |
| `endpoints-*.md` | mlcommons/endpoints | `main@47cc5c8` | 2026-09-13 |

The 2026-09-24 resync moved the policies snapshots (Offline point, power normalization, approved
drafter lists, publication modes, audit process) and the CLI snapshots (checker `v1.0.1.0`, which
implements those rules). `policies-MLPerf_Endpoints_Audit_Guidelines.md` is new at this resync.
The two reference-client files are unchanged at `main@e71b928`, so their snapshot is still current.

Not snapshotted, but mined: `endpoints/AGENTS.md`, `endpoints/src/inference_endpoint/config/`
(schema, templates, rulesets), `endpoints-submission-cli/src/submission_checker/data/seed_sets.yaml`,
and the two public pages listed in [prompt.md](../../prompt.md) §0.

Also inspected at the 2026-09-19 resync, but deliberately **not** snapshotted because it is not
merged: `mlcommons/endpoints@doc/alicheng-steady-state-design` (commit `3a51022`), which holds
`scripts/steady_state_diagnostics.{md,py}` and `docs/steady-state-detection.md`. The rules' §4.4
links a pinned commit on that branch. See **B9** in `docs/help/open-questions.md`.

Mined at the 2026-09-24 resync, not snapshotted: `mlcommons/endpoints@main` (`e71b928`) now carries
`scripts/steady_state_diagnostics.{md,py}` and `docs/steady-state-detection.md` (merged in #447) as
an ad-hoc tool, with no run-time integration in `src/`. From the checker source at `f25f71e`:
`models/file/system_power.py`, `models/file/steady_state.py`, `models/aggregate/context.py`,
`submissions/builder.py`, and `data/{seed_sets,approved_drafters}.yaml`.
