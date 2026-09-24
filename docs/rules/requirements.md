# Requirements you must meet

Every binding constraint, grouped by what it constrains, with a note on how to tell whether you
comply. Rules marked :material-alert-octagon:{ style="color:#c62828" } have a failure action of
**reject the submission**.

--8<-- "precedence-notice.md"

## Curve structure

| Requirement | How to check |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } **≥ 8 measurement points** including a dedicated Offline run; **≥ 7** if you elect `C_max` as the Offline result, or for an agentic benchmark | `submission-checker check` — rule `point-count` |
| **≤ 32 measurement points**, Offline included | rule `point-cap` |
| :material-alert-octagon:{ style="color:#c62828" } **Exactly one Offline point** for a non-agentic benchmark, **none** for an agentic one | rule `offline-point-present` — see the warning below |
| :material-alert-octagon:{ style="color:#c62828" } **≥ 1 point at concurrency 1–32** (Ultra Low) | rule `ultra-low-concurrency-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } **≥ 1 point in each of Low, Medium, High Concurrency** | rules `low-` / `med-` / `high-concurrency-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } **`C_max` declared and > 32** | rule `max-concurrency-declared` |
| Every concurrency falls in a valid region (a dedicated Offline run is exempt) | rule `concurrency-in-range` |

Regions are computed in log-2 space from your own `C_min` and `C_max`. See
[Plan your Pareto curve](../workflow/plan-your-curve.md) and
[Metrics and regions](../reference/metrics-and-regions.md).

!!! warning "The checker can't enforce the Offline requirement yet"
    It has no way to tell whether your benchmark is agentic, so a submission with **no** Offline
    point only gets a warning locally. For a non-agentic benchmark the rules reject it.

## The Offline point

Required for non-agentic benchmarks, and not allowed for agentic ones.

| Requirement | Detail |
|---|---|
| How it's satisfied | A **dedicated** run under the Offline load pattern, or **electing** your `C_max` point |
| Load pattern | The whole sample set available at once, not paced to a concurrency |
| Reported `concurrency` | The number of queries in one pass over the performance dataset |
| Throughput | Dedicated run: `system_tps` ≥ **0.98 ×** the `C_max` point's (rule `offline-ordering`, flagged) |
| Concurrency | Dedicated run: ≥ `C_max` (rule `offline-ordering`, flagged) |
| Latency | TTFT isn't required and **can't** be the basis of a compliance check or objection |
| Reordering | Allowed within one pass over the dataset, **never across passes** |
| Region coverage | A dedicated run counts toward none. An elected point keeps its own region |
| Everything else | Minimum duration, completed queries, dataset handling and accuracy apply as for any other point |
| Declaration | `offline: dedicated` or `offline: elected` in that point's `point.yaml`. `elected` only on the `C_max` point |

The TTFT exemption and the pass-boundary rule apply only to a dedicated run. An elected point is
an ordinary fixed-concurrency point and reports latency like the rest.

## Per-point run requirements

| Requirement | Value | How to check |
|---|---|---|
| Load pattern | The benchmark's **fixed-concurrency** pattern for every point except a dedicated Offline run, which uses the Offline pattern. `poisson` is invalid | rule `load-pattern` |
| Steady-state duration | **600 s** Ultra Low; **1,200 s** Low / Medium / High | rule `point-duration` |
| Completed queries | At least one pass over the dataset | rule `min-query-count` |
| Samples issued | A whole-number multiple of the dataset size | — |
| Streaming | `stream_all_chunks = true` | rule `streaming-config` |
| Sampling order | Performance: with replacement. Accuracy: without replacement | — |

!!! warning "Section 6 is pending ratification"
    The duration and query-count values are marked as example values subject to working-group
    ratification, though they are stated as locked and Task Force-approved for the v0.7 round.
    Check [Open questions](../help/open-questions.md) before planning around them.

## Warmup

Warmup is optional, capped at 24 hours per point, and entirely up to you, but it is
heavily constrained and heavily disclosed, because warmup state materially affects the measurement.

| Requirement | Detail |
|---|---|
| Excluded from metrics | Everything issued before `TEST_STARTED` |
| **No performance-dataset samples** | Direct use, subsets, truncations, or anything derived from them. If the client does use the performance dataset, **salting must be enabled** — and it is not on by default |
| Logs retained | Must be available for reviewer inspection; reviewers may cross-check against the dataset |
| Declared per point | `duration_s`, `requests_issued`, `requests_completed`, `data_source`, `concurrency`, `initialization_steps` |

Incomplete or ambiguous warmup documentation is explicit grounds for a Methodology objection.

## Accuracy

| Requirement | Detail |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } Accuracy results at every required point | The four mandatory region points plus the Offline point: **5** for non-agentic benchmarks, **4** for agentic. Rule `accuracy-coverage` |
| :material-alert-octagon:{ style="color:#c62828" } The results pass the quality target | **Single-turn:** every result passes. **Multi-turn:** the average of them passes |
| Placement (single-turn) | Same concurrency as the point, same instance, immediately after that point's performance run |
| Same configuration | Identical endpoint config, weights and software stack as the performance runs |
| Dataset | The **un-salted** accuracy dataset |
| Tolerance | **None.** Accuracy is a hard gate at compliance and throughout review |

!!! question "The targets are not published"
    Per-benchmark accuracy tolerance values are marked `[WIP]` upstream. They are not in any source
    this documentation could verify. Ask MLCommons. Tracked as **C2** in
    [Open questions](../help/open-questions.md).

## Seeds

| Requirement | Detail |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } One seed set per submission | Every point records the **same** set |
| Adoption window | The set must have been published for your `target_cohort` or one of the three preceding cohorts |
| Binding lifetime | Once bound, the set stays valid for that submission even after newer sets publish |
| Runtime match | The client's RNG seeds must equal the bound set's values |
| Salt entropy | At least 64 bits per query, generated at request-construction time, never stored in the dataset |
| Salt placement | **Between** the system prompt and the per-query user context |

For amendments, new or replacement points must match the original submission's bound set — the
four-cohort adoption test is not reapplied.

## Consistency across the curve

Same model, endpoint configuration, software stack, dataset and seed set at **every** point. Every
point's `system_desc.json` must describe the same system, and the whole curve uses one provisioned
power figure. If you use speculative decoding, it's the same drafter at every point. Freeze the
stack before the first run.

## Power normalization

Standardized results, CoP and CoN, are normalised by **provisioned power**. RDI may report it.
Serviced is deferred to a later version.

| Requirement | Detail |
|---|---|
| :material-alert-octagon:{ style="color:#c62828" } `system_power.json` for each system | At `results/<system>/system_power.json`. Rule `power-descriptor` |
| Normalized metric | `system_tps_per_kw = system_tps / provisioned_power_kw`. Rule `metric-consistency-tps-per-kw` |
| Power model | `(CPU + accelerator + scale-up switches) × (1 + overhead)`, overhead **0.30** liquid-cooled, **0.50** air-cooled |
| Component power | `count × TDP` for each group, backed by a public, verifiable source. Vendor spec sheets, conference papers, and statements at keynotes or earnings calls count. Analyst blogs, social media and press speculation don't |
| Declaring a total instead | Allowed, with the same evidence standard. Where a spec gives a range, use the upper bound |
| Partially populated systems | Published power for that configuration; or `P_rack × Y/N` for `Y` whole nodes of an `N`-node rack; or the formula with only the installed components counted. **Not** linear scaling inside a node |
| Below rated TDP | Public evidence of the reduced rating, plus evidence reproducible by an audit, such as `nvidia-smi` or `rocm-smi` output |
| Fixed per system | The same figure divides every point, however much of the system a point used. A power-capped variant is a different system |
| Values you leave out | Filled in by MLCommons from conservative estimates. The result is tagged **"MLC Estimated Power"**. Rule `power-estimated` warns |
| Scope | Remote storage racks can be left out. With data-centre-level liquid cooling you may give the power of the whole data centre, scaled to your system's size. A rack or system with its own CDU must include its cooling power |

If you disagree with an MLCommons estimate, you have to point to a better public source, or publish
the figure yourself. Non-public information is at MLCommons's discretion.

Field names and an example: [`system_power.json`](../reference/system-power-json.md).

!!! warning "Section 4.5 is pending ratification"
    The tiers, overhead fractions, reference components and even the metric's name and units are
    marked as subject to change. v1.0 uses Tier 3, the component sum. Nameplate power (Tier 2) and
    measured power are later.

## Disclosure by division

| | Standardized | Serviced | RDI |
|---|---|---|---|
| Source code to reproduce | Required | N/A (remote optional) | Optional |
| Rack / node hardware | Required | Optional | Required |
| Primary accelerator | Required | Required | Required |
| Parallelism mapping (TP/EP/PP/DP) | Required | Optional | Optional |
| Software stack versions | Required | From public docs and API metadata | Required |
| Model name and version | Required | Required, as advertised | Required |
| Pricing and rate limits | — | **Required** | — |

Software disclosure must name, at minimum: the serving framework with version and commit or release
tag, the accelerator compute library and build, the driver version, and the operating system.

## Reproducibility

| Division | By a third party | On a third-party system | On a public endpoint |
|---|---|---|---|
| Standardized — CoP | Required | Required | N/A |
| Standardized — CoN | Required | Required | N/A |
| Serviced — CoN | Required | Optional | **Required** |
| RDI | Optional | Optional | Optional |

Variability margins during review: **10%** when an independent party re-runs, **5%** on the exact
same system. Both apply to `system_tps` and `tps_per_user` only, **not** to any latency percentile.

## Tokenization

Official output token counts are produced by the **client-side reference tokenizer**, applied once
to the reconstructed assistant message through the model's official reference chat template. Not by
your serving stack, and not as a sum of per-chunk counts.

| Content | Counted? |
|---|---|
| Visible output | Yes |
| Tool-call content | Yes — reassembled by ascending tool-call index |
| Reasoning / thinking | Yes |
| Chat-template framing | Only payload-specific framing; empty-assistant framing is subtracted out |

!!! danger "Padding and duplication invalidate the point"
    Each received fragment must be assigned to exactly one response field and must not contribute
    twice. Deliberately duplicating substantially identical content across fields, or padding any
    field to inflate reported token counts, is prohibited and **invalidates the affected
    measurement point**.

    Emitting whitespace, control characters or punctuation just to stop the TTFT clock, rather than
    as a genuine part of the response, is also not allowed.

## See also

- [Model equivalence](model-equivalence.md) — if you are submitting Standardized
- [Publication status](publication-status.md) — the Available / Preview / RDI tests
- [Compliance checks](../reference/compliance-checks.md) — rule ID to clause cross-walk

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
