# 2. Install the tools

> Produces: a working `inference-endpoint`, `endpoints-submission-cli` and `submission-checker`.

!!! note "Before you begin"
    - Completed [1. Register and get a token](register.md)
    - Python **3.12+** available for the reference client
    - Python 3.10+ available for the submission CLI

## What you'll do

- Install the **reference client** (`mlcommons/endpoints`) — runs the benchmark
- Install the **submission tools** (`mlcommons/endpoints-submission-cli`) — registers runs, assembles
  and validates the bundle
- Install `gh` and confirm all three respond

There are three distinct CLIs and it is worth fixing which is which now:

| CLI | Package | Does |
|---|---|---|
| `inference-endpoint` | `mlcommons/endpoints` | Drives load at your endpoint and writes a run folder |
| `endpoints-submission-cli` | `endpoints-submission-cli` | Registers runs, assembles and uploads a submission |
| `submission-checker` | same package | Validates a bundle against the automated compliance rules |

## Steps

### 1. Install the reference client

!!! danger "Do not modify the source"
    For Client-on-Prem submissions the reference client must be used **without source-code
    modification**, built from a commit accessible to the review committee. Everything that changes
    behaviour must be expressible in the YAML config. The client logs its commit SHA, and review may
    run a seeded-RNG check against your bound seed set to detect undisclosed modifications.

=== "uv (recommended)"

    ```bash
    git clone https://github.com/mlcommons/endpoints.git
    cd endpoints
    uv sync
    ```

    Commands then run as `uv run inference-endpoint …`.

=== "pip + venv"

    ```bash
    git clone https://github.com/mlcommons/endpoints.git
    cd endpoints
    python3.12 -m venv venv && source venv/bin/activate
    pip install .
    ```

    Note this does **not** use `uv.lock`, so dependency versions may differ from the lockfile.
    After activating the venv, commands run without the `uv run` prefix.

Record the commit SHA you built from — you will need it for disclosure:

```bash
git -C endpoints rev-parse HEAD
```

### 2. Install the submission tools

=== "pip"

    ```bash
    pip install endpoints-submission-cli
    ```

=== "From source"

    ```bash
    git clone https://github.com/mlcommons/endpoints-submission-cli.git
    cd endpoints-submission-cli
    pip install -e ".[dev]"
    ```

=== "uv"

    ```bash
    uv sync --extra dev
    ```

Both `endpoints-submission-cli` and `submission-checker` come from this one package.

Optionally, install the results visualizer to preview your curves locally before you submit:

```bash
pip install mlperf-viz
```

See [Results visualizer](../reference/cli-visualizer.md).

### 3. Install `gh`

The [`gh` CLI](https://cli.github.com/) is required for creating, updating and withdrawing
submissions. Install it and authenticate with `gh auth login`.

### 4. Smoke-test the client against a local server

Before pointing anything at real hardware, confirm the client works end to end against the bundled
echo server:

```bash
uv run python -m inference_endpoint.testing.echo_server --port 8765 &
uv run inference-endpoint benchmark offline \
  --endpoints http://localhost:8765 \
  --model test-model \
  --dataset tests/assets/datasets/dummy_1k.jsonl
pkill -f echo_server
```

## Verify

All three must respond:

```bash
uv run inference-endpoint --version
endpoints-submission-cli --version
submission-checker --help
```

And the region calculator should produce sensible boundaries — this is the tool you will use in the
next step:

```bash
submission-checker regions --max-concurrency 1024 --min-concurrency 16
```

## Next

→ [3. Plan your Pareto curve](plan-your-curve.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#installation).
