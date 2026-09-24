# 4. Run the measurement points

> Produces: one run folder per Pareto point, with accuracy results at the required points.

!!! note "Before you begin"
    - Completed [3. Plan your Pareto curve](plan-your-curve.md)
    - You have a list of legal concurrency levels
    - The endpoint under test is up and reachable
    - You know which seed set you are binding to — see [Seeds](#seeds-and-salting) below

## What you'll do

- Probe the endpoint
- Write one YAML config per measurement point
- Run each point at **fixed concurrency** for its region's minimum duration
- Run the dedicated **Offline** point, if you chose one in step 3
- Run accuracy validations at the four mandatory region points and the Offline point
- Keep every run folder

This is the expensive step. Everything up to now took minutes. This takes days.

## The binding run constraints

The run requirements are in [§6 of the rules][rules-6], and that's where the values are. This
table only maps each one to where you meet it:

| Requirement | Rule | Where you set it |
|---|---|---|
| Load pattern | [§6.1][rules-6.1] | `settings.load_pattern.type`: `concurrency` for every point, `max_throughput` for a dedicated Offline run ([step 4](#4-run-the-offline-point)) |
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
    drain included, which understates what your system does. See
    [What your numbers are measured over](../reference/metrics-and-regions.md#what-your-numbers-are-measured-over).

!!! danger "Configuration consistency is checked across the whole curve"
    Every point must describe the same system, the same model and the same dataset. A curve
    assembled from points run against two different software versions will be flagged and can be
    rejected. Lock your software stack before the first point and don't change it until the last
    one is done.

## Steps

### 1. Probe the endpoint

```bash
uv run inference-endpoint probe \
  --endpoints http://your-endpoint:8000 \
  --model <your-model>
```

### 2. Generate and adapt a config template

```bash
uv run inference-endpoint init concurrency
```

A minimal concurrency config looks like this:

```yaml
name: point-c64
type: online
model_params:
  name: "<canonical model id>"
datasets:
  - name: perf
    type: performance
    path: "<performance dataset path>"
settings:
  runtime:
    scheduler_random_seed: <from your bound seed set>
    dataloader_random_seed: <from your bound seed set>
  load_pattern:
    type: concurrency        # the ONLY valid pattern for a Pareto point
    target_concurrency: 64
  timeouts:
    endpoint_response_idle_timeout_s: 300
endpoint_config:
  endpoints:
    - "http://your-endpoint:8000"
```

Validate the config before running it:

```bash
uv run inference-endpoint validate-yaml -c point-c64.yaml
```

### 3. Run each point

```bash
uv run inference-endpoint benchmark from-config --config point-c64.yaml
```

!!! tip "`from-config` has a narrow CLI surface"
    It accepts only `--config`, `--timeout` and `--mode`. There's no `--report-dir` override, so set
    `report_dir` in the YAML if you need to control where artifacts land.

Repeat for every concurrency level in your plan. Each writes its own run folder.

### 4. Run the Offline point

Skip this if the benchmark is agentic, or if you're electing your `C_max` point as the Offline
result. An elected point is run exactly like the others in step 3; you only declare it differently
in [step 5](author-disclosures.md).

A dedicated Offline run gives the system the whole performance dataset at once instead of holding a
target concurrency. The reference client's pattern for that is **`max_throughput`**, which issues
every query at t=0. It's what `benchmark offline` and `init offline` set up:

```bash
uv run inference-endpoint init offline
```

```yaml
settings:
  load_pattern:
    type: max_throughput     # every query issued at t=0
```

!!! question "Confirm the pattern name"
    The rules call this "the benchmark-defined Offline load pattern" and don't name a client
    setting. `max_throughput` matches the definition, and the checker doesn't reject a dedicated
    Offline point for its load pattern, but no source says in so many words that this is the
    pattern MLCommons means. Tracked as **C7** in [Open questions](../help/open-questions.md).

!!! warning "Turn streaming on explicitly"
    The client's `streaming: auto` default resolves to **off** for offline runs. The rules still
    require `stream_all_chunks: true` for every performance run, Offline included, so set streaming
    on in the config rather than relying on the default.

Everything else in the constraints table still applies: the same stack and seed set, a sample
count that's a whole multiple of the dataset size, and the warmup rules. Three things are specific
to Offline:

- **Size it for duration.** An Offline run ends when the queue drains, not on a clock, but the
  minimum run duration still applies. Issue enough passes over the dataset that it lasts at least
  1,200 seconds.
- **Don't let passes mix.** The system can reorder and batch within one pass over the dataset as
  it likes. It must not batch or reorder a query from one pass against a query from another.
  Reviewers look at this specifically: batches full of repeated copies of the same sample are the
  giveaway.
- **Check it against your `C_max` point.** Its `system_tps` has to be at least 98% of your
  `C_max` point's. Compare the two as soon as both have run, while you can still re-run cheaply.

TTFT is still recorded, but it isn't required for this point and can't be used against it.

### 5. Run the accuracy validations

You need accuracy results at the four mandatory region points — Ultra Low, Low, Medium and High
Concurrency — and at the Offline point: five runs, or four for an agentic benchmark. If you
elected your `C_max` point, its accuracy run counts for both it and Offline. All of them use the
**same** endpoint configuration, model weights and software stack as the performance runs.

For a **single-turn** benchmark, each accuracy run goes at the same concurrency as its point, on
the same instance, **immediately after** that point's performance run. Don't batch them up at the
end: the rule ties each accuracy run to the performance run it follows. Every result has to pass.

For a **multi-turn** benchmark, only the average of the results has to pass, and the concurrency
can differ — these runs are expensive, so the rules give you room here.

The simplest way to get the ordering right is `--mode both` on each of those points' configs, so
one invocation writes the performance run and then the accuracy run into the same report
directory, with `accuracy/accuracy_results.json` alongside `performance/result_summary.json`.

!!! danger "Accuracy has no tolerance"
    Throughput results get reproducibility margins. Accuracy doesn't. It's a hard gate at automated
    compliance and throughout review. Miss the quality target and the submission is rejected.

### 6. Keep everything

Each run folder contains:

```
<report_dir>/
├── config.yaml                       # resolved config as run (secrets redacted)
├── report.txt                        # human-readable summary
├── events.jsonl                      # per-event log (the large one)
├── sample_idx_map.json
├── performance/result_summary.json   # written when the performance phase ran
├── accuracy/accuracy_results.json    # written when the accuracy phase ran
└── metrics/final_snapshot.json
```

Full detail in [Submission package layout](../reference/package-layout.md).

!!! warning "Warmup logs must be retained"
    All requests issued before `TEST_STARTED` are warmup and must not appear in any reported
    metric, but their logs must be **retained and available for reviewer inspection**. Reviewers
    may cross-check them against the performance dataset.

## Seeds and salting

Your submission binds to exactly **one seed set**, chosen when it first appears, and every point
must record that same set. The seeds drive the client's request-issue / sample-order RNG and the
per-query salt. Once bound, the set stays valid for that submission even after newer sets are
published. Seed rotation never forces an in-flight submission to be re-run.

The **salt** is a per-query unique value inserted between the shared system prompt and the per-query
user context. It is what makes blanket cross-query KV-cache reuse legal: the only prefix two queries
can share is the system prompt itself. Accuracy runs use the **un-salted** dataset so that model
output matches the canonical implementation exactly.

!!! warning "Warmup must not use performance-dataset samples"
    Warmup requests must not use any sample from the benchmark performance dataset — directly, as a
    subset, truncated, or derived. The accuracy dataset and other sources are fine. If your client
    does use the performance dataset during warmup, **salting must be enabled**, and note that the
    salting flag is **not on by default**.

!!! success "The v1.0 seed set is published"
    As of 2026-09-15 the policies repo carries `seedset.yaml`: one set, **`id: A`**, published for
    cohort **`2026-10-C1`**. Since checker `v1.0.1.0` the checker bundles the same file, cohort key
    included, and `seed-set-adoption` tests your `target_cohort` against the four-cohort window
    for real.

    ```yaml
    seed_set: A
    target_cohort: 2026-10-C1
    ```

    Set `A` can be adopted by submissions targeting `2026-10-C1` through the three cohorts after it.
    On an older checker, adoption reports **SKIP**. Upgrade rather than override.

## Verify

For each run folder, confirm the phase directory you expected exists and the run completed:

```bash
ls <report_dir>/performance/result_summary.json
python -c "import json;d=json.load(open('<report_dir>/performance/result_summary.json'));print(d['complete'], d['n_samples_completed'], d['duration_ns']/1e9)"
```

Check three things:

- `complete` is `true` — a `false` means the run drained out or was interrupted, and the point is
  not usable
- `duration_ns` meets your region's minimum in seconds
- `n_samples_completed` represents at least one dataset pass

Also confirm the run used the right pattern. `run_config` in `result_summary.json` should show the
concurrency load pattern at your target level, or `max_throughput` for a dedicated Offline run.

For a dedicated Offline run, check it clears your `C_max` point before you tear anything down. This
computes `system_tps` the way the checker does, as output tokens over run seconds:

```bash
python - <<'EOF'
import json
def tps(d):
    s = json.load(open(f"{d}/performance/result_summary.json"))
    return s["output_sequence_lengths"]["total"] / (s["duration_ns"] / 1e9)
off, cmax = tps("<offline_dir>"), tps("<c_max_dir>")
print(f"offline {off:.1f}  c_max {cmax:.1f}  ratio {off / cmax:.3f}  (needs >= 0.98)")
EOF
```

## Next

→ [5. Author the disclosure files](author-disclosures.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#running-the-benchmark).
