# Why submissions get rejected

Failure modes as **symptom → cause → fix**, each linked to the step where it is avoided.

!!! note "Derived from the rules, not from observed submissions"
    MLPerf Endpoints has not yet completed a submission round under these rules. Every entry below is
    derived from a stated rejection action in the compliance rules, a checker rule, or a documented
    tool failure mode — **not** from failures seen in the wild. Once the first round supplies real
    data, this page should be rewritten from support threads and reviewer findings, and speculative
    entries deleted.

## The hard stops

These have a failure action of **reject the submission** at automated compliance. There is no
patching in place — you correct and resubmit as a new submission, losing your cohort slot.

| Symptom | Cause | Fix |
|---|---|---|
| `point-count` fails | Fewer than 8 points with a dedicated Offline run, or fewer than 7 otherwise | Run more points. Note withdrawn points do not count and **cannot be replaced** — [plan a spare](../workflow/plan-your-curve.md) |
| `offline-point-present` fails | Two points declare `offline`, or `elected` sits on a point that isn't at `C_max` | Keep one declaration. Elect only the point at your declared `C_max` |
| No Offline point (non-agentic) | Nothing declares `offline`. **Locally this is only a warning** | Run a dedicated Offline point or elect `C_max` — [step 3](../workflow/plan-your-curve.md#6-decide-how-to-meet-the-offline-requirement) |
| `power-descriptor` fails | No `system_power.json` for a system, or one with nothing a total can be derived from | Author it — [step 5](../workflow/author-disclosures.md#3-write-system_powerjson-for-each-system) |
| `*-concurrency-coverage` fails | No point in Low, Medium or High Concurrency | Recompute boundaries with `submission-checker regions` and run the missing region. Remember the 10% margin is **not** High Concurrency |
| `ultra-low-concurrency-coverage` fails | No point at concurrency ≤ 32 | Run one in 1–32 |
| `max-concurrency-declared` fails | `max_supported_concurrency` missing, or ≤ 32 | Declare a `C_max` > 32 in `system_desc.json` |
| `accuracy-gate` fails | Accuracy run missed the benchmark quality target | No tolerance exists. Fix the configuration and re-run — and check whether an approximation under model equivalence pushed you under |
| `accuracy-present` / `accuracy-coverage` fails | No accuracy results in one of the four mandatory regions, or none at the Offline point | Run the missing validation — one at each mandatory region point and one at Offline, same stack as performance |
| `shared-path-resolution` fails | `shared_src` / `shared_docs` do not resolve under the submission root | Fix the pointers in every `point.yaml` — [step 5](../workflow/author-disclosures.md) |
| `seed-set-consistency` / `seed-set-membership` fails | Points record different seed sets, or a set MLCommons never published | Bind **one** published set and record it at every point |
| Required files missing | A point lacks `point.yaml`, `system_desc.json` or `result_summary.json` | [Step 5](../workflow/author-disclosures.md) — nothing generates these for you |
| `approved-drafter` / `drafter-approval-lead-time` fails | A point used speculative decoding with a drafter not on the approved list, or approved too recently | These reject the **points**. With no list published yet, re-run without speculation |

## The flags, where objections come from

These do not stop the automated checks, but reviewers see them and Weeks 1–3 is when they act.

| Symptom | Cause | Fix |
|---|---|---|
| `point-duration` warns | A point ran under its region's minimum steady-state duration | Re-run at 600 s (Ultra Low) or 1,200 s |
| `min-query-count` warns | Fewer completed queries than one dataset pass | Re-run with a longer issue window |
| `streaming-config` flags | `stream_all_chunks` not `true` | Set it — per-token timing depends on it |
| `load-pattern` flags | A point other than a dedicated Offline run used `max_throughput` or `poisson` | Only the fixed-concurrency pattern is valid there. Re-run, or declare `offline: dedicated` if it really is your Offline run |
| `offline-ordering` warns | Offline `system_tps` under 0.98× the `C_max` point's, or Offline concurrency under `C_max` | Re-run the Offline point so it saturates the system, or elect `C_max` instead |
| `power-estimated` warns | A component group in `system_power.json` has no count or TDP | Fill it in from a public spec sheet, or accept the "MLC Estimated Power" tag |
| `metric-consistency-tps-per-kw` fails | Stored `system_tps_per_kw` disagrees with `system_tps / provisioned_power_kw` | Don't hand-edit it |
| `region-placement` warns | Declared `region` disagrees with the computed one | Correct the declared value |
| `concurrency-in-range` flags | A point sits outside every valid region | Recompute boundaries — remember `C_min` is **derived from your own lowest point** |
| `warmup-present` flags | Warmup declaration incomplete | Declare all six fields |
| `warmup-salt` warns | Warmup salt is enabled | Expected if warmup used the performance dataset — but be ready to explain it |
| `tps-utilization` fails | Value is not `system_tps / max(system_tps)` over your own curve | Recompute after every point has run |
| `system-description-consistency` fails | Points describe different systems | One curve, one system. Freeze the stack before the first run |
| `config-consistency-dataset` fails | Points used different datasets | Re-run the odd ones out |
| `metric-consistency-*` fails | Stored `system_tps` or `tps_per_user` disagrees with the derived value | Do not hand-edit result files |

## Tooling failures before you even submit

| Symptom | Cause | Fix |
|---|---|---|
| *"missing required file(s): performance/result_summary.json"* | Flat run folder with the summary at top level | Use the layout the reference client writes. Flat layouts are rejected |
| Build fails naming a run | Builder cannot tell if the run is accuracy or performance | Supply `config.yaml`, or set `dataset_type` to exactly `Accuracy` or `Performance`. `Accuracy + Performance` describes the dataset, not the run |
| `submissions update --run-ids` rejected | The list would **add** a run | Adding points after submission was removed. The list may only shrink |
| Authentication error on every command | Token missing or wrongly scoped | Scope the PRISM key to **MLPerf Endpoints** |

## Rejections that come from judgement, not checks

Reviewers look for what automation cannot see. Each of these is an objection type, not a checker
rule.

| Concern | What triggers it |
|---|---|
| **Implausible curve shape** | Throughput that does not rise with concurrency to saturation and then plateau |
| **Manipulated distributions** | Suspiciously uniform TTFT across very different concurrency levels |
| **Fake first token** | Whitespace, control characters or punctuation emitted to stop the TTFT clock rather than as genuine response content |
| **Inflated token counts** | Content duplicated across response fields, or padding added to any field |
| **Contaminated warmup** | Warmup drew on performance-dataset samples — reviewers cross-check your retained logs |
| **Offline reordering across passes** | Batches in the Offline run made of repeated copies of the same sample |
| **Unsupported power figures** | Component power with no public, verifiable source, or a TDP below rated with no evidence |
| **Inaccurate system description** | The description does not match what actually ran |
| **Wrong division** | Serviced claimed for a non-GA or privileged endpoint; Standardized claimed without meeting equivalence |
| **Wrong availability** | Components that fail the four-point test at submission time |

!!! tip "Division and availability objections usually mean reclassification"
    The committee may let you reclassify (Available to Preview, or to a different division) rather
    than withdraw. That's the usual outcome.

## The rejection nobody plans for

!!! danger "Not responding"
    At **10 business days** without a response to a filed objection, your submission is **withdrawn**.
    No rule was broken, no metric was wrong. This is the cheapest failure to avoid and it needs
    exactly one thing: a named person who is watching the review thread for six weeks. See
    [After you submit](../workflow/after-submission.md).

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
