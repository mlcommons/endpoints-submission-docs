# What is MLPerf Endpoints?

MLPerf Endpoints measures how well an **inference endpoint serves generative AI models**. A separate
client sends load to your model-serving API over HTTP. What gets tested is the endpoint itself, not
a framework integration.

## What it measures

Endpoints measures a curve, not a single number. MLPerf Inference reports latency or throughput at
one operating point. Real serving systems don't have one operating point — they have a tradeoff
between total throughput and per-user speed, and you choose where to sit on it. Endpoints measures
that whole tradeoff as a Pareto curve across concurrency levels.

Four dimensions are captured at each measurement point ([§4.1][rules-4.1]):

| Metric | Field | What it is |
|---|---|---|
| System throughput | `system_tps` | Total output tokens per second across all concurrent users |
| Interactivity | `tps_per_user` | Per-user output rate, derived as `1000 / tpot_p90_ms` |
| First-token latency | `ttft_p90_ms` | P90 milliseconds from query issue to the first non-empty text fragment |
| Load | `concurrency` | Target number of in-flight queries at that point |

The main published chart plots `system_tps` against `tps_per_user`: total capacity against per-user
experience. Agentic benchmarks swap the X-axis for `e2e_avg_interactivity`, the output rate across
completed turns with tool-call time left out ([§4.2][rules-4.2]). Full definitions in [Metrics and
regions](../reference/metrics-and-regions.md).

!!! warning "P90, not P95"
    v1.0 requires `ttft_p90_ms` ([§4.1][rules-4.1]). The v0.7 rules used P95, and some public
    MLCommons material still shows P95. Extra percentiles in your `point.yaml` won't be charted.

## Why a step function

The official curve is a **step function**: each point is a step, and the curve holds at the last
measured value until the next one ([§5.2][rules-5.2]). A smooth line would suggest operating points
you never measured, and the gaps between points are where problems like memory pressure and latency
spikes tend to show up.

## What a submission actually is

One submission is one **Pareto curve**: one system, one benchmark model, one dataset. For most
benchmarks it has **8 to 32** measurement points, structured `1 + 3 + 3 + 1` ([§5.3][rules-5.3]):

- **1** point in the Ultra Low Concurrency region (concurrency 1–32),
- **1 each** in the Low, Medium and High Concurrency regions,
- **3** anywhere in those three regions, at your discretion,
- **1** Offline point (see below).

Agentic benchmarks drop the Offline point, so they need 7.

Region boundaries differ between submitters. They're calculated from your own minimum and maximum
concurrency, which is covered in [Plan your Pareto curve](../workflow/plan-your-curve.md). Pick your
maximum badly and you'll have to re-run points.

Each point is a sustained run: **600 seconds** of steady state at ultra-low concurrency, **1,200
seconds** elsewhere, not counting warmup ([§6.2][rules-6.2]).

## The Offline point

Every other point holds a fixed number of queries in flight. The Offline point doesn't: the client
hands the system the whole dataset at once and lets it drain the queue as fast as it can. It's the
Endpoints version of the Offline scenario in MLPerf Inference, and it measures the system's
throughput ceiling. Only throughput counts, and the reported concurrency is the dataset size
([§5.7.1][rules-5.7.1]).

You can either run it as a separate point or, if your `C_max` point already is your ceiling,
**elect** that point as the Offline result, which is the 7-point route ([§5.7.2][rules-5.7.2]). How
to choose is in [Plan your Pareto
curve](../workflow/plan-your-curve.md#6-decide-how-to-meet-the-offline-requirement).

## Normalised by provisioned power

A larger system will always post a bigger `system_tps` than a smaller one. To make results
comparable across scale, v1.0 also reports `system_tps_per_kw`: throughput divided by the system's
**provisioned power** ([§4.5.3][rules-4.5.3]). That's what the system is built to draw, estimated
from the rated power of its main components plus a fixed overhead ([Power
Model][rules-4.5.2-power-model]), not what a meter read during the run. You supply the inputs from
public spec sheets in a `system_power.json` per system. The figure is fixed for the system, so a
concurrency-1 point is still divided by the power of the whole system.

This is required for the Standardized division, optional for RDI, and not yet defined for Serviced.
See [Requirements you must meet](../rules/requirements.md#power-normalization).

## How tokens are counted

Token counts don't come from your serving stack. The reference client rebuilds the whole response,
formats it with the model's official chat template, and counts it once with the reference tokenizer
([§2.8][rules-2.8]). So your published `system_tps` may differ from the number your serving
framework reports, which is expected. The tokenizer you use internally doesn't affect your score.

## Accuracy is a hard gate

Each benchmark has a quality target, and you have to hit it at several points on the curve, not once
for the whole submission: the four mandatory region points and the Offline point, so five in all, or
four for an agentic benchmark ([§5.3][rules-5.3]). Single-turn benchmarks must pass at every one of
those points; multi-turn benchmarks must pass on the average ([§4.3][rules-4.3]). Throughput is
allowed some variation between runs. Accuracy isn't: miss the target and the submission is rejected.

!!! warning "This changed twice in September"
    Until mid-September 2026 the rules asked for one accuracy run per submission. Then it became
    four. Since the Offline point became mandatory for non-agentic benchmarks on 2026-09-22, it's
    five. If you planned hardware time against an older number, re-plan.

**Next:** [How submission works](how-submission-works.md)

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
