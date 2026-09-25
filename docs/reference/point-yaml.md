# `point.yaml`

Authored in [step 5](../workflow/author-disclosures.md).

## Fields

This page lists the values the checker accepts for the [§8.3][rules-8.3] fields it validates by
value, and where the tooling adds a condition the rules don't state.

| Field | What the checker accepts |
|---|---|
| `region` | `low_latency`, `low_concurrency`, `med_concurrency`, `high_concurrency` or `submitters_choice` (`region-declared`) |

| `dataset_type` | Any of the §8.3 values, but the bundle builder only uses it when it is exactly `Accuracy` or `Performance`. |
| `offline` | `dedicated`, `elected` or `none`, or absent. `elected` only on the point whose concurrency equals `max_supported_concurrency` (`offline-point-present`) |
| `target_cohort` | `YYYY-MM-C0` or `YYYY-MM-C1` (`target-cohort`) |
| `shared_src`, `shared_docs` | Must resolve to directories under the submission root. |

!!! note "`region` on a dedicated Offline run"
    Its `concurrency` is the size of the performance dataset and it counts toward no region, but the
    rules don't say what `region` should hold. `submitters_choice` passes without a placement
    warning. Tracked as **C7** in [Open questions](../help/open-questions.md).

## The `steady_state` block

What the block records is defined in [§8.3][rules-8.3], and how the status and verdict are decided
in [§4.4][rules-4.4]. Background: [What your numbers are measured
over](metrics-and-regions.md#what-your-numbers-are-measured-over).

Two things the checker reads that §8.3 doesn't name: the `window` sub-keys are `super_pass_start`,
`super_pass_end`, `super_pass_size`, `n_samples` and `duration_s`, and the block also carries a
`verdict`, which takes one of the §4.4 verdict values.

!!! warning "You run the detector yourself, for now"
    The detector is on `mlcommons/endpoints` `main` as an ad-hoc script,
    `scripts/steady_state_diagnostics.py`, which you point at a run directory or its `events.jsonl`.
    It isn't wired into the benchmark run, so nothing fills in this block for you. Its own
    documentation scopes it to single-turn workloads and prints *not yet supported* for agentic
    runs. A dedicated Offline run sits outside the rules' steady-state scope too. Tracked as **B9**
    in [Open questions](../help/open-questions.md).

## The `warmup` block

What it must declare is set by [§6.3.3][rules-6.3.3], and the sub-field names by [§8.3][rules-8.3].

## See also

- [`system_desc.json`](system-desc-json.md) — the other file you author per point
- [Submission package layout](package-layout.md), where this file sits
- [Metrics and regions](metrics-and-regions.md) — how to determine the right `region` value

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef),
`mlcommons/endpoints-submission-cli@main` (f25f71e) and `mlcommons/endpoints@main` (e71b928),
2026-09-24.*
