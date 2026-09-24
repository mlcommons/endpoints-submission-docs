# Submission checker — `submission-checker`

Validates a submission folder against the automated compliance rules — the same checks that run
server-side during Week 0. Used in [step 6](../workflow/validate.md).

Ships with [`endpoints-submission-cli`](cli-submission.md). Use **`v1.0.1.0` or later**: that
release added the Offline, power, accuracy-coverage, steady-state and drafter checks.

## `check`

```bash
submission-checker check /path/to/submission
```

The path may be the submitting organisation's directory or a `<submission_id>/` directory below it.
A submission root is the level holding `results/` and `docs/`.

| Flag | Description |
|---|---|
| `--strict` | Treat warnings as errors — exit 1 on any warning |
| `--quiet` / `-q` | Suppress INFO-level passing checks |
| `--output FILE` / `-o FILE` | Write full results as JSON |
| `--seed-sets FILE` | Published seed sets to check against. Defaults to the bundled set; also settable via `$MLPERF_ENDPOINTS_SEED_SETS` |
| `--approved-drafters FILE` | Published approved-drafter list to check against. Defaults to the bundled list, which is empty; also settable via `$MLPERF_ENDPOINTS_APPROVED_DRAFTERS` |

**Exit codes:** `0` all checks passed · `1` one or more errors, or warnings under `--strict`.

!!! tip "Run with `--strict` at least once"
    Warnings are where methodology objections come from. A point that merely warns on duration or
    region placement is the kind of thing reviewers raise objections about in Weeks 1–3.

## `regions`

```bash
submission-checker regions --max-concurrency 1024 --min-concurrency 16
```

Prints the concurrency range for each region for a given `(C_max, C_min)` pair, using the reference
algorithm.

`--min-concurrency` defaults to 32. In a real submission `C_min` is **derived** from the lowest
measurement point rather than declared, so the boundaries you get here are only correct if you pass
the `C_min` you will actually submit.

Full algorithm and pre-computed tables: [Metrics and regions](metrics-and-regions.md).

## Programmatic API

```python
from pathlib import Path
from submission_checker import SubmissionChecker, Report

checker = SubmissionChecker(Path("/submissions/acme_corp"))
report = checker.run()

if report.passed:
    print("All checks passed")
else:
    for result in report.errors:
        print(f"[{result.rule}] {result.message}")
```

`Report` also exposes `report.warnings` and serialises via `report.model_dump_json()`. Wiring this
into CI validates your disclosure files on every change.

## Bundled data

| File | Contents | Override |
|---|---|---|
| `src/submission_checker/data/seed_sets.yaml` | A mirror of the published `seedset.yaml`: set `A`, `cohort-id: 2026-10-C1`. The four-cohort adoption window is derived from the cohort ID | `--seed-sets FILE` or `$MLPERF_ENDPOINTS_SEED_SETS` |
| `src/submission_checker/data/approved_drafters.yaml` | `drafters: []`. No list has been published, so any point using speculative decoding fails `approved-drafter` | `--approved-drafters FILE` or `$MLPERF_ENDPOINTS_APPROVED_DRAFTERS` |

Override either one to check against something published after your installed release.
Checkers before `v1.0.1.0` shipped a seed-set file with no cohort keys, so `seed-set-adoption`
reported **SKIP**.

## Expected layout

```
<submitting_organization>/
└── <submission_id>/
    ├── src/
    │   └── <implementation>/
    │       └── README.md            # required
    ├── docs/
    └── results/
        └── <system>/
            ├── system_power.json      # REQUIRED, one per system
            └── <model_name>/
                └── r<N>/
                    ├── point.yaml
                    ├── system_desc.json
                    ├── result_summary.json
                    ├── accuracy_results.json
                    ├── config.yaml            # OPTIONAL as of v1.0
                    └── server_configs/        # OPTIONAL
```

`src/` and `docs/` are shared across the whole submission; each `point.yaml` names them via
`shared_src` and `shared_docs`, which must resolve to a directory under the submission root.

## What gets checked

Eight families of rules. Full cross-walk from rule ID to clause, with severity:
[Compliance checks](compliance-checks.md).

| Family | Covers |
|---|---|
| Structure | Directories, required files, shared-path resolution |
| System description | Schema validity, consistency, model name, `C_max`, `tps_utilization`, `system_power.json` |
| Regions | Derived `C_min`, boundary computation, coverage of all four regions, the Offline point, point count and cap |
| Measurement points | `point.yaml` schema, disclosure completeness, load pattern, streaming, duration, steady-state block, query count, warmup |
| Seed binding | Set consistency, membership, runtime match, target cohort, adoption window |
| Speculative decoding | Drafter on the approved list, approval lead time |
| Metrics | Result schema, duration, sample accounting, `system_tps`, TPOT P90, `tps_per_user`, `system_tps_per_kw`, agentic interactivity |
| Accuracy | Presence, coverage of the required points, validity, sample count, quality gate |

*Last verified against: `mlcommons/endpoints-submission-cli@main` (f25f71e, `v1.0.1.0`), 2026-09-24.*
