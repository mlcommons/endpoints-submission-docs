# 4. Run the measurement points

> Produces: one run folder per Pareto point, with accuracy results at the required points.

!!! note "Before you begin"
    - Completed [3. Plan your Pareto curve](plan-your-curve.md)
    - You have a list of legal concurrency levels
    - The endpoint under test is up and reachable
    - You know which seed set you are binding to — see [seed set and salt](#seeds-and-salting) below

## What you'll do

- Run each point at **fixed concurrency** for its region's minimum duration
- Run the dedicated **Offline** point, if you chose one in step 3
- Run accuracy validations at the four mandatory region points and the Offline point
- Keep every run folder

This is the expensive step. Everything up to now took minutes. This takes days.

## The binding run constraints

The run requirements are in [§6 of the rules][rules-6], and that's where the values are.

| Requirement | Rule | Where you set it |
|---|---|---|
| Load pattern | [§6.1][rules-6.1] | `settings.load_pattern.type`: `concurrency` for every point, `max_throughput` for a dedicated Offline run ([below](#run-the-offline-point)) |
| Minimum duration | [§6.2][rules-6.2] | Run length. It's measured over the steady-state window's issue time, not wall clock |
| Minimum completed queries | [§6.4][rules-6.4] | Sample count: whole passes over the dataset |
| Warmup | [§6.3][rules-6.3] | Your warmup procedure, declared in `point.yaml` |
| Sampling and streaming | [§6.5][rules-6.5] | `stream_all_chunks: true` on every performance run. The rule also fixes the sampling order for performance and accuracy runs |
| Consistency across points | [§9.1][rules-9.1] | Lock the model, endpoint config, software stack and seed set before the first point |
| Speculative decoding | [§2.9.4][rules-2.9.4] | Needs a drafter from the benchmark's approved list. None is published yet, so **leave it off** for now |

!!! tip "Run longer than the minimum"
    Your official numbers now come from a **steady-state window** the tooling detects inside the
    run, not from the whole run. A run that only just clears its minimum can fail to produce a
    usable window, and then the published number falls back to the whole-run average — ramp-up and
    drain included, which understates what your system does. See [What your numbers are measured
    over](../reference/metrics-and-regions.md#what-your-numbers-are-measured-over).

!!! warning "Configuration consistency is checked across the whole curve"
    Every point must describe the same system, the same model and the same dataset. A curve
    assembled from points run against two different software versions will be flagged and can be
    rejected. Lock your software stack before the first point and don't change it until the last one
    is done.

## Before your first run: seed set and salt {#seeds-and-salting}

Your submission binds to exactly **one seed set** when it first appears in a
[cohort](../understand/how-submission-works.md#rolling-submission-and-cohorts), and every point must
record that same set. Seed rotation never forces you to re-run. The binding and adoption window are
in [Submission Rules §4.6][srules-4.6].

The seeds also drive the per-query **salt**, which is what makes cross-query KV-cache reuse legal.
Accuracy runs use the **un-salted** dataset. See [§2.9.5.1][rules-2.9.5.1].

!!! warning "Warmup must not use performance-dataset samples"
    Warmup can't use performance-dataset samples in any form; the accuracy dataset is fine
    ([§6.3.1][rules-6.3.1]). If your client does use the performance dataset during warmup,
    **salting must be enabled**, and the salting flag is **not on by default**.

!!! success "The v1.0 seed set is published"
    As of 2026-09-15 the policies repo carries `seedset.yaml`: one set, **`id: A`**, published for
    cohort **`2026-10-C1`**. Since checker `v1.0.1.0` the checker bundles the same file, cohort key
    included, and `seed-set-adoption` tests your `target_cohort` against the four-cohort window for
    real.

    ```yaml
    seed_set: A
    target_cohort: 2026-10-C1
    ```

    Set `A` can be adopted by submissions targeting `2026-10-C1` through the three cohorts after it.
    Checkers before `v1.0.1.0` don't test adoption at all, so upgrade rather than rely on an older
    one.

## Steps

### 1. Run each point

How to probe your endpoint, write a config and run a benchmark is covered in the client's
[README](https://github.com/mlcommons/endpoints/blob/main/README.md#quick-start) and in the example
for your benchmark:

| Benchmark | Example |
|---|---|
| Llama 3.1 8B | [`05_Llama_Examples`](https://github.com/mlcommons/endpoints/blob/main/examples/05_Llama_Examples/README.md) |
| GPT-OSS 120B | [`04_GPTOSS120B_Example`](https://github.com/mlcommons/endpoints/blob/main/examples/04_GPTOSS120B_Example/Readme.md) |
| DeepSeek-R1 | [`07_DeepSeekR1_Example`](https://github.com/mlcommons/endpoints/blob/main/examples/07_DeepSeekR1_Example/README.md) |
| Agentic | [`10_Agentic_Inference`](https://github.com/mlcommons/endpoints/blob/main/examples/10_Agentic_Inference/README.md) |

Run every concurrency level in your plan with the `concurrency` load pattern at that level. Each run
writes its own run folder.

### 2. Run the Offline point {#run-the-offline-point}

Skip this if the benchmark is agentic, or if you're electing your `C_max` point as the Offline
result. An elected point is run exactly like the others in step 1; you only declare it differently
in [step 5](author-disclosures.md).

A dedicated Offline run gives the system the whole performance dataset at once instead of holding a
target concurrency ([§5.7.1][rules-5.7.1]). The reference client's pattern for that is
**`max_throughput`**, which issues every query at t=0.

!!! question "Confirm the pattern name"
    The rules call this "the benchmark-defined Offline load pattern" and don't name a client
    setting. `max_throughput` matches the definition, and the checker doesn't reject a dedicated
    Offline point for its load pattern, but no source says in so many words that this is the pattern
    MLCommons means. Tracked as **C7** in [Open questions](../help/open-questions.md).

!!! warning "Turn streaming on explicitly"
    The client's `streaming: auto` default resolves to **off** for offline runs. The rules still
    require `stream_all_chunks: true` for every performance run, Offline included, so set streaming
    on in the config rather than relying on the default.

Everything else in the constraints table still applies ([§5.7.3][rules-5.7.3]). What's different
about Offline, including the TTFT exemption and the limit on reordering, is in
[§5.7.1][rules-5.7.1]. In practice:

- **Size it for duration.** An Offline run ends when the queue drains, not on a clock, but the
  minimum run duration still applies. Issue enough passes over the dataset that it lasts at least
  1,200 seconds.
- **Don't let passes mix.** Reordering has to stay inside one pass over the dataset, and reviewers
  look for it: batches full of repeated copies of the same sample are the giveaway.
- **Check it against your `C_max` point** ([§5.7.2][rules-5.7.2]) as soon as both have run, while
  you can still re-run cheaply.

### 3. Run the accuracy validations

You need accuracy results at the four mandatory region points — Ultra Low, Low, Medium and High
Concurrency — and at the Offline point: five runs, or four for an agentic benchmark. If you elected
your `C_max` point, its accuracy run counts for both it and Offline.

How each run has to be done is in [§4.3 of the rules][rules-4.3], and it differs between single-turn
and multi-turn benchmarks. For a **single-turn** benchmark the constraint that catches people is
ordering: each accuracy run goes **immediately after** its point's performance run, so don't batch
them up at the end.

The simplest way to get the ordering right is `--mode both` on each of those points' configs, so one
invocation writes the performance run and then the accuracy run into the same report directory, with
`accuracy/accuracy_results.json` alongside `performance/result_summary.json`.

!!! danger "Accuracy has no tolerance"
    Throughput results get reproducibility margins. Accuracy doesn't ([Submission Rules
    §6.6][srules-reproducibility-expectations]). Miss the quality target and the submission is
    rejected.

### 4. Keep everything

Keep every run folder the client writes. What's in one is in [Submission package
layout](../reference/package-layout.md).


!!! warning "Warmup logs must be retained"
    Warmup requests don't count toward any metric, but their logs must be **kept for reviewers**
    ([§6.3.2][rules-6.3.2]).

## Next

→ [5. Author the disclosure files](author-disclosures.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#running-the-benchmark).
