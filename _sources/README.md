# Provenance snapshots

Read-only copies of the upstream documents these docs were written from. They exist so a later
writer can `diff` a snapshot against current upstream and see exactly what changed since a page was
last verified. **Never edit these, and never link readers here** — link the upstream repository
instead.

| Snapshot | Upstream | Ref | Captured |
|---|---|---|---|
| `policies-*.md`, `policies-seedset.yaml` | mlcommons/endpoints_policies | `v1.0_rules_dev@a7ec3cc` | 2026-09-19 |
| `submission-cli-*.md` | mlcommons/endpoints-submission-cli | `main@f48ca84` | 2026-09-13 |
| `endpoints-*.md` | mlcommons/endpoints | `main@47cc5c8` | 2026-09-13 |
| `mlperf-viz-README.md` | mlcommons/mlperf-viz (`cli/README.md`) | `cli-v1.1.0@654dca5` | 2026-10-02 |

The dates differ because only the policies repo moved at the 2026-09-19 resync; the CLI and
reference-client snapshots are still current at their original capture.

Not snapshotted, but mined: `endpoints/AGENTS.md`, `endpoints/src/inference_endpoint/config/`
(schema, templates, rulesets), `endpoints-submission-cli/src/submission_checker/data/seed_sets.yaml`,
and the two public pages listed in [prompt.md](../../prompt.md) §0.

Also inspected at the 2026-09-19 resync, but deliberately **not** snapshotted because it is not
merged: `mlcommons/endpoints@doc/alicheng-steady-state-design` (commit `3a51022`), which holds
`scripts/steady_state_diagnostics.{md,py}` and `docs/steady-state-detection.md`. The rules' §4.4
links a pinned commit on that branch. See **B9** in `docs/help/open-questions.md`.
