# `point.yaml`

The per-measurement-point disclosure file. One per Pareto point, at the top level of the run folder.

!!! danger "Authored by you, copied verbatim"
    No tool generates or completes this file. The submission CLI copies it into the bundle
    **verbatim** — it does not derive it from `config.yaml` and does not fill in missing fields.
    Whatever you write is what gets submitted and what gets checked.

Authored in [step 5](../workflow/author-disclosures.md).

## Fields

The fields, and what each one means, are defined in [rules §8.3][rules-8.3]. This page doesn't
repeat that table. It lists the values the checker accepts for the fields it validates by value,
and where the tooling adds a condition the rules don't state.

| Field | What the checker accepts |
|---|---|
| `region` | `low_latency`, `low_concurrency`, `med_concurrency`, `high_concurrency` or `submitters_choice` (`region-declared`) |
| `division` | `Standardized`, `Serviced` or `RDI` |
| `dataset_type` | Any of the §8.3 values, but the bundle builder only uses it when it is exactly `Accuracy` or `Performance`. See [below](#dataset_type-does-real-work) |
| `offline` | `dedicated`, `elected` or `none`, or absent. `elected` only on the point whose concurrency equals `max_supported_concurrency` (`offline-point-present`) |
| `target_cohort` | `YYYY-MM-C0` or `YYYY-MM-C1` (`target-cohort`) |
| `shared_src`, `shared_docs` | Must resolve to directories under the submission root. See [below](#shared_src-and-shared_docs) |

!!! note "`region` on a dedicated Offline run"
    Its `concurrency` is the size of the performance dataset and it counts toward no region, but
    the rules don't say what `region` should hold. `submitters_choice` passes without a placement
    warning. Tracked as **C7** in [Open questions](../help/open-questions.md).

## The `steady_state` block

What the block records is defined in [§8.3][rules-8.3], and how the status and verdict are
decided in [§4.4][rules-4.4]. Background:
[What your numbers are measured over](metrics-and-regions.md#what-your-numbers-are-measured-over).

§8.3 describes the block in prose. The key names the checker actually reads are:

```yaml
steady_state:
  status: windowable        # windowable | insufficient_duration | insufficient_passes | partial_dataset
  verdict: STEADY STATE     # STEADY STATE | drifting_up | drifting_down | anomaly | not found
  window:
    super_pass_start: 1
    super_pass_end: 4
    super_pass_size: 1000
    n_samples: 4000
    duration_s: 1200.0      # issue-time span, checked against the minimum run duration
  state:                    # per gating metric: Plateau | Drifting Up | Drifting Down
    tpot_p50: Plateau
    tpot_p90: Plateau
  # anomaly: ...            # only when the detector confirmed a level shift
```

`verdict` is read by the checker but isn't listed in §8.3.

!!! warning "You run the detector yourself, for now"
    The detector is on `mlcommons/endpoints` `main` as an ad-hoc script,
    `scripts/steady_state_diagnostics.py`, which you point at a run directory or its
    `events.jsonl`. It isn't wired into the benchmark run, so nothing fills in this block for you.
    Its own documentation scopes it to single-turn workloads and prints *not yet supported* for
    agentic runs. A dedicated Offline run sits outside the rules' steady-state scope too.
    Tracked as **B9** in [Open questions](../help/open-questions.md).

## The `warmup` block

What it must declare is set by [§6.3.3][rules-6.3.3], and the sub-field names by [§8.3][rules-8.3]:
`duration_s`, `requests_issued`, `requests_completed`, `data_source`, `concurrency` and
`initialization_steps`. Checked by `warmup-present`.

!!! warning "Warmup documentation is objection territory"
    Warmup is at your discretion, but because warmup state materially affects the measurement, the
    procedure must be documented in enough detail for an independent team to reproduce it.
    Incomplete or ambiguous warmup documentation is explicit grounds for a **Methodology objection**.

## `dataset_type` does real work

!!! danger "The builder will not guess"
    The bundle builder must know whether a run is an **accuracy** or a **performance** run. It reads
    `datasets[].type` from `config.yaml` first, then falls back to `point.yaml`'s `dataset_type` —
    but **only** when that value is exactly `Accuracy` or `Performance`.

    A `dataset_type` of `Accuracy + Performance` describes the *dataset*, not this run. With no
    `config.yaml` and that value, the build **fails**, naming the run.

    That strictness is deliberate. Defaulting to "performance" used to be silently destructive: an
    accuracy run shipped without a `config.yaml` would be filed as its concurrency's performance
    run, collide with the real one, and drop its accuracy results from the bundle.

## `shared_src` and `shared_docs`

Each point declares which shared content it used. Both must resolve to an existing directory under
the submission root.

!!! danger "Unresolvable pointers reject the submission"
    Checker rule `shared-path-resolution`. A point whose pointers are missing or do not resolve is
    incomplete.

## Checker rules that read this file

| Rule | Checks |
|---|---|
| `measurement-points-present` | Every `r<N>/` carries a `point.yaml` |
| `point-config-valid` | It parses against the `PointConfig` schema |
| `point-disclosure-complete` | Every required disclosure field is present |
| `point-dirname-concurrency` | The `r<N>/` directory name matches the declared concurrency *(warn)* |
| `region-declared` | `region` is one of the permitted values |
| `region-placement` | The declared region matches the computed one *(warn)* |
| `offline-declared` | `offline` is `dedicated`, `elected` or `none` |
| `offline-point-present` | Exactly one point declares Offline, and `elected` sits on the `C_max` point |
| `load-pattern` | `load_pattern` is `concurrency` with a positive level. A dedicated Offline run is exempt |
| `streaming-config` | `stream_all_chunks` is `True` |
| `point-duration` | The steady-state window's issue-time span meets its region's minimum *(warn)* |
| `steady-state-valid` | `status`, `verdict` and each `state` use the permitted values |
| `steady-state-consistency` | `status` agrees with the window it describes |
| `steady-state-basis` | Reports which basis supplies the official result; flags fallbacks and drift *(warn)* |
| `approved-drafter` | The declared drafter is on the benchmark's published list |
| `drafter-approval-lead-time` | It was approved at least two cohorts before `target_cohort` |
| `min-query-count` | `n_samples_completed` meets the dataset minimum |
| `warmup-present` | The warmup declaration is present |
| `warmup-logs-retained` | Log retention is declared *(warn)* |
| `warmup-salt` | Warns when the warmup salt is enabled |
| `config-consistency-dataset` | All points use the same dataset |
| `seed-set-consistency` | Every point records the same seed set |
| `target-cohort` | `target_cohort` matches the cohort ID format |
| `shared-path-resolution` | `shared_src` / `shared_docs` resolve |

Full cross-walk: [Compliance checks](compliance-checks.md).

## See also

- [`system_desc.json`](system-desc-json.md) — the other file you author per point
- [Submission package layout](package-layout.md), where this file sits
- [Metrics and regions](metrics-and-regions.md) — how to determine the right `region` value

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef),
`mlcommons/endpoints-submission-cli@main` (f25f71e) and `mlcommons/endpoints@main` (e71b928),
2026-09-24.*
