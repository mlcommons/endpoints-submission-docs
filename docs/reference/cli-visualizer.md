# Results visualizer — `mlperf-viz`

Renders a submission folder as a local browser dashboard — the Pareto Explorer — so you can look at
your curves before submitting, or browse published results. It reads the same layout the
[submission
checker](https://github.com/mlcommons/endpoints-submission-cli/blob/main/README.md#submission-checker)
expects. It does **not** validate anything: use the checker for that.

Ships as its own package, [`mlperf-viz`](https://pypi.org/project/mlperf-viz/), from
[`mlcommons/mlperf-viz`](https://github.com/mlcommons/mlperf-viz). Nothing leaves your machine — no
account, no token, no upload.

## Install

```bash
pip install mlperf-viz
mlperf-viz -V
```

## Usage

```bash
mlperf-viz [PATH] [options]
```

`PATH` is the submission tree root and defaults to the current directory. It may be a whole results
repository, a single organisation's directory, or one submission.

| Flag | Description |
|---|---|
| `--version {1.0,0.7}` | Submission **format** to read. Default `1.0` |
| `--host HOST` | Interface to bind. Default `127.0.0.1` |
| `--port PORT` | Preferred port; the next free one is used if taken. Default `8000` |
| `--no-browser` | Do not open a browser window |
| `--export FILE` | Write the dashboard dataset as JSON and exit |
| `--build DIR` | Write a self-contained copy of the site plus data to `DIR` and exit |
| `--strict` | Exit non-zero if any structural warning was raised |
| `-V`, `--cli-version` | Print the tool's own version and exit |

!!! note "`--version` is the submission format, not the tool version"
    `--version 0.7` reads the v0.7 layout. `-V` prints which `mlperf-viz` you have installed.

**Exit codes:** `0` on success · non-zero under `--strict` when a structural warning was raised.
Without `--strict`, missing or unreadable files are reported as warnings and skipped, so a partial
submission still renders.

## Formats

| | v1.0 (default) | v0.7 (`--version 0.7`) |
|---|---|---|
| Anchored on | `results/<System>/<Model>/r<N>/` point directories | `systems/*.json` |
| System description | `system_desc.json`, one per point | `systems/<id>.json`, one per model directory |
| Point config | `point.yaml`, inside the point | `points/point_<N>.yaml` |
| Metrics | `result_summary.json` | `run_metadata.json` |
| Accuracy | `accuracy_results.json` | on the system description |
| Dashboard entry | `<Org>-<submission_id>-<System>-<Model>` | `<Org>-<System>-<Model>` |

One `<System>/<Model>` pair is one Pareto curve and one dashboard entry, so a submission covering
two models shows as two entries. The v1.0 layout itself is described in
[Submission package layout](package-layout.md).

For v1.0, `system_tps`, `tps_per_user` and `qps` are derived from the raw stat blocks when the
summary does not already report them:

```text
system_tps   = output_sequence_lengths.total / (duration_ns / 1e9)
tps_per_user = 1000 / tpot_p90_ms
qps          = n_samples_completed / (duration_ns / 1e9)
```

v0.7's `run_metadata.json` already carries all three.

## Example: your own submission (v1.0)

Point it at the same folder you pass to `submission-checker check`:

```bash
mlperf-viz /path/to/submission
```

It prints how many submissions and runs it found, starts a local server and opens the Pareto
Explorer. Add `--no-browser` on a remote machine and forward the port.

## Example: v0.7 published results

The v0.7 results are published in the public repository
[`mlcommons/endpoints_results_v0.7`](https://github.com/mlcommons/endpoints_results_v0.7), in the
published-results variant of the v0.7 layout (`<Org>/<System>/<Model>/`).

```bash
git clone https://github.com/mlcommons/endpoints_results_v0.7.git
mlperf-viz endpoints_results_v0.7 --version 0.7
```

```text
Found 10 submission(s) and 100 run(s) in v0.7 format.
```

To look at one organisation, point at its directory:

```bash
mlperf-viz endpoints_results_v0.7/Krai --version 0.7
```

To share a snapshot without anyone installing anything, build a static copy and host the directory
anywhere that serves files:

```bash
mlperf-viz endpoints_results_v0.7 --version 0.7 --build v0.7-site
```

To use the data programmatically instead, `--export` writes the dashboard dataset — top-level
`submissions` and `runs` arrays — as JSON:

```bash
mlperf-viz endpoints_results_v0.7 --version 0.7 --export v0.7.json
```

??? note "Caveats"
    - **The format is chosen by the flag, not detected.** Reading a v0.7 tree without
      `--version 0.7` finds no submissions; the tool prints the layout it expected and names the
      flag.
    - **v0.7 has a second layout** — the submission-checker variant,
      `<Org>/systems/<system_desc_id>.json` with `pareto/<system_desc_id>/<model>/{points,results}/`
      — which `--version 0.7` also reads.
    - **Entries are identified by path**, not by the `submission_id` field, which is frequently
      absent, empty, or shared between distinct submissions.
    - **One accelerator per node.** A v1.0 node type that discloses more than one accelerator
      configuration is shown with the first, and a warning says how many were omitted.
    - **Fonts and analytics** load from Google when online; offline the dashboard falls back to
      system fonts.

*Last verified against: `mlcommons/mlperf-viz@cli-v1.1.0` (654dca5) with `mlperf-viz 1.1.0` from
PyPI, and `mlcommons/endpoints_results_v0.7@main`, 2026-10-02.*
