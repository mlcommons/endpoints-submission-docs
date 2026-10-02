# 6. Validate locally

> Produces: a clean `submission-checker` report.

!!! note "Before you begin"
    - Completed [5. Author the disclosure files](author-disclosures.md)
    - Every run folder has `system_desc.json` and `point.yaml`

!!! danger "You only get one attempt at Week 0"
    The same checks run on the server after you submit. **If your submission fails any automated
    check by the end of Week 0, it's rejected.** You can't patch it in place: you fix the issues and
    resubmit as an entirely new submission, losing your place in the cohort queue. Local validation
    is the cheap version of that.

## What you'll do

- Run `submission-checker` against an assembled submission directory
- Or run `submissions create --dry-run`, which assembles and checks without uploading
- Fix, re-run, repeat until clean

## Steps

### 1. Check an assembled submission

```bash
submission-checker check /path/to/submission
```

The path may be the submitting organisation's directory or a `<submission_id>/` directory below it.
A submission root is the level holding `results/` and `docs/`.

| Flag | Does |
|---|---|
| `--strict` | Treat warnings as errors — exit 1 on any warning |
| `--quiet` / `-q` | Suppress INFO-level passing checks |
| `--output FILE` / `-o` | Write the full result as JSON |
| `--seed-sets FILE` | Check against a specific published seed-set file |

Exit codes: `0` all checks passed, `1` one or more errors (or warnings under `--strict`).

### 2. Or dry-run the real pipeline

Once your runs are registered ([step 7](submit.md)), this exercises the actual assembly path:

```bash
endpoints-submission-cli submissions create \
  --division standardized \
  --scenario cop \
  --availability available \
  --run-ids <run-id> --run-ids <run-id> … \
  --dry-run
```

It downloads the archives, assembles the folder, runs the checker, prints the layout, and exits
without creating anything. This is the closest local approximation to what Week 0 will do.

### 3. Read the report properly

Errors and warnings are not the same thing:

| | Meaning | Consequence |
|---|---|---|
| **Error** | A rule whose failure action is *reject* or *reject points* | Blocks submission |
| **Warning** | A rule that is flagged rather than fatal | Does not block locally, but reviewers see it and may object |

!!! tip "Run with `--strict` at least once"
    Warnings are where methodology objections come from. A point that merely *warns* on duration or
    region placement is exactly the kind of thing a reviewer files an objection about in Weeks 1–3,
    and an objection costs you far more than a re-run does now.

### 4. Know what is being checked

The checks fall into seven groups: structure, system description, regions, measurement points,
seed binding, metrics, and accuracy. Every rule ID maps to a clause in the rules.

Full cross-walk from rule ID to clause: [Compliance checks](../reference/compliance-checks.md).

The ones that **reject** rather than flag:

- Submission completeness — required files, YAML, artifacts, system descriptions
- `shared_src` / `shared_docs` resolution
- Point count ≥ 7
- Coverage of Ultra Low, Low, Medium and High Concurrency
- `C_max` declared and > 32
- Accuracy — at least one run passing the quality target
- Seed-set validity

### 5. Use the programmatic API for CI

```python
from pathlib import Path
from submission_checker import SubmissionChecker

report = SubmissionChecker(Path("/submissions/acme_corp")).run()

if not report.passed:
    for result in report.errors:
        print(f"[{result.rule}] {result.message}")
```

`report.warnings` and `report.model_dump_json()` are also available. Wiring this into CI so every
change to your disclosure files is checked is worth the hour it takes.

### 6. Look at your curves (optional)

The checker tells you whether the submission is valid, not whether the curve looks the way you
expect. To see it, render the same folder in a local dashboard:

```bash
pip install mlperf-viz
mlperf-viz /path/to/submission
```

It opens the Pareto Explorer in your browser; nothing is uploaded. Each `<System>/<Model>` pair is
one curve. Flags and formats: [Results visualizer](../reference/cli-visualizer.md).

## Verify

```bash
submission-checker check /path/to/submission --strict --output checker.json
echo "exit: $?"
```

You're ready to submit when the exit code is `0`. Keep `checker.json`, which is useful evidence if a
reviewer later questions something the checker already passed.

## Next

→ [7. Submit](submit.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#validation-failures) and
[Why submissions get rejected](../rules/rejection-reasons.md).
