# Metrics and regions

Metric definitions and the region-boundary algorithm. Conceptual background:
[What is MLPerf Endpoints?](../understand/what-is-mlperf-endpoints.md). Applied in
[step 3](../workflow/plan-your-curve.md).

## Primary metrics

Captured at each measurement point, at a specific concurrency level. The definitions, including
how each one is computed, are in [rules §4.1][rules-4.1]. The field names are the ones in your
result files:

| Metric | Field |
|---|---|
| System tokens per second | `system_tps` |
| TPS per user | `tps_per_user` |
| Time to first token (P90) | `ttft_p90_ms` |
| E2E average interactivity, **agentic benchmarks only** | `e2e_avg_interactivity` |
| Concurrency | `concurrency` |

!!! note "TTFT doesn't apply to a dedicated Offline run"
    Every query is queued at the start, so TTFT is unbounded in principle. `ttft_p90_ms` isn't
    required for a dedicated Offline run and can't be used as a compliance criterion or an objection
    against it. An elected `C_max` point reports TTFT as normal.

## Normalized metric

`system_tps_per_kw` divides `system_tps` by the provisioned power from the system's
[`system_power.json`](system-power-json.md). Definition and scope: [rules §4.5.3][rules-4.5.3].

The denominator is the **provisioned** power of the whole system and is the same at every point,
so the normalized curve is the throughput curve scaled by one constant.

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

[Rules §4.4][rules-4.4] sets the conditions and defines the coverage `status` values and the
detector's verdicts. In outline: the window has to span at least 4 super-passes, every gating
metric has to be flat, and the window's issue-time span has to meet the
[minimum run duration][rules-6.2] for the point's region. When any of that fails, the point's
official result falls back to the whole-run `total` metrics.

At high concurrency the duration floor is the one that binds, because 4 super-passes can finish
well inside the minimum.

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

The charts, and which metric goes on each axis, are listed in [rules §4.2][rules-4.2]. The primary
chart plots `system_tps` against `tps_per_user`, or against `e2e_avg_interactivity` for an agentic
benchmark. Higher is better on both axes.

The official curve is a **step function**. Each point is a discrete step, and between points the
curve holds at the last measured value. No interpolation, curve fitting or smoothing. Tools may
overlay a smoothed curve if labelled *"interpolated (not official)"*, but it cannot replace the
step function.

How the curve is drawn is in [§5.2][rules-5.2]. A dedicated Offline run appears as the
**throughput ceiling**, labelled *Offline* and drawn differently from the fixed-concurrency points. It doesn't define a step. An elected Offline result
adds no marker: the `C_max` point is labelled as the Offline result as well.

## Token counting

Official output token counts come from the **client-side reference tokenizer**, the one published
with the model in its Hugging Face repository. It's applied **once** to the
reconstructed assistant message through the model's official reference chat template.

Which parts of a response are counted — visible output, tool calls, reasoning, and chat-template
framing — and the baseline subtraction that removes empty-message framing are defined in
[rules §2.8][rules-2.8].

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

Beyond `C_min`, the space up to `C_max` is divided into three equal regions **in log-2 space**.
The reference algorithm, including the 10% margin, is in [rules §5.5][rules-5.5]. Rounding is
half-to-even, which is Python's built-in `round()` and easy to get wrong by hand.

Don't implement it yourself. Use the checker, which implements it:

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

The rules cover `C_max` ≤ 33, very large `C_max` and boundary collisions at the end of
[Concurrency Regions in §5.4][rules-5.4-concurrency]. The one that needs action from you is `C_max` ≤ 33: you must notify the working
group with written justification.

## Pre-computed boundaries

A quick-reference table for common `C_min` and `C_max` combinations is in
[Appendix B of the rules][rules-appendix-b]. For any other combination, run
`submission-checker regions`.

## Point counts

How many points you need, and how many carry accuracy results, is set by
[rules §5.3][rules-5.3], with the Offline point in [§5.7][rules-5.7] and the 32-point cap in
[§5.6][rules-5.6]. [Step 3](../workflow/plan-your-curve.md) walks through applying them.

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints@main` (e71b928), 2026-09-24.*
