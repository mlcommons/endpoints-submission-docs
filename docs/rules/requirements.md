# Requirements you must meet

Every binding constraint, grouped by what it constrains, with a note on how to tell whether you
comply. Rules marked :material-alert-octagon:{ style="color:#c62828" } have a failure action of
**reject the submission**.

--8<-- "precedence-notice.md"

## Curve structure

From [§5.3][rules-5.3], [§5.4][rules-5.4] and [§5.6][rules-5.6], with failure actions from
[§9.1][rules-9.1].

| Requirement | How to check |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } **≥ 8 measurement points** including a dedicated Offline run; **≥ 7** if you elect `C_max` as the Offline result, or for an agentic benchmark | `endpoints-submission-cli check-submission` — rule `point-count` |
| **≤ 32 measurement points**, Offline included | rule `point-cap` |
| :material-alert-octagon:{ style="color:#c62828" } **Exactly one Offline point** for a non-agentic benchmark, **none** for an agentic one | rule `offline-point-present` — see the warning below |
| :material-alert-octagon:{ style="color:#c62828" } **≥ 1 point at concurrency 1–32** (Ultra Low) | rule `ultra-low-concurrency-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } **≥ 1 point in each of Low, Medium, High Concurrency** | rules `low-` / `med-` / `high-concurrency-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } **`C_max` declared and > 32** | rules `system-description-valid` (missing) and `region-computation` (≤ 32) |
| Every concurrency falls in a valid region (a dedicated Offline run is exempt) | rule `concurrency-in-range` |

Regions are computed in log-2 space from your own `C_min` and `C_max`. See [Plan your Pareto
curve](../workflow/plan-your-curve.md) and [Metrics and
regions](../reference/metrics-and-regions.md).

!!! warning "The checker can't enforce the Offline requirement yet"
    It has no way to tell whether your benchmark is agentic, so a submission with **no** Offline
    point only gets a warning locally. For a non-agentic benchmark the rules reject it.

## The Offline point

Required for non-agentic benchmarks, and not allowed for agentic ones ([§5.7][rules-5.7]). You
either make a **dedicated** Offline run or **elect** your `C_max` point; both are defined in
[§5.7.2][rules-5.7.2], and what an Offline run is (dataset-sized concurrency, no TTFT requirement,
reordering only within a pass) is in [§5.7.1][rules-5.7.1]. What the checker does with it:

| Requirement | How to check |
|---|---|
| Declaration: `offline: dedicated` or `offline: elected` in that point's `point.yaml`; `elected` only on the `C_max` point | rule `offline-point-present` |
| Dedicated run: `system_tps` ≥ **0.98 ×** the `C_max` point's, and concurrency ≥ `C_max` | rule `offline-ordering` (flagged). The throughput half is skipped silently if no point sits at exactly `C_max` |
| Dedicated run uses the Offline load pattern | Not checked: `load-pattern` exempts a dedicated Offline point |

A dedicated run counts toward no region. An elected point keeps its own region and reports latency
like any other point.

## Per-point run requirements

Defined in [§6 of the rules][rules-6]. The values at a glance:

| Requirement | Value | How to check |
|---|---|---|
| Load pattern ([§6.1][rules-6.1]) | Fixed-concurrency for every point except a dedicated Offline run. `poisson` is invalid | rule `load-pattern` |
| Steady-state duration ([§6.2][rules-6.2]) | **600 s** Ultra Low; **1,200 s** Low / Medium / High | rule `point-duration` |
| Completed queries ([§6.4][rules-6.4]) | At least one pass over the dataset | rule `min-query-count` |
| Samples issued ([§6.4][rules-6.4]) | A whole-number multiple of the dataset size | — |
| Streaming ([§6.5][rules-6.5]) | `stream_all_chunks = true` | rule `streaming-config` |
| Sampling order ([§6.5][rules-6.5]) | Performance: with replacement. Accuracy: without replacement | — |

!!! warning "Section 6 is pending ratification"
    The duration and query-count values are marked as example values subject to working-group
    ratification, though they are stated as locked and Task Force-approved for the v0.7 round. Check
    [Open questions](../help/open-questions.md) before planning around them.

## Warmup

Optional, capped at 24 hours per point, and it must not use performance-dataset samples; if the
client does use the performance dataset, salting must be enabled ([§6.3.1][rules-6.3.1]). Every
point declares six warmup fields ([§6.3.3][rules-6.3.3]), checked by rule `warmup-present`. Salting
is **not** on by default in the client, so if your warmup uses the performance dataset you have to
turn it on yourself; rule `warmup-salt` then warns, and reviewers may ask about it. Incomplete
warmup documentation is grounds for a Methodology objection.

## Accuracy

Accuracy results are needed at the four mandatory region points plus the Offline point: **5** for
non-agentic benchmarks, **4** for agentic ([§5.3][rules-5.3]). How they're judged, single-turn per
point or multi-turn on the average, is in [§4.3][rules-4.3]. Accuracy runs use the same
configuration and stack as the performance runs, on the un-salted accuracy dataset, and there is no
tolerance ([Reproducibility Expectations][srules-reproducibility-expectations]).

| Requirement | How to check |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } Accuracy results at every required point | rule `accuracy-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } The results pass the quality target | rule `accuracy-gate` |

!!! question "The targets are not published"
    The per-benchmark targets are marked `[WIP]` in [§2.9.8][rules-2.9.8]. For the legacy benchmarks
    the checker gates against the MLPerf Inference targets (see [Benchmarks and
    models](../reference/benchmarks.md#accuracy-targets)); agentic targets aren't in the checker
    yet. Tracked as **C2** in [Open questions](../help/open-questions.md).

## Seeds

:material-alert-octagon:{ style="color:#c62828" } Every point records the **same** published seed
set, adopted within its window and bound for the life of the submission ([Submission Rules
§4.6][srules-4.6]). The checker tests this as `seed-set-consistency` and `seed-set-membership`; the
exact test, including the rule for amendments, is the *Seed-set validity* row of [§9.1][rules-9.1].
The per-query salt is seeded from the same set and has its own requirements in
[§2.9.5.1][rules-2.9.5.1].

## Consistency across the curve

Same model, endpoint configuration, software stack and seed set at **every** point (*Configuration
consistency* in [§9.1][rules-9.1]). The checker also requires the same dataset
(`config-consistency-dataset`). Every point's `system_desc.json` must describe the same system, and
the whole curve uses one provisioned power figure ([§4.5.3][rules-4.5.3]). If you use speculative
decoding, it's the same drafter at every point. Freeze the stack before the first run.

## Power normalization

Standardized results, CoP and CoN, are normalised by **provisioned power**, estimated from the rated
power of CPUs, accelerators and scale-up switches plus a fixed overhead. The model is in [Power
Model][rules-4.5.2-power-model], the evidence standard and what MLCommons does with gaps in
[Methodology][rules-4.5.2-methodology], partial racks and nodes in [§4.5.2.1][rules-4.5.2.1], and
the metric and why the figure is fixed per system in [§4.5.3][rules-4.5.3]. RDI may report it;
Serviced is deferred.

| Requirement | How to check |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } `system_power.json` for each system, at `results/<system>/system_power.json` | rule `power-descriptor` |
| `system_tps_per_kw = system_tps / provisioned_power_kw` | rule `metric-consistency-tps-per-kw` |
| Values you leave out are estimated by MLCommons and the result tagged **"MLC Estimated Power"** | rule `power-estimated` warns |

Field names and an example: [`system_power.json`](../reference/system-power-json.md).

!!! warning "Section 4.5 is pending ratification"
    The tiers, overhead fractions, reference components and even the metric's name and units are
    marked as subject to change ([§4.5][rules-4.5]).

## Disclosure by division

What each division has to disclose (source code, hardware, parallelism mapping, software stack) is
tabulated in [§2.7 Transparency Requirements][rules-2.7]. The minimum software list for Standardized
and RDI is in [§8.4][rules-8.4]. Serviced additionally discloses pricing and rate limits
([§2.3.1][rules-2.3.1]).

## Reproducibility

Standardized must be reproducible by a third party on a third-party system; Serviced on the public
endpoint; RDI has no requirement ([§2.6][rules-2.6]). The variability margins reviewers apply (10%
independent re-run, 5% same system, throughput only) are in [Reproducibility
Expectations][srules-reproducibility-expectations].

## Tokenization

Official output token counts come from the **client-side reference tokenizer**, applied once to the
reconstructed response through the model's reference chat template, not from your serving stack
([§2.8][rules-2.8]). Visible output, tool calls and reasoning all count.

!!! danger "Padding and duplication invalidate the point"
    Duplicating content across response fields, or padding any field, to inflate token counts
    invalidates the affected measurement point ([§2.8][rules-2.8]). Reviewers also look for leading
    content emitted only to stop the TTFT clock ([§9.2][rules-9.2]).

## See also

- [Model equivalence](model-equivalence.md) — if you are submitting Standardized
- [Publication status](publication-status.md) — the Available / Preview / RDI tests
- [Compliance checks](../reference/compliance-checks.md) — rule ID to clause cross-walk

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
