# 2. Install the tools

> Produces: a working `inference-endpoint` and `endpoints-submission-cli`, including its checker.

!!! note "Before you begin"
    - Completed [1. Get access and an API key](register.md)
    - Python **3.12+** available for the reference client
    - Python 3.10+ available for the submission CLI

## What you'll do

- Install the **reference client** ([`mlcommons/endpoints`](https://github.com/mlcommons/endpoints)) — runs the benchmark
- Install the **submission tools** ([`mlcommons/endpoints-submission-cli`](https://github.com/mlcommons/endpoints-submission-cli)) — registers runs, assembles
  and validates the bundle
- Check the client and submission tools

Use these commands:

| CLI | Package | Does |
|---|---|---|
| `inference-endpoint` | [`mlcommons/endpoints`](https://github.com/mlcommons/endpoints) | Drives load at your endpoint and writes a run folder |
| `endpoints-submission-cli` | [`endpoints-submission-cli`](https://github.com/mlcommons/endpoints-submission-cli) | Registers runs, assembles and uploads a submission |
| `endpoints-submission-cli check-submission` | same package | Validates a bundle against the automated compliance rules |

## Steps

### 1. Install the reference client

!!! danger "Do not modify the source"
    For Client-on-Prem submissions the reference client must be used **without source-code
    modification**, built from a commit the review committee can access. Anything you want to change
    goes in the YAML config. Review can detect modifications ([§2.1.1 of the rules][rules-2.1.1]).

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

    Note this does **not** use `uv.lock`, so dependency versions may differ from the lockfile. After
    activating the venv, commands run without the `uv run` prefix.

Record the commit SHA you built from — you will need it for disclosure:

```bash
git rev-parse HEAD
```

### 2. Install the submission tools

=== "pip"

    ```bash
    pip install endpoints-submission-cli
    ```

=== "From source"

    ```bash
    git clone https://github.com/mlcommons/endpoints-submission-cli.git ../endpoints-submission-cli
    pip install -e "../endpoints-submission-cli[dev]"
    ```

=== "uv"

    ```bash
    uv tool install endpoints-submission-cli
    ```

Use `endpoints-submission-cli check-submission` to validate a bundle.

Optionally, install the results visualizer to preview your frontiers locally before you submit:

```bash
pip install mlperf-viz
```

See [Results visualizer](../reference/cli-visualizer.md).

### 3. Smoke-test the client against a local server

In the `endpoints` checkout, start the echo server:

```bash
uv run python -m inference_endpoint.testing.echo_server --host 127.0.0.1 --port 8765
```

In a second terminal, open the same checkout and run:

```bash
uv run inference-endpoint probe --endpoints http://127.0.0.1:8765 --model test-model --requests 2
```

Expect two successful responses. Stop the server with Ctrl+C.
For the pip setup, use the active environment and omit `uv run`.

## Verify

Both must respond:

```bash
uv run inference-endpoint --version
endpoints-submission-cli --version
```

And the region calculator should produce sensible boundaries — this is the tool you will use in the
next step:

```bash
uv tool run --from endpoints-submission-cli python -c "from submission_checker.cli import main; main()" regions --max-concurrency 1024 --min-concurrency 16
```

## Next

→ [3. Plan your Pareto frontier](plan-your-curve.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#installation).
