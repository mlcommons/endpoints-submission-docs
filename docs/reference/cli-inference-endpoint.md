# Benchmark runner CLI — `inference-endpoint`

The reference client from [`mlcommons/endpoints`](https://github.com/mlcommons/endpoints). Drives
load at your endpoint and writes a run folder. Used in [step 4](../workflow/run-the-points.md).

!!! note "Tracks upstream"
    This page tracks `docs/CLI_QUICK_REFERENCE.md` in `mlcommons/endpoints`. The upstream document
    is authoritative; run `--help` for the full generated flag list.

!!! danger "For CoP submissions, do not modify the source"
    The client must be used without source-code modification, built from a commit accessible to the
    review committee. Everything that changes behaviour must be expressible in the YAML config.

Requires **Python 3.12+**. Commands below assume an activated venv; without one, prefix with
`uv run`.

## Commands

| Command | Does |
|---|---|
| `benchmark offline` | Max-throughput burst — the dedicated **Offline** point only, never a fixed-concurrency point |
| `benchmark online` | Sustained load with a load pattern |
| `benchmark from-config` | Run from a YAML config |
| `probe` | Test endpoint connectivity |
| `validate-yaml` | Validate a YAML config without running |
| `init` | Generate a config template |
| `info` | Show system info |
| `eval` | Accuracy evaluation — **not yet implemented** |

### For a submission

```bash
inference-endpoint init concurrency          # or: offline, online, eval, submission
inference-endpoint validate-yaml -c point.yaml
inference-endpoint benchmark from-config --config point.yaml
```

!!! warning "`from-config` accepts only three flags"
    `--config`, `--timeout` and `--mode`. There is **no** `--report-dir` override — set `report_dir`
    in the YAML if you need to control the output location.

## Load patterns

| Pattern | Behaviour | Valid for a Pareto point? |
|---|---|---|
| `concurrency` | Maintains N concurrent requests; QPS emerges from concurrency/latency | **Yes — every fixed-concurrency point** |
| `max_throughput` | All queries issued at t=0 | **Only** a dedicated Offline run — see below |
| `poisson` | Fixed QPS with Poisson arrivals | No |
| `agentic_inference` | Multi-turn conversations with turn sequencing | Agentic benchmarks. Not in the CLI quick reference; see `docs/load_generator/DESIGN.md` upstream |

!!! question "`max_throughput` for the Offline point"
    The rules define the Offline point as "all queries available to the system at once" and call
    its pattern "the benchmark-defined Offline load pattern", without naming a client setting.
    `max_throughput` matches that description, but no source confirms it's the intended pattern.
    Tracked as **C7** in [Open questions](../help/open-questions.md).

```yaml
settings:
  load_pattern:
    type: concurrency
    target_concurrency: 64
```

## Test modes

| Mode | Behaviour |
|---|---|
| `perf` (default) | Performance only. No response storage. Metrics: QPS, latency, TTFT, TPOT |
| `acc` | Accuracy only. Collects and evaluates responses. Requires `accuracy_config` on datasets |
| `both` | Combined — performance datasets give metrics, accuracy datasets are collected and evaluated |

Use `--mode both` for a combined run that writes both `performance/` and `accuracy/`.

## Common options

Flags exist as `--full.dotted.path` and, where defined, a short alias. Both forms work.

**Required for CLI-mode benchmarks:**

| Flag | Alias |
|---|---|
| `--endpoint-config.endpoints` | `--endpoints` |
| `--model-params.name` | `--model` |
| `--dataset` | — |

**Frequently used:**

| Flag | Alias | Default | Notes |
|---|---|---|---|
| `--model-params.max-new-tokens` | `--max-output-tokens` | 1024 | |
| `--model-params.osl-distribution.min` | `--min-output-tokens` | 1 | |
| `--model-params.streaming` | `--streaming` | `auto` | `auto` resolves to off for offline, on for online. Submission runs, the Offline point included, need streaming on |
| `--runtime.n-samples-to-issue` | `--num-samples` | — | Explicit sample count |
| `--runtime.min-issue-duration-ms` | — | — | Poisson sizing from QPS × duration |
| `--runtime.max-issue-duration-ms` | — | — | Caps performance issuing; in-flight responses still drain |
| `--client.num-workers` | `--workers` | `-1` (auto) | HTTP workers |
| `--client.max-connections` | `--max-connections` | `-1` | Max TCP connections |
| `--endpoint-config.api-key` | `--api-key` | — | |
| `--endpoint-config.api-type` | `--api-type` | `openai` | `openai` or `sglang` |
| `--report-dir` | — | — | CLI-mode only, not `from-config` |
| `--timeout` | — | off | Whole-run watchdog |
| `--no-early-stopping` | — | on | Opt out of early-stopping percentile estimates |
| `--load-pattern.target-concurrency` | `--concurrency` | — | Required for the concurrency pattern |

Every other schema field is reachable via its dotted path.

## Endpoint liveness

Fail the run when in-flight work stops responding. Use **≥ 300 seconds**; raise it for long
requests.

```yaml
settings:
  timeouts:
    endpoint_response_idle_timeout_s: 300
```

## YAML structure

```yaml
name: "point-c64"
type: "online"               # offline | online | eval | submission

model_params:
  name: "<model id>"
  temperature: 0.7
  max_new_tokens: 2048

datasets:
  - name: "perf"
    type: "performance"      # performance | accuracy
    path: "openorca.jsonl"
  - name: "gpqa"
    type: "accuracy"
    path: "gpqa.jsonl"
    eval_method: "exact_match"

settings:
  runtime:
    min_issue_duration_ms: null
    max_issue_duration_ms: null
    n_samples_to_issue: null
    scheduler_random_seed: 42      # from your bound seed set
    dataloader_random_seed: 42     # from your bound seed set
  timeouts:
    run_timeout_s: null
    endpoint_response_idle_timeout_s: 300
  load_pattern:
    type: "concurrency"
    target_concurrency: 64
  client:
    num_workers: -1

endpoint_config:
  endpoints:
    - "http://localhost:8000"
  api_key: null
  api_type: "openai"
```

Environment variables interpolate as `${VAR}` or `${VAR:-default}`.

!!! warning "Submission-type configs are YAML-only"
    `type: submission` requires `submission_ref` and `benchmark_mode`, which are not exposed on the
    CLI.

## Seeds and salting

| Setting | Purpose |
|---|---|
| `settings.runtime.scheduler_random_seed` | Scheduler / request-issue RNG |
| `settings.runtime.dataloader_random_seed` | Dataset ordering RNG |
| Warmup salt (`--warmup-salt`) | Prepends a unique random hex salt to each warmup prompt |

!!! danger "The warmup salt is off by default"
    If warmup uses the performance dataset, salting **must** be enabled, and you must also disable
    KV cache reuse for warmup. It is not enabled automatically.

Salt requires a dict sample with a text `prompt` field. A dataset whose samples carry `messages`, or
multimodal content parts, cannot be salted — the client validates every sample before issuing and
fails rather than shipping an unsalted payload.

## Datasets

Format auto-detected from the extension; override with `format=<ext>`.

**Supported:** `.csv`, `.json`, `.jsonl`, `.parquet`, `huggingface`

```bash
--dataset data.jsonl                                      # simple path
--dataset acc:eval.jsonl                                  # accuracy dataset
--dataset data.csv,samples=500,parser.prompt=article      # with options
--dataset perf:data.jsonl,format=.jsonl,parser.prompt=text
```

The `perf:` / `acc:` prefix is optional and defaults to `perf`. `--dataset` is repeatable.

## Local testing

```bash
python -m inference_endpoint.testing.echo_server --port 8765 &
inference-endpoint benchmark offline \
  --endpoints http://localhost:8765 \
  --model test-model \
  --dataset tests/assets/datasets/dummy_1k.jsonl
pkill -f echo_server
```

## Output

See [Submission package layout](package-layout.md) for the run folder the client writes.

!!! warning "The written `config.yaml` is sanitized"
    Credentials and other secrets are replaced with `<redacted>`. Restore them before reusing that
    file as benchmark input.

*Last verified against: `mlcommons/endpoints@main` (e71b928) and
`mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
