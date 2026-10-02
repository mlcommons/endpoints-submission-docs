<!--
  PROVENANCE SNAPSHOT — do not edit.
  Upstream : mlperf-viz/cli/README.md
  Repo     : mlcommons/mlperf-viz @ cli-v1.1.0@654dca5
  Captured : 2026-10-02
  Purpose  : diff this against upstream to find what drifted since the docs were written.
-->

# mlperf-viz

Visualize an MLPerf Endpoints submission folder — the same layout the
[submission checker](https://github.com/mlcommons/endpoints-submission-cli) expects —
in a local browser dashboard.

```sh
pip install mlperf-viz
mlperf-viz /path/to/endpoints_results
```

That reads the submission tree, converts it to the dashboard's dataset, starts a
local server, and opens the Pareto Explorer. No Node, no Bun, no account, and
nothing leaves your machine.

## Usage

```
mlperf-viz [PATH]

  PATH                 submission tree root (default: current directory)
  --version {1.0,0.7}  submission format to read (default: 1.0)
  --host HOST          interface to bind (default: 127.0.0.1)
  --port PORT          preferred port; the next free one is used if taken (default: 8000)
  --no-browser         do not open a browser window
  --export FILE        write the bundle JSON and exit
  --build DIR          write a self-contained copy of the site plus data to DIR and exit
  --strict             exit non-zero if any structural warning was raised
  -V, --cli-version    print this tool's version and exit
```

Note that `--version` selects the **submission format**, not this tool's version;
`-V` prints the tool's version.

## Expected layout

### v1.0 (default)

The layout since `endpoints_policies` PR #119, anchored on a `results/` tree whose
leaves are per-concurrency point directories:

```
<Org>/<submission_id>/
    src/<implementation>/README.md
    docs/
    results/<System>/<Model>/r<N>/
        point.yaml
        result_summary.json
        accuracy_results.json
        system_desc.json
        config.yaml          (optional)
```

One `<System>/<Model>` pair is one Pareto curve, and one entry in the dashboard —
§8.5 defines a result as "one system, one benchmark model, one dataset". A
submission holding two models therefore shows as two entries.

`system_desc.json` sits in every point directory; §8.2 requires the points of a
curve to agree on all of it but `tps_utilization`, so the first point describes the
system and each point supplies its own utilisation.

### v0.7 (`--version 0.7`)

The pre-PR#119 layout. Both variants are anchored on a `systems/*.json` file:

```
# Published results repository
<Org>/<System>/<Model>/
    systems/<System>.json
    points/point_<N>.yaml
    results/point_<N>/{run_metadata.json, results_summary.json, ...}

# Submission checker layout
<Org>/
    systems/<system_desc_id>.json
    pareto/<system_desc_id>/<model>/{points,results}/...
```

```sh
mlperf-viz ./endpoints_results_0.7 --version 0.7
```

`PATH` may be a whole results repository, a single organization directory, or one
submission.

### What changed between them

|                          | v0.7                                   | v1.0                                        |
| ------------------------ | -------------------------------------- | ------------------------------------------- |
| Point directory          | `results/point_<N>/`                   | `results/<System>/<Model>/r<N>/`            |
| System description       | `systems/<id>.json`, one per model dir | `system_desc.json`, one per point           |
| Point config             | `points/point_<N>.yaml`                | `point.yaml`, inside the point              |
| Metrics                  | `run_metadata.json`                    | `result_summary.json`                       |
| Accelerators             | flat on `node_types[]`                 | nested in `node_types[].accelerator_info[]` |
| Model / dataset metadata | on the system description              | on `point.yaml`                             |
| Accuracy                 | on the system description              | `accuracy_results.json`                     |

v0.7's `run_metadata.json` shipped `system_tps`, `tps_per_user` and `qps` already
computed. v1.0 removed that file, so the visualizer derives them from the raw stat
blocks using §9.1's definitions:

```
system_tps   = output_sequence_lengths.total / (duration_ns / 1e9)
tps_per_user = 1000 / tpot_p90_ms
qps          = n_samples_completed / (duration_ns / 1e9)
```

A v1.0 summary that still reports `system_tps` or `tps_per_user` is taken at its
word. Timing blocks are nanoseconds, and percentile keys are read under both the
`"90"` and `"90.0"` spellings, because endpoints and the checker's corpus disagree.

Missing or unreadable files produce warnings and are skipped, so a partial
submission still renders. Use `--strict` in CI to turn those warnings into a
non-zero exit. This tool does **not** validate a submission — use the submission
checker for that.

## Notes

- Submissions are identified by their path — `<Org>-<System>-<Model>` in v0.7,
  `<Org>-<submission_id>-<System>-<Model>` in v1.0 — not by the `submission_id`
  field, which is frequently absent, empty, or shared between distinct submissions.
- The format is chosen by the flag rather than sniffed from the tree: a directory
  that half-matches both layouts is a broken submission, and guessing would hide
  that. Reading a tree in the wrong format reports no submissions found, and names
  the flag.
- A v1.0 node type that discloses more than one accelerator configuration is shown
  with the first; the dashboard displays one accelerator per node, and a warning
  says how many were omitted.
- The dashboard loads web fonts and analytics from Google when online; offline it
  falls back to system fonts and those requests simply fail.

## Development

The package embeds a prebuilt copy of the [mlperf-viz](https://github.com/mlcommons/mlperf-viz)
dashboard. From a source checkout, build it first:

```sh
bash cli/scripts/build_webapp.sh   # requires Bun
cd cli && pip install -e '.[dev]' && pytest
```

## License

Apache-2.0
