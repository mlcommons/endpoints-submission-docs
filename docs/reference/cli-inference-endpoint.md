# Benchmark runner CLI — `inference-endpoint`

The reference client from [`mlcommons/endpoints`](https://github.com/mlcommons/endpoints). Drives
load at your endpoint and writes a run folder. Used in [step 4](../workflow/run-the-points.md).

Commands, flags, the YAML config structure, dataset formats and test modes are documented upstream:

- [CLI quick reference](https://github.com/mlcommons/endpoints/blob/main/docs/CLI_QUICK_REFERENCE.md)
- [README](https://github.com/mlcommons/endpoints/blob/main/README.md#quick-start)
- [Examples](https://github.com/mlcommons/endpoints/tree/main/examples)

Run `inference-endpoint --help` for the full generated flag list.

## What a submission run needs

Upstream describes what the client can do. These are the settings a submission is held to:

| Setting | For a submission |
|---|---|
| `load_pattern.type` | `concurrency` for every fixed-concurrency point of a single-turn benchmark, `agentic_inference` for agentic benchmarks. `max_throughput` only for a dedicated Offline run. `poisson` is never valid |
| `streaming` | On for every performance run, the Offline point included. The default, `auto`, resolves to off for offline runs |
| `scheduler_random_seed`, `dataloader_random_seed` | From your bound seed set. See [Seeds and salting](../workflow/run-the-points.md#seeds-and-salting) |

!!! warning "Do not modify the source"
    Configure the client through its YAML config only ([§2.1.1 of the rules][rules-2.1.1]).

!!! warning "The warmup salt is off by default"
    [§6.3.1][rules-6.3.1] prohibits performance-dataset samples in warmup. If the client does use
    them, salting must be enabled, and the rules' v0.7 note also requires KV-cache reuse to be off.
    The client does not enable the salt for you (`--warmup-salt`).

!!! question "`max_throughput` for the Offline point"
    The rules call the Offline pattern "the benchmark-defined Offline load pattern" without naming a
    client setting. `max_throughput` matches the description, but no source confirms it's the
    intended pattern. Tracked as **C7** in [Open questions](../help/open-questions.md).

*Last verified against: `mlcommons/endpoints@main` (e71b928) and
`mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
