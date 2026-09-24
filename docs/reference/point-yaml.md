# `point.yaml`

The per-measurement-point disclosure file. One per Pareto point, at the top level of the run folder.

!!! danger "Authored by you, copied verbatim"
    No tool generates or completes this file. The submission CLI copies it into the bundle
    **verbatim** — it does not derive it from `config.yaml` and does not fill in missing fields.
    Whatever you write is what gets submitted and what gets checked.

Authored in [step 5](../workflow/author-disclosures.md).

## Required fields

| Field | Description |
|---|---|
| `concurrency` | The target concurrency level for this point |
| `region` | Which region this point satisfies — `low_latency`, `low_concurrency`, `med_concurrency`, `high_concurrency`, or `submitters_choice` |
| `runtime_settings` | The settings used for the run: load pattern, `min_duration_ms`, `min_sample_count`, `stream_all_chunks` |
| `dataset` | Dataset name, and any `n_samples_from_dataset` override |
| `warmup` | The warmup declaration — see [below](#the-warmup-block) |
| `division` | `Standardized`, `Serviced` or `RDI` |
| `max_supported_concurrency` | Your declared `C_max` |
| `model_name` | Display name of the model; must be consistent across all external usages |
| `model_precision` | **Lowest** precision numerical format used for the weights. A model mixing FP16 and FP8 has `model_precision: FP8` |
| `link_to_model` | Link to the submitted model |
| `link_to_model_transformation` | Link to the calibration / quantization / transformation write-up |
| `model_notes` | Free-form supplementary notes |
| `dataset_name` | Display name of the dataset; consistent across external usages |
| `dataset_type` | `Accuracy`, `Performance`, or `Accuracy + Performance` — see the warning below |
| `dataset_link` | Link to the data used |
| `seed_set` | The seed set this submission is bound to |
| `target_cohort` | The cohort targeted, as `YYYY-MM-C0` / `YYYY-MM-C1` |
| `shared_src` | Pointer to the shared `src/` content this point used |
| `shared_docs` | Pointer to the shared `docs/` content this point used |
| `steady_state` | The reporting block described [below](#the-steady-state-block) |

## Conditional fields

| Field | When | Description |
|---|---|---|
| `offline` | Non-agentic benchmarks | `dedicated` on a dedicated Offline run; `elected` on the `C_max` point when it's elected as the Offline result; absent or `none` on every other point. Exactly one point per curve carries `dedicated` or `elected` |
| `speculative_decoding` | Points that use it | The drafter's model ID and checksum, precision, public release date, a link to its model card or technical report, and tokenizer-compatibility notes for the drafter/target pair, plus the per-point configuration |

!!! note "`region` on a dedicated Offline run"
    Its `concurrency` is the size of the performance dataset and it counts toward no region, but
    the rules don't say what `region` should hold. `submitters_choice` passes without a placement
    warning. Tracked as **C7** in [Open questions](../help/open-questions.md).

## The steady-state block

Added in 2026-09. It records how the point's official numbers were derived — see
[What your numbers are measured over](metrics-and-regions.md#what-your-numbers-are-measured-over).

| Field | Description |
|---|---|
| `status` | `windowable`, `insufficient_duration`, `insufficient_passes` or `partial_dataset` |
| `verdict` | The detector's shape verdict: `STEADY STATE`, `drifting_up`, `drifting_down`, `anomaly` or `not found`. Read by the checker; not listed in rules §8.3 |
| `window` | `super_pass_start`, `super_pass_end`, `super_pass_size`, `n_samples`, and `duration_s` — the window's issue-time span, which is what gets checked against the minimum run duration |
| `state` | Per gating metric: `Plateau`, `Drifting Up` or `Drifting Down` |
| `anomaly` | Present **only** when the detector confirmed a level shift |

`total` metrics are reported alongside as supplementary. The window sub-field names are the ones
the checker reads; §8.3 describes them only in prose.

!!! warning "You run the detector yourself, for now"
    The detector is on `mlcommons/endpoints` `main` as an ad-hoc script,
    `scripts/steady_state_diagnostics.py`, which you point at a run directory or its
    `events.jsonl`. It isn't wired into the benchmark run, so nothing fills in this block for you.
    Its own documentation scopes it to single-turn workloads and prints *not yet supported* for
    agentic runs. A dedicated Offline run sits outside the rules' steady-state scope too.
    Tracked as **B9** in [Open questions](../help/open-questions.md).

## The warmup block

Required by the run-requirements rules and checked by `warmup-present`.

| Field | Description |
|---|---|
| `duration_s` | Seconds from the first warmup request to `TEST_STARTED` |
| `requests_issued` | Total warmup requests issued |
| `requests_completed` | Total warmup requests completed |
| `data_source` | Description of the warmup data and its origin — dataset name and split, synthetic generation method and parameters, or fixed prompt text |
| `concurrency` | Concurrency level used during warmup |
| `initialization_steps` | Platform-specific initialization performed (CUDA graph capture, engine loading, JIT triggers), and confirmation it completed before `TEST_STARTED` |

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
