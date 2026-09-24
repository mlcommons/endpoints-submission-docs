# What is MLPerf Endpoints?

MLPerf Endpoints measures how well an **inference endpoint serves generative AI models**. A
separate client sends load to your model-serving API over HTTP. What gets tested is the endpoint
itself, not a framework integration.

## What it measures

Endpoints measures a curve, not a single number. MLPerf Inference reports latency or throughput
at one operating point. Real serving systems don't
have one operating point — they have a tradeoff between total throughput and per-user speed, and
you choose where to sit on it. Endpoints measures that whole tradeoff as a Pareto curve across
concurrency levels.

Four dimensions are captured at each measurement point:

| Metric | Field | What it is |
|---|---|---|
| System throughput | `system_tps` | Total output tokens per second across all concurrent users |
| Interactivity | `tps_per_user` | Per-user output rate, derived as `1000 / tpot_p90_ms` |
| First-token latency | `ttft_p90_ms` | P90 milliseconds from query issue to the first non-empty text fragment |
| Load | `concurrency` | Target number of in-flight queries at that point |

The main published chart plots `system_tps` against `tps_per_user`: total capacity against
per-user experience. Agentic benchmarks swap the X-axis for `e2e_avg_interactivity`, the output
rate across completed turns with tool-call time left out. Full definitions in
[Metrics and regions](../reference/metrics-and-regions.md).

!!! warning "P90, not P95"
    v1.0 requires `ttft_p90_ms`. The v0.7 rules used P95, and some public MLCommons material still
    shows P95. Only P90 is plotted in the v1.0 publication chart. You may report additional
    percentiles in your `point.yaml`; they will not be charted.

## Why a step function

The official curve is a **step function**. Each submitted point defines a discrete step; between
points the curve holds at the last measured value. No interpolation, no curve fitting, no
smoothing.

A smooth line would suggest operating points you never actually measured, and the gaps between
points are where problems like memory pressure and latency spikes tend to show up. Tools can overlay
a smoothed curve for readability, but it has to be labelled *interpolated (not official)* and can't
replace the step function.

## What a submission actually is

One submission is one **Pareto curve**: one system, one benchmark model, one dataset. For most
benchmarks it has **8 to 32** measurement points, structured `1 + 3 + 3 + 1`:

- **1** point in the Ultra Low Concurrency region (concurrency 1–32),
- **1 each** in the Low, Medium and High Concurrency regions,
- **3** anywhere in those three regions, at your discretion,
- **1** Offline point (see below).

Agentic benchmarks drop the Offline point, so they need 7.

Region boundaries differ between submitters. They're calculated from your own minimum and maximum
concurrency, which is covered in [Plan your Pareto curve](../workflow/plan-your-curve.md). Pick your
maximum badly and you'll have to re-run points.

Each point is a sustained run: **600 seconds** of steady state at ultra-low concurrency, **1,200
seconds** elsewhere, not counting warmup.

## The Offline point

Every other point holds a fixed number of queries in flight. The Offline point doesn't: the client
hands the system the whole dataset at the start and lets it work through the queue as fast as it
can. It's the Endpoints version of the Offline scenario in MLPerf Inference, and it answers one
question: how much total throughput can this system sustain when nothing is pacing it?

A few things follow from that.

- **Only throughput counts.** Queries at the back of the queue can wait a long time for their
  first token, so TTFT isn't required for this point and can't be used to object to it.
- **Its concurrency is the dataset size.** The reported `concurrency` is the number of queries in
  one pass over the performance dataset. You don't choose it.
- **It should be your ceiling.** A dedicated Offline run has to reach at least 98% of the
  throughput of your `C_max` point. If it can't, one of the two runs didn't measure what it
  claims to.
- **Reordering is allowed, within a pass.** The system can sort and batch the queue however it
  likes, but it can't mix queries from different passes over the dataset. Doing that would let
  repeated copies of the same prompt run together, which says more about the replay than about
  the system.

If your `C_max` point already is the most throughput your system can deliver, you can **elect** it
as the Offline result instead of running a separate Offline point. That's the 7-point route for a
non-agentic benchmark. The rules allow it without verification because it can only understate
your system. How to choose between the two is in
[Plan your Pareto curve](../workflow/plan-your-curve.md#6-decide-how-to-meet-the-offline-requirement).

## Normalised by provisioned power

A larger system will always post a bigger `system_tps` than a smaller one. To make results
comparable across scale, v1.0 also divides throughput by the system's **provisioned power**:

```
system_tps_per_kw = system_tps / provisioned_power_kw
```

Provisioned power is what the system is built to draw, not what a meter read during the run. It's
estimated from the rated power of the CPUs, accelerators and scale-up switches, plus a fixed
overhead for everything else: 30% for liquid-cooled systems, 50% for air-cooled. You supply the
inputs, from public spec sheets, in a `system_power.json` file per system. Anything you leave out,
MLCommons fills in from its own conservative estimates, and the result is then tagged
**"MLC Estimated Power"**.

The figure is fixed for the system. A concurrency-1 point that leaves most of the accelerators
idle is still divided by the power of the whole system.

This is required for the Standardized division, optional for RDI, and not yet defined for
Serviced. See [Requirements you must meet](../rules/requirements.md#power-normalization).

## How tokens are counted

Token counts don't come from your serving stack. The reference client rebuilds the full response —
visible output, reasoning traces and tool calls — formats it with the model's official chat
template, and counts it **once** using the reference tokenizer from the model's Hugging Face
repository.

This means every submitter is measured the same way, no matter how their system batches or streams
output. It also means your published `system_tps` may differ from the number your serving framework
reports, which is expected. You can use any tokenizer internally; it doesn't affect your score.

## Accuracy is a hard gate

Each benchmark has a quality target, and you have to hit it at several points on the curve, not
once for the whole submission. Accuracy runs are required at the four mandatory region points
(Ultra Low, Low, Medium and High Concurrency) and at the Offline point: five in all, or four for an
agentic benchmark, which has no Offline point. Every accuracy run uses the same endpoint
configuration, weights and software stack as your performance runs.

How those runs are judged depends on the benchmark:

| Benchmark type | The gate |
|---|---|
| Single-turn | **Every** result must meet the target. Each run uses the same concurrency as its point, on the same instance, immediately after that point's performance run |
| Multi-turn | The **average** of the results must meet the target. Individual results may fall short, and the concurrency may differ |

Throughput results are allowed some variation between runs. Accuracy isn't: miss the target and
the submission is rejected.

!!! warning "This changed twice in September"
    Until early 2026-09 the rules asked for one accuracy run per submission. Then it became four.
    Since the Offline point became mandatory for non-agentic benchmarks on 2026-09-22, it's five.
    If you planned hardware time against an older number, re-plan.

**Next:** [How submission works](how-submission-works.md)

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
