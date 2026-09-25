# Metrics and regions

Metric definitions and the region-boundary algorithm. Conceptual background: [What is MLPerf
Endpoints?](../understand/what-is-mlperf-endpoints.md). Applied in [step
3](../workflow/plan-your-curve.md).

## Primary metrics

Captured at each measurement point, at a specific concurrency level. The metrics, their field names
and how each one is computed are in [rules §4.1][rules-4.1].

!!! note "TTFT doesn't apply to a dedicated Offline run"
    `ttft_p90_ms` isn't required for a dedicated Offline run ([§5.7.1][rules-5.7.1]). An elected
    `C_max` point reports TTFT as normal.

## Normalized metric

`system_tps_per_kw` divides `system_tps` by the provisioned power from the system's
[`system_power.json`](system-power-json.md). Definition and scope: [rules §4.5.3][rules-4.5.3].

The denominator is the **provisioned** power of the whole system and is the same at every point, so
the normalized curve is the throughput curve scaled by one constant.

!!! warning "P90, not P95"
    v1.0 uses P90 ([§4.1][rules-4.1]). The v0.7 rules used P95, and some public MLCommons material
    still shows P95.

!!! danger "The first token must be genuine"
    Padding the start of a response to stop the TTFT clock early is not allowed, and manual
    reviewers look for it ([§4.1][rules-4.1], [§9.2][rules-9.2]).

TPOT is counted differently from `system_tps`: see the last bullets of [§2.8][rules-2.8].

## What your numbers are measured over

From v1.0 the official result is computed over a **steady-state window**, not the whole run, with
the whole-run `total` figures kept as supplementary. What the window excludes and why:
[§4.4][rules-4.4].

The window is found after the run, from `events.jsonl`, so you don't do anything during the run to
produce it. It does change how you plan a run, though, because a run can fail to have one.

!!! warning "The detector is an ad-hoc script for now"
    `scripts/steady_state_diagnostics.py` is on `mlcommons/endpoints` `main` (merged 2026-09-16),
    and you run it yourself over a run directory or its `events.jsonl`:

    ```bash
    uv run scripts/steady_state_diagnostics.py <run_dir>/
    ```

    It isn't wired into `inference-endpoint` yet, so a run doesn't produce the `steady_state` block
    by itself; you fill it in from the script's output. The script's own documentation scopes it to
    single-turn workloads; for agentic runs it prints *not yet supported*. Tracked as **B9** in
    [Open questions](../help/open-questions.md).

### When the steady-state result counts as official

[Rules §4.4][rules-4.4] sets the conditions and defines super-passes, the coverage `status` values
and the detector's verdicts. In outline: the window has to span at least 4 super-passes (by default,
one super-pass is one pass over the dataset), every gating metric has to be flat, and the window's
issue-time span has to meet the [minimum run duration][rules-6.2] for the point's region. When any
of that fails, the point's official result falls back to the whole-run `total` metrics.

!!! tip "What this means in practice"
    Run longer than the minimum. A run that only just clears 1,200 s can still come back
    `insufficient_passes` or `not found`, and then your published number is the whole-run average —
    ramp-up, drain and all — which is worse than what your system actually does.

!!! warning "Only fixed-concurrency points are in scope"
    The Scope paragraph of [§4.4][rules-4.4] leaves `MaxThroughput`, `Poisson` and single-pass
    agentic workloads out. That leaves a dedicated Offline run reporting its whole-run `total`
    metrics. The rules imply this rather than stating it. Tracked in [Open
    questions](../help/open-questions.md) under **C7**.

!!! question "Which metrics gate is still not settled"
    The rules' definition table and the detector's own documentation on `main` now agree on **TPOT
    at P50 and P90**, with TTFT as a diagnostic and drift warning only. But the next paragraph of
    rules §4.4 still says **TTFT and TPOT at P50/P90**. Tracked as **B8**.

    Also pending ratification: whether the 4 super-pass floor rises, and whether a `not found` run
    is *invalid* rather than merely reported-with-flags. See [Open
    questions](../help/open-questions.md).

## Publication charts

The charts, and which metric goes on each axis, are listed in [rules §4.2][rules-4.2]. The primary
chart plots `system_tps` against `tps_per_user`, or against `e2e_avg_interactivity` for an agentic
benchmark. Higher is better on both axes.

The official curve is a **step function** with no interpolation ([§5.2][rules-5.2]). How a dedicated
or elected Offline result is drawn on it is in [§5.7.3][rules-5.7.3].

## Token counting

Official token counts, output and ISL alike, come from the client applying the model's reference
tokenizer once, through its reference chat template. Your server's own tokenizer doesn't affect
scoring. What is counted, and how ISL is built: [rules §2.8][rules-2.8].

## Region boundaries

The concurrency space is divided into four regions. Boundaries are **specific to your submission**:
computed from your own `C_min` and `C_max`.

### Ultra Low Concurrency

Fixed at concurrency **1 to 32 inclusive** for every submission, one point required ([Ultra Low
Concurrency Region in §5.4][rules-5.4-ultra-low]).

### The three concurrency regions

Beyond `C_min`, the space up to `C_max` is divided into three equal regions **in log-2 space**. The
reference algorithm, including the 10% margin, is in [rules §5.5][rules-5.5]. Rounding is
half-to-even, which is Python's built-in `round()` and easy to get wrong by hand.

Don't implement it yourself. Use the checker, which implements it:

```bash
python -c "from submission_checker.cli import main; main()" regions --max-concurrency 1024 --min-concurrency 16
```

!!! note "`C_min` is derived in v1.0"
    The checker derives `C_min` from your submission's own points rather than reading a declared
    value, so boundaries differ per curve. Your lowest point silently sets every other boundary.

### The 10% margin

The High Concurrency region carries a margin extending the valid upper bound to `ceil(1.10 × C_max)`
([Concurrency Regions in §5.4][rules-5.4-concurrency]).

!!! warning "The margin is its own region"
    A point in the margin is legal but does **not** satisfy High Concurrency coverage. The rules
    still say the margin is there for adding points after submission, but the Submission Rules no
    longer provide that window.

### Edge cases

The rules cover `C_max` ≤ 33, very large `C_max` and boundary collisions at the end of [Concurrency
Regions in §5.4][rules-5.4-concurrency]. The one that needs action from you is `C_max` ≤ 33: you
must notify the working group with written justification.

## Pre-computed boundaries

A quick-reference table for common `C_min` and `C_max` combinations is in [Appendix B of the
rules][rules-appendix-b]. For any other combination, run the checker's `regions` command, as above.

## Point counts

How many points you need, and how many carry accuracy results, is set by [rules §5.3][rules-5.3],
with the Offline point in [§5.7][rules-5.7] and the 32-point cap in [§5.6][rules-5.6]. [Step
3](../workflow/plan-your-curve.md) walks through applying them.

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints@main` (e71b928), 2026-09-24.*
