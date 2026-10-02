# Submission package layout

Two layouts matter: the **run folder** the reference client writes, and the **submission bundle**
the CLI assembles from run folders. Authored in [step 5](../workflow/author-disclosures.md).

## The run folder

What `mlcommons/endpoints` writes to `report_dir` for one benchmark run.

```
<report_dir>/
├── config.yaml                       # resolved config as run          [always]
├── report.txt                        # human-readable summary          [always]
├── events.jsonl                      # one JSON object per event       [always]
├── sample_idx_map.json               # sample index → dataset row      [always]
├── performance/
│   └── result_summary.json           # performance metrics             [performance phase]
├── accuracy/
│   └── accuracy_results.json         # per-dataset accuracy scores     [accuracy phase]
└── metrics/
    ├── final_snapshot.json           # aggregator end-of-run snapshot  [always]
    └── .ready                        # zero-byte completion sentinel   [always]
```

Phase directories exist only for phases that ran:

| Invocation | `performance/` | `accuracy/` |
|---|---|---|
| `--mode perf` (default) | yes | no |
| `--accuracy-only` | no | yes |
| `--mode both` | yes | yes |

### Plus what you author

!!! danger "Three required files are not written by the reference client"
    `runs create` requires **`system_desc.json`** and **`point.yaml`** in every run folder, and the
    bundle needs a **`system_power.json`** from at least one run folder per system. None is an
    endpoints artifact. You author `point.yaml` and `system_power.json`; `system_desc.json` you can
    capture with [`mlperf-sysinfo`](https://docs.mlcommons.org/mlperf-sysinfo/) or write from the template. Drop them in before upload.

```
<run-folder>/
├── system_desc.json        # you supply — hardware/software description (mlperf-sysinfo)
├── point.yaml              # you author — per-point disclosure
├── system_power.json       # you author — per system; any run folder of that system
├── src/<implementation>/   # merged into the bundle's shared src/ (README.md required)
└── documentation/          # merged into the bundle's shared docs/
```

!!! warning "This is the only accepted layout"
    A flat folder with `result_summary.json` at the top level is rejected: *"Run folder error: … is
    missing required file(s): performance/result_summary.json"*.

### Sizes

From a 60-second, 16-concurrency run issuing 1,200 samples:

| File | Size | Scales with |
|---|---|---|
| `events.jsonl` | **46 MB** | samples × events per sample |
| `sample_idx_map.json` | 100 KB | samples issued |
| `metrics/final_snapshot.json` | 28 KB | fixed |
| `performance/result_summary.json` | 20 KB | fixed (histogram buckets) |
| `accuracy/accuracy_results.json` | 8 KB | datasets scored |
| `config.yaml`, `report.txt` | 4–8 KB | fixed |

`events.jsonl` dominates. A 600-second Pareto point produces several hundred megabytes, so run
archives are large — plan disk and upload bandwidth accordingly.

### Key fields in `result_summary.json`

```
version, git_sha, test_started_at, n_samples_issued, n_samples_completed,
n_samples_failed, n_samples_succeeded, n_samples_dropped, duration_ns, state,
complete, ttft, tpot, latency, output_sequence_lengths, input_sequence_lengths,
legacy_loadgen_window_duration_ns, qps, tps, finish_reason_counts, run_config
```

`ttft`, `tpot`, `latency` and the sequence-length entries are stat blocks of `{total, min, max,
median, avg, std_dev, percentiles, histogram}`.

!!! warning "Percentile keys are decimal strings"
    `"50.0"`, `"90.0"`, `"99.9"` — **not** `"50"` / `"90"`. A lookup by integer string returns
    nothing.

### `accuracy_results.json`

```json
{"osl_tokenization_s": 0.0,
 "accuracy_scores": [
   {"dataset_name": "...", "extractor": "...", "ground_truth_column": "...",
    "score": 0.0, "unit_samples": 0, "num_repeats": 0, "total_samples": 0,
    "duration_s": 0.0, "complete": true, "dataset_type": "...",
    "response_counts": {}, "output_sequence_lengths": {}, "osl_tokenize_s": 0.0}
 ]}
```

`accuracy_scores` is a **list** of per-dataset entries — index it by `dataset_name`, not by
position.

## The submission bundle

What `endpoints-submission-cli` assembles and uploads.

```
<submitting_organization>/
└── <submission_id>/                  # assigned by MLCommons; one per submission
    ├── cli_metadata.json             # which CLI shaped this bundle
    │
    ├── src/                          # SHARED across the whole submission
    │   └── <implementation>/         # e.g. trtllm/, vllm/, sglang/
    │       ├── README.md             # REQUIRED — how to build/launch and reproduce a point
    │       └── <endpoint interface code, infra setup, client harness>
    │
    ├── docs/                         # SHARED across the whole submission
    │   ├── calibration.adoc          # if weight transformations were applied
    │   ├── software_disclosure.md
    │   └── <additional documentation>
    │
    └── results/
        └── <system>/                 # e.g. H200-SXM-141GBx8_TRT/
            ├── system_power.json     # REQUIRED, one per system — provisioned power
            └── <model_name>/         # e.g. deepseek-r1/, gpt-oss-120b/
                └── r<N>/             # one PARETO POINT per concurrency (r1, r32, r256, …)
                                      #   a dedicated Offline run's N is the dataset size
                    ├── point.yaml
                    ├── system_desc.json
                    ├── result_summary.json
                    ├── accuracy_results.json
                    ├── config.yaml            # OPTIONAL as of v1.0
                    └── server_configs/        # OPTIONAL, point-specific, submitter-defined
```

### Shared versus per-point

`src/` and `docs/` are written once per submission; only what changes with concurrency lives under
`r<N>/`. The split, and the `shared_src` / `shared_docs` pointers that tie each point to the shared
content, are set by [rules §8.1][rules-8.1].

!!! note "Unresolvable pointers reject the submission"
    The checker enforces the pointers with `shared-path-resolution`: a point whose `shared_src` or
    `shared_docs` doesn't resolve to a directory under the submission root fails it.

### `system_desc.json` is per point, `system_power.json` is per system

Since policies PR #119 **every Pareto point carries its own `system_desc.json`**, and the checker
verifies that all points of a curve describe the same system.

`system_power.json` is the one per-system file, because provisioned power is fixed across the whole
curve. The builder takes it from whichever of the system's run folders supply it and writes it to
`results/<system>/`. If two run folders of the same system carry different contents, the build
fails. See [`system_power.json`](system-power-json.md).

### `cli_metadata.json`

Written inside `<submission_id>/`, not at the organisation level.

```json
{
  "command": "remove-run",
  "cli_version": "1.0.0.0",
  "most_recent_cli_used": "1.2.0.0",
  "created_at": "2026-08-20T17:08:02.936070Z"
}
```

| Key | Meaning |
|---|---|
| `command` | The command that assembled this bundle |
| `cli_version` | The CLI that **created** the submission |
| `most_recent_cli_used` | The CLI that ran **this** command |
| `created_at` | Submission creation time — not rebuild time |

## Known gaps

??? warning "Things the tooling does not do that you might expect"
    - **`run_metadata.json` is never written** by the reference client. `build_archive(..., run_date=…)`
      injects `run_date` into that file, so for a client-written run folder the injection is a silent
      no-op. The field only takes effect if you author the file.
    - **`test_started_at` is 0.** The real wall-clock start is not recorded in `result_summary.json`,
      so `started_at` / `finished_at` are reconstructed as `now() - duration_ns`. The window length is
      correct; its absolute position is the *upload* time, not the run time.
    - **`config.yaml` is sanitized.** Report directories contain a `config.yaml` with credentials and
      other secrets replaced by `<redacted>`. Restore them before reusing that file as benchmark input.

*Last verified against: `mlcommons/endpoints-submission-cli@main` (f25f71e) and
`mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
