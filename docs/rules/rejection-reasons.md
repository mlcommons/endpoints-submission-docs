# Why submissions get rejected

Failure modes as **symptom → cause → fix**, each linked to the step where it is avoided.

!!! note "Derived from the rules, not from observed submissions"
    MLPerf Endpoints has not yet completed a submission round under these rules. Every entry below
    is derived from a stated rejection action in the compliance rules, a checker rule, or a
    documented tool failure mode — **not** from failures seen in the wild. Once the first round
    supplies real data, this page should be rewritten from support threads and reviewer findings,
    and speculative entries deleted.

## The hard stops

These have a failure action of **reject** in [§9.1][rules-9.1]. There is no patching in place: you
correct and resubmit as a new submission ([Submission Rules §6.1][srules-6.1]).

| Symptom | Cause | Fix |
|---|---|---|
| `point-count` fails | Fewer than 8 points with a dedicated Offline run, or fewer than 7 otherwise | Run more points. Withdrawn points don't count, and there's no way to add a point to a submission — [plan a spare](../workflow/plan-your-curve.md) |
| `offline-point-present` fails | Nothing declares `offline` on a single-turn frontier, two points do, `elected` sits on a point that isn't at `C_max`, or an agentic frontier declares one | Keep exactly one declaration, and elect only the point at your declared `C_max`. An agentic frontier has none — [step 3](../workflow/plan-your-curve.md#6-decide-how-to-meet-the-offline-requirement) |
| `power-descriptor` fails | No `system_power.json` for a system, one with nothing a total can be derived from, or (per §9.1) one that doesn't conform to [Appendix E][rules-appendix-e] and its [validation rules][rules-e7] | Author it — [step 5](../workflow/author-disclosures.md#3-write-system_powerjson-for-each-system) |
| Per-point power checks (§9.1: *Per-point node declaration*, *Per-point denominator*, *Maximal engagement*) | A declared `nodes_used` names an unknown node set, exceeds what is provisioned, or disagrees with the parallelism in `system_desc.json`; a stored `system_tps_per_kw` differs from the recomputed value; or a point engaged fewer replicas than fit without a `dp_shortfall` | Correct `nodes_used`, or declare `dp_shortfall` — [§4.5.3][rules-4.5.3] |
| *Accuracy run count* (§9.1) | More or fewer accuracy validation results than required: 5 non-agentic, 4 agentic, one per mandatory point and none at a submitter's-choice point. An elected `C_max` point's run also covers Offline. Agentic inline scores don't count | Keep exactly that set — [§4.3][rules-4.3] |
| *C_min override* (§9.1) | `c_min_override` isn't an integer in 1–32, matches no submitted point, or isn't the same on every point | Fix or drop it on every point — [§5.4][rules-5.4-cmax] |
| `*-concurrency-coverage` fails | No point in Low, Medium or High Concurrency | Recompute boundaries with the checker's `regions` command ([step 3](../workflow/plan-your-curve.md#3-compute-the-boundaries)) and run the missing region. Remember the 10% margin is **not** High Concurrency |
| `ultra-low-concurrency-coverage` fails | No point in Ultra Low Concurrency (1–`C_min`) | Run one there — [step 3](../workflow/plan-your-curve.md#2-understand-what-sets-c_min) |
| `system-description-valid` or `region-computation` fails | `max_supported_concurrency` missing (the first), or ≤ 32 (the second). §9.1 calls this *Max concurrency declared* | Declare a `C_max` > 32 in `system_desc.json` |
| `accuracy-gate` fails | Accuracy run missed the benchmark quality target | No tolerance exists. Fix the configuration and re-run — and check whether an approximation under model equivalence pushed you under |
| `agentic-accuracy-inline` / `agentic-accuracy-swebench` / `agentic-osl-range` fails | An agentic point has a missing or failing inline score or output length, or the mean SWE-bench score across the four regions is too low | No tolerance. Fix the configuration and re-run — [targets](../reference/benchmarks.md#agentic-benchmarks) |
| `accuracy-present` / `accuracy-coverage` fails | No accuracy results in one of the four mandatory regions, or none at the Offline point | Run the missing validation — one at each mandatory region point and one at Offline, same stack as performance |
| `shared-path-resolution` fails | `shared_src` / `shared_docs` do not resolve under the submission root | Fix the pointers in every `point.yaml` — [step 5](../workflow/author-disclosures.md) |
| `seed-set-consistency` / `seed-set-membership` fails | Points record different seed sets, or a set MLCommons never published | Bind **one** published set and record it at every point |
| Required files missing | A point lacks `point.yaml`, `system_desc.json` or `result_summary.json` | [Step 5](../workflow/author-disclosures.md). The reference client doesn't write them; [`mlperf-sysinfo`](https://docs.mlcommons.org/mlperf-sysinfo/) can capture `system_desc.json` |
| `load-pattern` fails | A point other than a dedicated Offline run used `max_throughput` or `poisson`. This rejects the **point** | Only the fixed-concurrency pattern is valid there. Re-run, or declare `offline: dedicated` if it really is your Offline run |
| `approved-drafter` / `drafter-approval-lead-time` fails | A point used a drafter that isn't on the [approved list](https://github.com/mlcommons/endpoints-submission-cli/blob/main/src/submission_checker/data/approved_drafters.yaml) or a native head of an approved checkpoint, or was approved too recently | These reject the **points**. Checker `v1.1.0.0` ships an empty list; use a later release, or re-run without speculation |

## Flagged by the rules, but they fail the local check

[§9.1][rules-9.1] only flags these, but the checker reports them as errors, and `submissions create`
stops on any error. Fix them before you submit.

| Symptom | Cause | Fix |
|---|---|---|
| `min-query-count` fails | Fewer completed queries than the minimum | Re-run with a longer issue window |
| `streaming-config` fails | `point.yaml` records `stream_all_chunks: false` | Re-run with it `true`. The rules now allow either value, but the checker still requires `true` |
| `benchmark-type-consistency` fails | Some points use `agentic_inference` and others don't | Use one load pattern for the whole frontier. Until you do, it's checked as single-turn |
| `config-consistency-model` fails | Points declare different `model_name` values in `point.yaml` | One frontier, one model. Fix the name, spelled as in [Benchmarks and models](../reference/benchmarks.md) |
| `concurrency-in-range` fails | A point sits outside every valid region | Recompute boundaries — `C_min` is your lowest point unless you declare `c_min_override`, which checkers before `v1.5.1.0` ignore |
| `warmup-present` fails | Warmup declaration missing | Declare the warmup block in `point.yaml` |
| `tps-utilization` fails | Value is not `system_tps / max(system_tps)` over your own frontier | Recompute after every point has run |
| `system-description-consistency` fails | Points describe different systems | One frontier, one system. Freeze the stack before the first run |
| `config-consistency-dataset` fails | Points used different datasets | Re-run the odd ones out |
| `metric-consistency-*` fails | A stored `system_tps`, `tps_per_user` or `system_tps_per_kw` disagrees with the derived value | Don't hand-edit result files |

## Warnings, where objections come from

These don't stop the checker, but reviewers see them and Weeks 1–3 is when they act.

| Symptom | Cause | Fix |
|---|---|---|
| `point-duration` warns | A point ran under its minimum steady-state duration | Re-run: 600 s at concurrency 1–32, otherwise 1,200 s (2,400 s agentic) |
| `offline-ordering` warns | Offline `system_tps` under 0.98× the `C_max` point's, or Offline concurrency under `C_max` | Re-run the Offline point so it saturates the system, or elect `C_max` instead |
| `power-estimated` warns | A component group in `system_power.json` has no count or TDP | Fill it in from a public spec sheet, or accept the "MLC Estimated Power" tag |
| `region-placement` warns | Declared `region` disagrees with the computed one | Correct the declared value |
| `warmup-salt` warns | Warmup salt is enabled | Expected if warmup used the performance dataset — but be ready to explain it |

## Tooling failures before you even submit

| Symptom | Cause | Fix |
|---|---|---|
| *"missing required file(s): performance/result_summary.json"* | Flat run folder with the summary at top level | Use the layout the reference client writes. Flat layouts are rejected |
| Build fails naming a run | Builder cannot tell if the run is accuracy or performance | Supply `config.yaml`, or set `dataset_type` to exactly `Accuracy` or `Performance`. `Accuracy + Performance` describes the dataset, not the run |
| `submissions update --run-ids` rejected | The list would **add** a run | Adding points after submission was removed. The list may only shrink |
| Authentication error on every command | Token missing or wrongly scoped | Scope the [PRISM key](../understand/eligibility/prism-api-key.md) to **MLPerf Endpoints** |

## Rejections that come from judgement, not checks

Reviewers look for what automation cannot see: implausible frontier shapes, suspiciously uniform TTFT,
fake first tokens, padded token counts, contaminated warmup, Offline reordering across passes, and
system descriptions that don't match what ran. The full list is [§9.2 of the rules][rules-9.2], and
each concern becomes one of the objection types in [Submission Rules §6.8][srules-6.8], not a
checker rule. Power figures without a public, verifiable source
([Methodology][rules-4.5.2-methodology]), the wrong division and the wrong availability status are
raised the same way.

!!! tip "Division and availability objections usually mean reclassification"
    The committee may let you reclassify (Available to Preview, or to a different division) rather
    than withdraw ([Submission Rules §6.8][srules-6.8]). That's the usual outcome.

## The rejection nobody plans for

!!! danger "Not responding"
    At **10 business days** without a response to a filed objection, your submission is
    **withdrawn** ([Submission Rules §6.3][srules-6.3]). No rule was broken, no metric was wrong.
    This is the cheapest failure to avoid and it needs exactly one thing: a named person who is
    watching the review thread for six weeks. See [After you
    submit](../workflow/after-submission.md).

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@main` (4ba8224), 2026-10-30, and
`mlcommons/endpoints-submission-cli@main` (a42a056, `v1.1.0.0`), 2026-10-02.*
