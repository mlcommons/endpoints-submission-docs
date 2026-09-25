# Compliance checks

Every automated check, cross-walked from **checker rule ID** to the **rules clause** it enforces and
the **failure action** the rules assign. Use it to work out what a failed check actually means.

Run locally with `endpoints-submission-cli check-submission` (documented upstream as
[`submission-checker`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/README.md#submission-checker),
see **B16**); run server-side during **Week 0**. This page describes checker **`v1.0.1.0`**. Older
versions lack the Offline, power, accuracy-coverage, steady-state and drafter rules.

!!! danger "Week 0 failures reject the submission"
    A submission that fails any automated check by the end of Week 0 is rejected ([§6.1 of the
    Submission Rules][srules-6.1]). You resubmit as a **new** submission — there is no in-place
    patching, and you lose your cohort slot.

## How to read the severity column

| Severity | Meaning |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } **Reject** | The rules assign *Reject submission* to this check |
| :material-alert:{ style="color:#ef6c00" } **Reject points** | Non-conforming points are rejected |
| :material-flag:{ style="color:#f9a825" } **Flag** | Flagged for reviewer attention — does not block automatically, but is objection material |
| :material-information:{ style="color:#1565c0" } **Warn** | Checker-level warning; becomes an error under `--strict` |

!!! warning "Local severity can be stricter than §9.1"
    The icons show the failure action in the rules. The checker sets its own severity, and most
    **Flag** rows are errors locally, which stops `submissions create`:
    `system-description-consistency`, `model-name-consistency`, `tps-utilization`,
    `concurrency-in-range`, `streaming-config`, `min-query-count`, `warmup-present`,
    `config-consistency-dataset` and every `metric-consistency-*` rule. Of the **Warn** rows,
    `steady-state-valid`, `steady-state-consistency`, `seed-config-legacy` (a seed other than 42)
    and `region-basis` (no parsable `point.yaml`) can also fail as errors.

## Structure

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `path-exists` | §1 | Submission root directory exists | :material-alert-octagon:{ style="color:#c62828" } |
| `required-dir` | §1 | `results/` and `docs/` present | :material-alert-octagon:{ style="color:#c62828" } |
| `src-dir` | §2.2.1 | `src/` present with at least one implementation directory | :material-alert-octagon:{ style="color:#c62828" } |
| `src-readme` | §2.2.1 | Each `src/<implementation>/` has a `README.md` | :material-alert-octagon:{ style="color:#c62828" } |
| `system-results-dir` | §1 | At least one `results/<system>/` exists | :material-alert-octagon:{ style="color:#c62828" } |
| `benchmark-model-dir` | §1 | At least one model directory per system | :material-alert-octagon:{ style="color:#c62828" } |
| `point-dirs` | §1 | At least one `r<N>/` point directory per model | :material-alert-octagon:{ style="color:#c62828" } |
| `measurement-points-present` | §1 | Every `r<N>/` carries a `point.yaml` | :material-alert-octagon:{ style="color:#c62828" } |
| `result-summary-present` | §1 | `result_summary.json` exists for each point | :material-alert-octagon:{ style="color:#c62828" } |
| `shared-path-resolution` | §9.1 | `shared_src` / `shared_docs` resolve under the submission root | :material-alert-octagon:{ style="color:#c62828" } |
| `point-dirname-concurrency` | §1 | `r<N>/` name matches the declared concurrency | :material-information:{ style="color:#1565c0" } |

## System description

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `system-description-present` | §8.2 | Every point has a `system_desc.json` | :material-alert-octagon:{ style="color:#c62828" } |
| `system-description-valid` | §8.2 | Parses against the `SystemDescription` schema | :material-alert-octagon:{ style="color:#c62828" } |
| `system-description-consistency` | §9.1 | Every point of a curve describes the same system | :material-flag:{ style="color:#f9a825" } |
| `model-name-valid` | §3.2 | `model_name` is one of the round's supported models | :material-alert-octagon:{ style="color:#c62828" } |
| `model-name-consistency` | §8.2 | Matches the results directory name | :material-flag:{ style="color:#f9a825" } |
| `max-concurrency-declared` | §9.1 | Reports `max_supported_concurrency`. A missing value fails `system-description-valid`, and a value ≤ 32 fails `region-computation` | :material-alert-octagon:{ style="color:#c62828" } |
| `tps-utilization` | §8.2 | Equals `system_tps / max(system_tps)` over the point's own curve | :material-flag:{ style="color:#f9a825" } |
| `power-descriptor` | §4.5.2, §9.1 | `results/<system>/system_power.json` exists and a total power can be derived from it | :material-alert-octagon:{ style="color:#c62828" } |
| `power-estimated` | §4.5.2 | Names component groups left for MLCommons to fill in — the result will carry "MLC Estimated Power" | :material-information:{ style="color:#1565c0" } |

## Regions and curve structure

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `region-basis` | §5.4 | Reports the derived `C_min` and how many points it came from | :material-information:{ style="color:#1565c0" } |
| `region-computation` | §5.5 | `(C_max, C_min)` is a valid input to the reference algorithm | :material-alert-octagon:{ style="color:#c62828" } |
| `concurrency-in-range` | §9.1 | Each concurrency falls in a valid region, margin included. A dedicated Offline run is exempt | :material-flag:{ style="color:#f9a825" } |
| `region-declared` | §8.3 | Declared `region` is one of the permitted values | :material-alert:{ style="color:#ef6c00" } |
| `region-placement` | §8.3 | Declared region matches the computed one | :material-information:{ style="color:#1565c0" } |
| `offline-declared` | §8.3 | `offline` is `dedicated`, `elected` or `none` | :material-alert:{ style="color:#ef6c00" } |
| `offline-point-present` | §5.7, §9.1 | Exactly one Offline point, and `elected` only on the `C_max` point. **None present only warns** — see below | :material-alert-octagon:{ style="color:#c62828" } |
| `offline-ordering` | §5.7.2, §9.1 | Dedicated Offline run: `system_tps` ≥ 0.98 × the `C_max` point's, and concurrency ≥ `C_max` | :material-flag:{ style="color:#f9a825" } |
| `ultra-low-concurrency-coverage` | §5.4, §9.1 | At least one point at concurrency ≤ 32 | :material-alert-octagon:{ style="color:#c62828" } |
| `low-concurrency-coverage` | §9.1 | At least one point in Low Concurrency | :material-alert-octagon:{ style="color:#c62828" } |
| `med-concurrency-coverage` | §9.1 | At least one point in Medium Concurrency | :material-alert-octagon:{ style="color:#c62828" } |
| `high-concurrency-coverage` | §9.1 | At least one point in High Concurrency | :material-alert-octagon:{ style="color:#c62828" } |
| `point-count` | §5.3, §9.1 | At least 8 points when one declares `offline: dedicated`, otherwise at least 7 | :material-alert-octagon:{ style="color:#c62828" } |
| `point-cap` | §5.6, §9.1 | Does not exceed 32 points, Offline included | :material-alert:{ style="color:#ef6c00" } |

!!! warning "The 10% margin does not satisfy High Concurrency"
    A point in `C_max + 1 … ceil(1.10 × C_max)` is in its own region. It passes
    `concurrency-in-range` but does **not** count toward `high-concurrency-coverage`. The rules say
    a dedicated Offline run counts toward no region ([§5.7.2][rules-5.7.2]), but the checker's
    coverage checks don't exclude it: a dedicated run at exactly `C_max` is counted as High
    Concurrency.

!!! danger "A missing Offline point is only a warning locally"
    The checker can't tell whether a benchmark is agentic, because nothing it reads says so. So when
    **no** point declares `offline`, `offline-point-present` warns instead of failing, and
    `point-count` applies the 7-point minimum. For a non-agentic benchmark the rules reject that
    submission. Check this yourself.

!!! note "`offline-ordering` needs a point at exactly `C_max`"
    The throughput half of the check looks up the point whose concurrency equals your declared
    `C_max`. If there isn't one, it skips the comparison without reporting anything.

## Measurement points

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `point-config-valid` | §8.3 | `point.yaml` parses against the `PointConfig` schema | :material-alert-octagon:{ style="color:#c62828" } |
| `point-disclosure-complete` | §8.3 | Every required disclosure field is present | :material-alert-octagon:{ style="color:#c62828" } |
| `load-pattern` | §6.1 | `load_pattern` is `concurrency` with a positive level. A point declaring `offline: dedicated` is exempt, and no Offline pattern name is tested | :material-alert:{ style="color:#ef6c00" } |
| `streaming-config` | §6.5, §9.1 | `stream_all_chunks` is `True` | :material-flag:{ style="color:#f9a825" } |
| `point-duration` | §6.2 | The steady-state window's issue-time span meets the region's minimum | :material-flag:{ style="color:#f9a825" } |
| `steady-state-valid` | §4.4, §8.3 | `status`, `verdict` and gating `state` values are from the permitted vocabulary | :material-information:{ style="color:#1565c0" } |
| `steady-state-consistency` | §4.4 | The declared `status` agrees with the window it describes | :material-information:{ style="color:#1565c0" } |
| `steady-state-basis` | §4.4 | Reports whether steady-state or `total` supplies the official result; warns on fallback and drift | :material-information:{ style="color:#1565c0" } |
| `min-query-count` | §6.4 | `n_samples_completed` meets the dataset minimum | :material-flag:{ style="color:#f9a825" } |
| `warmup-present` | §6.3.3 | Warmup declaration present | :material-flag:{ style="color:#f9a825" } |
| `warmup-logs-retained` | §6.3.2 | Log retention declared | :material-information:{ style="color:#1565c0" } |
| `warmup-salt` | §6.3.3 | Warns when the warmup salt is enabled | :material-information:{ style="color:#1565c0" } |
| `config-consistency-dataset` | §9.1 | All points use the same dataset | :material-flag:{ style="color:#f9a825" } |

## Seed binding

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `seed-set-consistency` | §9.1 | Every point records the same seed set | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-set-membership` | §9.1 | The bound set is one MLCommons published | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-runtime-match` | §2.1.1 | The RNG seeds equal the bound set's values | :material-alert-octagon:{ style="color:#c62828" } |
| `target-cohort` | §4.6 | `target_cohort` matches `YYYY-MM-C0` / `YYYY-MM-C1` | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-set-adoption` | §4.6 | Set published for the target cohort or the three before it | :material-alert-octagon:{ style="color:#c62828" } |
| `seed-config-legacy` | §4.6 | v0.7 fallback — seeds equal 42 when no `seed_set` is declared | :material-information:{ style="color:#1565c0" } |
| `seed-set-registry` | §4.6 | Warns when the seed-set file cannot be read | :material-information:{ style="color:#1565c0" } |

!!! note "`seed-set-adoption` now runs"
    From `v1.0.1.0` the bundled file mirrors the published `seedset.yaml`, cohort key included, and
    the adoption window is derived from it. Older checkers report **SKIP** here; upgrade rather than
    override. See [step 4](../workflow/run-the-points.md#seeds-and-salting).

## Speculative decoding

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `approved-drafter` | §2.9.4, §9.1 | The declared drafter is on the benchmark's published list, by weight checksum or by target checksum plus configuration | :material-alert:{ style="color:#ef6c00" } |
| `drafter-approval-lead-time` | §2.9.4, §9.1 | It was approved at least two cohorts before `target_cohort` | :material-alert:{ style="color:#ef6c00" } |
| `drafter-list-registry` | §2.9.4 | Warns when the drafter list itself can't be read | :material-information:{ style="color:#1565c0" } |

!!! warning "The bundled list is empty"
    The reference repository lists approved heads for the agentic benchmarks only, and the checker
    hasn't picked them up: it ships an empty list and rejects any point that uses speculative
    decoding. Point `--approved-drafters FILE` or `$MLPERF_ENDPOINTS_APPROVED_DRAFTERS` at a
    published list once one exists.

!!! note "Amendments are not re-tested for adoption"
    For an amendment, every new or replacement point must match the **original** submission's bound
    seed set. The four-cohort adoption test is not reapplied using the amendment's later cohort.

## Metrics

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `result-file-valid` | §8.3 | `result_summary.json` parses against `PointSummary` | :material-alert-octagon:{ style="color:#c62828" } |
| `metric-consistency-duration` | §9.1 | `duration_ns > 0` | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-accounting` | §9.1 | `completed + failed == issued` | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-output-tokens` | §9.1 | `total_output_tokens ≥ 0` | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-system-tps` | §9.1 | Stored `system_tps` matches the derived value | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-tpot-p90` | §9.1 | Reported TPOT P90 present, finite, strictly positive | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-tps-per-user` | §9.1 | Stored `tps_per_user` matches `1000 / tpot_p90_ms` | :material-flag:{ style="color:#f9a825" } |
| `metric-consistency-tps-per-kw` | §4.5.3 | A stored `system_tps_per_kw` matches `system_tps / provisioned_power_kw` | :material-flag:{ style="color:#f9a825" } |
| `agentic-metric-consistency` | §4.1, §9.1 | `e2e_avg_interactivity` is derivable from its reported inputs | :material-flag:{ style="color:#f9a825" } |

## Accuracy

| Rule ID | Clause | Checks | Severity |
|---|---|---|---|
| `accuracy-present` | §6.6, §9.1 | At least one model carries accuracy results | :material-alert-octagon:{ style="color:#c62828" } |
| `accuracy-coverage` | §5.3, §9.1 | Accuracy results at a point in each of Ultra Low, Low, Medium and High Concurrency, and at the Offline point | :material-alert-octagon:{ style="color:#c62828" } |
| `accuracy-valid` | §6.6 | `accuracy_results.json` parses correctly | :material-alert-octagon:{ style="color:#c62828" } |
| `accuracy-sample-count` | §6.6 | Issued sample count meets the model's minimum | :material-alert-octagon:{ style="color:#c62828" } |
| `accuracy-gate` | §9.1 | Score meets the benchmark quality target | :material-alert-octagon:{ style="color:#c62828" } |

**Accuracy has no variability allowance** at any stage ([Reproducibility
Expectations][srules-reproducibility-expectations] in the Submission Rules).

!!! note "The multi-turn mean is not what the checker tests"
    `accuracy-coverage` checks which regions have results, and `accuracy-gate` checks each score.
    Rules §4.3 judges multi-turn benchmarks on the **mean** of the `N` results instead, and the
    checker doesn't compute that. For a single-turn benchmark the two agree.

## What automation does not check

Manual reviewers focus on what the checker cannot see. These are not rule IDs — they are objection
grounds. See [Why submissions get
rejected](../rules/rejection-reasons.md#rejections-that-come-from-judgement-not-checks).

## Clause-numbering note

!!! note "Rule IDs cite some clause numbers that do not exist"
    The checker's own documentation cites §14, §15 and §16 for the metrics, accuracy and consistency
    families. Those sections are not present in the current rules document — the corresponding
    content is in §6.6, §8.5 and §9.1. The clause column above maps to the rules as they actually
    are. Tracked as **B3** in [Open questions](../help/open-questions.md).

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e, `v1.0.1.0`), 2026-09-24.*
