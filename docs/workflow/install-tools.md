# 2. Install the tools

> Produces: a working `inference-endpoint`, `endpoints-submission-cli` and `submission-checker`.

!!! note "Before you begin"
    - Completed [1. Register and get a token](register.md)
    - Python **3.12+** available for the reference client
    - Python 3.10+ available for the submission CLI

## What you'll do

- Install the **reference client** ([`mlcommons/endpoints`](https://github.com/mlcommons/endpoints)) — runs the benchmark
- Install the **submission tools** ([`mlcommons/endpoints-submission-cli`](https://github.com/mlcommons/endpoints-submission-cli)) — registers runs, assembles
  and validates the bundle
- Confirm all three CLIs respond

There are three distinct CLIs and it is worth fixing which is which now:

| CLI | Package | Does |
|---|---|---|
| `inference-endpoint` | [`mlcommons/endpoints`](https://github.com/mlcommons/endpoints) | Drives load at your endpoint and writes a run folder |
| `endpoints-submission-cli` | [`endpoints-submission-cli`](https://github.com/mlcommons/endpoints-submission-cli) | Registers runs, assembles and uploads a submission |
| `submission-checker` | same package | Validates a bundle against the automated compliance rules |

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
git -C endpoints rev-parse HEAD
```

### 2. Install the submission tools

```bash
pip install endpoints-submission-cli
```

Both `endpoints-submission-cli` and `submission-checker` come from this one package.

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
