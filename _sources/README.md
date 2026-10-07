# Provenance snapshots

Read-only copies of the upstream documents these docs were written from. They exist so a later
writer can `diff` a snapshot against current upstream and see exactly what changed since a page was
last verified. **Never edit these, and never link readers here** — link the upstream repository
instead.

| Snapshot | Upstream | Ref | Captured |
|---|---|---|---|
| `policies-*.md`, `policies-seedset.yaml` | mlcommons/endpoints_policies | `v1.0_rules_dev@d2d9da6` | 2026-10-02 |
| `submission-cli-*.md` | mlcommons/endpoints-submission-cli | `main@a42a056` (after tag `v1.1.0.0`) | 2026-10-02 |
| `endpoints-*.md` | mlcommons/endpoints | `main@47cc5c8` | 2026-09-13 |
| `mlperf-viz-README.md` | mlcommons/mlperf-viz (`cli/README.md`) | `cli-v1.1.0@654dca5` | 2026-10-02 |

The 2026-10-02 resync moved the policies snapshots (streaming and the multi-token stream interval,
`model_name` and `link_config` out of `system_desc.json`, file-name fixes) and the CLI snapshots
(checker `v1.1.0.0`: agentic models and accuracy gates, the §4.5.2 power model, no
`--publication-cycle`, submissions left in `COMPLIANCE_CHECKING`, the new API default, and
`create-local` deprecated on `main`). The two reference-client files are unchanged at
`main@f1100cf`. Mined, not snapshotted: endpoints#514 (steady state computed during the run,
detector moved to `src/inference_endpoint/metrics/`), endpoints#519 (DeepSeek-V4.1-Flash), and
from the checker at `a42a056`: `checker.py`, `models/aggregate/context.py`,
`models/file/{point_config,system_power}.py` and `data/approved_drafters.yaml`.

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
