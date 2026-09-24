# Metrics and regions

Metric definitions and the region-boundary algorithm. Conceptual background:
[What is MLPerf Endpoints?](../understand/what-is-mlperf-endpoints.md). Applied in
[step 3](../workflow/plan-your-curve.md).

## Primary metrics

Captured at each measurement point, at a specific concurrency level.

| Metric | Field | Definition |
|---|---|---|
| System tokens per second | `system_tps` | `total_output_tokens / elapsed_duration_seconds` — total output tokens per second across all concurrent users |
| TPS per user | `tps_per_user` | `1000 / tpot_p90_ms`, where `tpot_p90_ms` is the P90 of valid per-response TPOT samples. Higher is better |
| Time to first token | `ttft_p90_ms` | P90 milliseconds from query issuance until the client receives the first non-empty text fragment (`len(s) > 0`) in **any** response category — visible output, tool call, or reasoning |
| E2E average interactivity | `e2e_avg_interactivity` | **Agentic benchmarks only.** Output tokens summed across every completed turn, divided by the summed server-side turn time: `sum(output_tokens_per_turn) / sum(e2e_turn_time_seconds)`. Turn time runs from request receipt to response completion and **excludes tool-call execution**. One scalar per point, across all trajectories |
| Concurrency | `concurrency` | Target in-flight concurrent queries for this point. For a dedicated Offline run, the number of queries in one pass over the performance dataset |

!!! note "TTFT doesn't apply to a dedicated Offline run"
    Every query is queued at the start, so TTFT is unbounded in principle. `ttft_p90_ms` isn't
    required for a dedicated Offline run and can't be used as a compliance criterion or an objection
    against it. An elected `C_max` point reports TTFT as normal.

## Normalized metric

| Metric | Field | Definition |
|---|---|---|
| Total system throughput per kilowatt | `system_tps_per_kw` | `system_tps / provisioned_power_kw`, where `provisioned_power_kw` comes from the system's [`system_power.json`](system-power-json.md) |

The denominator is the **provisioned** power of the whole system and is the same at every point.
The normalized curve is therefore the throughput curve scaled by one constant. Required for
Standardized, optional for RDI, not yet defined for Serviced.

!!! warning "P90, not P95"
    Only `ttft_p90_ms` is required per point, and only P90 is plotted in the v1.0 publication chart.
    The v0.7 rules used P95; some public MLCommons material still shows P95. Additional percentiles
    may be reported but are not charted.

!!! danger "The first token must be genuine"
    Emitting whitespace, control characters, punctuation or other meaningless leading content solely
    to stop the TTFT clock, rather than as a genuine part of the model response, is not allowed,
    and manual reviewers look for it specifically.

### How TPOT differs from the others

`system_tps` derives from the single assistant-payload token count. **TPOT instead uses the suffix
after the first output-bearing streamed chunk**, tokenized once with the reference tokenizer.
Non-positive samples, non-finite samples and non-streaming responses are excluded. TTFT is a latency
measurement and is not derived from token counts at all.

## What your numbers are measured over

Up to v0.7, a point's metrics were averaged over the whole run after `TEST_STARTED`. That window
included two things that skew the result, and warmup removed neither:

- **Ramp-up** at the start, while in-flight concurrency and queue depth are still climbing to
  target. It inflates the TTFT tail.
- **Drain** at the end, while the last queries finish and no new ones are issued. It deflates
  throughput.

From v1.0 the official result is computed over a **steady-state window** instead — the stable
middle of the run. Whole-run figures (`total`) are still reported, but as supplementary.

A post-processing step reads the event log (`events.jsonl`) after the run and finds the window, off
the measured path — you don't do anything during the run to produce it. It does change how you plan
a run, though, because a run can fail to have one.

!!! warning "The detector is an ad-hoc script for now"
    `scripts/steady_state_diagnostics.py` is on `mlcommons/endpoints` `main` (merged 2026-09-16),
    and you run it yourself over a run directory or its `events.jsonl`:

    ```bash
    uv run scripts/steady_state_diagnostics.py <run_dir>/
    ```

    It isn't wired into `inference-endpoint` yet, so a run doesn't produce the `steady_state`
    block by itself; you fill it in from the script's output. The script's own documentation scopes it
    to single-turn workloads; for agentic runs it prints *not yet supported*. Tracked as **B9** in
    [Open questions](../help/open-questions.md).

### Super-passes

The window is measured in **super-passes**, not seconds. One super-pass is a contiguous block of
queries in issue order, sized to one full pass over the dataset — unless the benchmark definition
sets a different size.

### When the steady-state result counts as official

All three have to hold:

1. The window spans **at least 4 super-passes**. The trend test needs 4 samples (`MIN_TREND_N = 4`),
   so the run needs more than 4 super-passes in total.
2. Every gating metric is a **Plateau** — flat, no significant trend across the super-passes — and
   not drifting up.
3. The window's **issue-time span** meets the minimum run duration for the point's region: 600 s
   Ultra Low, 1,200 s elsewhere.

The effective floor is therefore `max(4 super-passes, your region's minimum duration)`. At high
concurrency the duration binds, because 4 super-passes can finish well inside 1,200 s.

### If it doesn't hold

The point falls back by coverage `status`:

| `status` | When | Official result |
|---|---|---|
| `windowable` | ≥ 4 super-pass window in Plateau, **and** its issue-time span meets the minimum duration | Steady-state metrics; `total` supplementary |
| `insufficient_duration` | ≥ 4 super-passes in Plateau, but the span is under the minimum duration | `total`. Steady-state reported as low-confidence, not official |
| `insufficient_passes` | ≥ 1 super-pass, but under 4 | `total`. Steady-state reported as low-confidence, not official |
| `partial_dataset` | Under 1 super-pass | `total` only — no steady-state claim at all |

### What the detector reports

Separately from coverage, the detector classifies the **shape** of the run and emits one verdict:

| Shape | Verdict | What gets reported |
|---|---|---|
| Gated metrics stable across the window | `STEADY STATE` | Steady-state metrics; `total` supplementary. **This is the one you want** |
| A gated metric keeps climbing after the window | `drifting_up` | Reported as a drift range or slope, never a single number. Window flagged *local-plateau only* |
| A gated metric trends down over the tail | `drifting_down` | Reported as drift, not a single number |
| The first plateau steps to a later, different one | `anomaly` (staircase) | The **first** plateau is the steady state; the later shift is disclosed as likely degradation |
| Nothing is steady enough, or the run is too short | `not found` | No steady-state claim. Falls back to whole-run `total` |

!!! tip "What this means in practice"
    Run longer than the minimum. A run that only just clears 1,200 s can still come back
    `insufficient_passes` or `not found`, and then your published number is the whole-run average —
    ramp-up, drain and all — which is worse than what your system actually does.

!!! warning "Only fixed-concurrency points are in scope"
    `MaxThroughput`, `Poisson` and single-pass agentic workloads are handled only by the ad-hoc
    diagnostic tool, not by this reporting basis. That leaves a dedicated Offline run reporting its
    whole-run `total` metrics. The rules imply this rather than stating it. Tracked in
    [Open questions](../help/open-questions.md) under **C7**.

!!! question "Which metrics gate is still not settled"
    The rules' definition table and the detector's own documentation on `main` now agree on
    **TPOT at P50 and P90**, with TTFT as a diagnostic and drift warning only. But the next
    paragraph of rules §4.4 still says **TTFT and TPOT at P50/P90**. Tracked as **B8**.

    Also pending ratification: whether the 4 super-pass floor rises, and whether a `not found` run
    is *invalid* rather than merely reported-with-flags. See
    [Open questions](../help/open-questions.md).

## Publication charts

| Chart | Y-axis | X-axis |
|---|---|---|
| **Pareto curve** (primary, single-turn) | `system_tps` | `tps_per_user` |
| **Agentic Pareto curve** (primary, agentic) | `system_tps` | `e2e_avg_interactivity` |
| System TPS vs. concurrency | `system_tps` | `concurrency` |
| TTFT (P90) vs. concurrency | `ttft_p90_ms` | `concurrency` |
| Interactivity vs. concurrency | `tps_per_user` | `concurrency` |

Agentic benchmarks use `e2e_avg_interactivity` in place of `tps_per_user` as the primary chart's
X-axis. Higher is better on both axes either way.

The official curve is a **step function**. Each point is a discrete step, and between points the
curve holds at the last measured value. No interpolation, curve fitting or smoothing. Tools may
overlay a smoothed curve if labelled *"interpolated (not official)"*, but it cannot replace the
step function.

A dedicated Offline run appears as the **throughput ceiling**, labelled *Offline* and drawn
differently from the fixed-concurrency points. It doesn't define a step. An elected Offline result
adds no marker: the `C_max` point is labelled as the Offline result as well.

## Token counting

Official output token counts come from the **client-side reference tokenizer**, the one published
with the model in its Hugging Face repository. It's applied **once** to the
reconstructed assistant message through the model's official reference chat template.

| Content category | Counted? | How |
|---|---|---|
| Visible output | Yes | Fragments concatenated in arrival order, supplied as assistant `content` |
| Tool-call content | Yes | Fragments reassembled into structured calls, ordered by ascending tool-call index, supplied as `tool_calls` |
| Reasoning / thinking | Yes | Fragments concatenated in arrival order, supplied as the reasoning field |
| Chat-template framing | Conditional | Payload-specific framing is counted; framing present for an *empty* assistant message is subtracted out |

The baseline subtraction is explicit: render and tokenize both (a) a minimal conversation with an
empty user message followed by the reconstructed assistant response, and (b) the same conversation
with an empty assistant message. The official count is `max(0, count(a) - count(b))`, both rendered
with `add_generation_prompt = false`.

You may use any tokenizer internally; it does not affect scoring, and no equivalence demonstration
is required for an internal *output* tokenizer. Input tokenization equivalence requirements still
apply.

**ISL** is computed by applying all required input preprocessing — request construction, salting,
the official reference chat template, then tokenizing with the reference tokenizer. Special tokens
inserted by the template are included. If the reference implementation truncates, ISL is the
post-truncation length.

## Region boundaries

The concurrency space is divided into four regions. Boundaries are **specific to your submission**:
computed from your own `C_min` and `C_max`.

### Ultra Low Concurrency

Fixed at concurrency **1 to 32 inclusive** for every submission, so every curve has at least one
directly comparable point. One point required.

!!! note "Bounds are final for v1.0"
    The 1–32 bounds are final for Endpoints v1.0, though the working group may adjust them in a
    future rules version.

### The three concurrency regions

Beyond `C_min`, the space up to `C_max` is divided into three equal regions **in log-2 space**:

```python
def compute_regions(C_max: int, C_min: int) -> dict:
    assert 1 <= C_min <= 32, "Minimum concurrency must be between 1 and 32 (inclusive)"
    assert C_max > 32,       "Maximum Supported Concurrency must be > 32"

    low_latency = {"start": 1, "end": C_min}

    I = math.log2(C_max - C_min) / 3

    low_conc_end = round(C_min + 2**I)
    med_conc_end = round(C_min + 2**(2 * I))

    low_concurrency  = {"start": C_min + 1,       "end": low_conc_end}
    med_concurrency  = {"start": low_conc_end+1,  "end": med_conc_end}
    high_concurrency = {"start": med_conc_end+1,  "end": C_max}

    margin_end = math.ceil(1.10 * C_max)

    return {
        "low_latency":      low_latency,
        "low_concurrency":  low_concurrency,
        "med_concurrency":  med_concurrency,
        "high_concurrency": high_concurrency,
        "margin":           {"start": C_max+1, "end": margin_end},
    }
```

Rounding is **half-to-even** (banker's rounding), matching Python's built-in `round()`.

Don't implement this yourself. Use the reference implementation:

```bash
submission-checker regions --max-concurrency 1024 --min-concurrency 16
```

!!! note "`C_min` is derived in v1.0"
    The checker derives `C_min` from your submission's own points rather than reading a declared
    value, so boundaries differ per curve. Your lowest point silently sets every other boundary.

**Why log spacing?** The difference between concurrency 1 and 10 is far more significant than
between 1000 and 1010. Log-space division makes each region represent a similarly meaningful range
of behavioural change regardless of absolute scale.

### The 10% margin

The High Concurrency region carries a margin extending the valid upper bound to
`ceil(1.10 × C_max)`.

!!! warning "The margin is its own region"
    A point in the margin is legal but does **not** satisfy High Concurrency coverage. The margin
    originally existed to allow adding points after submission; that window has since been removed.

### Edge cases

| Case | Behaviour |
|---|---|
| `C_max` ≤ 33 | All three regions collapse to roughly one level each. You must notify the working group with written justification, and they may request more information before accepting |
| `C_max` > 100,000 | The algorithm scales correctly. Low Concurrency is narrow; High Concurrency spans most of the range |
| Boundary collision | If rounding makes two boundaries equal, that region has zero width and a single valid level at the boundary value. One point there satisfies it |

## Pre-computed boundaries

??? abstract "Quick-reference table by `C_min` and `C_max`"

    | `C_max` | `C_min` | Low | Medium | High | 10% margin |
    |---|---|---|---|---|---|
    | 64 | 2 | 3–6 | 7–18 | 19–64 | 65–71 |
    | 128 | 2 | 3–7 | 8–27 | 28–128 | 129–141 |
    | 256 | 2 | 3–8 | 9–42 | 43–256 | 257–282 |
    | 256 | 8 | 9–14 | 15–47 | 48–256 | 257–282 |
    | 512 | 8 | 9–16 | 17–71 | 72–512 | 513–564 |
    | 1,024 | 8 | 9–18 | 19–109 | 110–1,024 | 1,025–1,127 |
    | 512 | 16 | 17–24 | 25–79 | 80–512 | 513–564 |
    | 1,024 | 16 | 17–26 | 27–117 | 118–1,024 | 1,025–1,127 |
    | 2,048 | 16 | 17–29 | 30–176 | 177–2,048 | 2,049–2,253 |
    | 1,024 | 32 | 33–42 | 43–131 | 132–1,024 | 1,025–1,127 |
    | 2,048 | 32 | 33–45 | 46–192 | 193–2,048 | 2,049–2,253 |
    | 4,096 | 32 | 33–48 | 49–287 | 288–4,096 | 4,097–4,506 |
    | 8,192 | 32 | 33–52 | 53–437 | 438–8,192 | 8,193–9,012 |
    | 16,384 | 32 | 33–57 | 58–676 | 677–16,384 | 16,385–18,023 |

    The Low Latency point is a single point at your `C_min`. All concurrency regions are
    submission-specific and depend on both values.

## Point counts

| | Non-agentic | Agentic |
|---|---|---|
| Minimum | **8** points, `1 + 3 + 3 + 1`, with a dedicated Offline run. **7** if the `C_max` point is elected as the Offline result | **7** points, `1 + 3 + 3`. No Offline point |
| Accuracy points | **5**: the four mandatory region points and Offline | **4**: the four mandatory region points |
| Maximum | **32** points, Offline included | **32** points |
| Spacing | No requirements — cluster or spread as you choose | Same |

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints@main` (e71b928), 2026-09-24.*
