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
| `--approved-drafters FILE` | Check against a specific published approved-drafter list |

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

The checks fall into eight groups: structure, system description and power, regions and the
Offline point, measurement points, seed binding, speculative decoding, metrics, and accuracy.
Every rule ID maps to a clause in the rules.

Full cross-walk from rule ID to clause: [Compliance checks](../reference/compliance-checks.md).

The ones that **reject** rather than flag:

- Submission completeness — required files, YAML, artifacts, system descriptions
- `shared_src` / `shared_docs` resolution
- A `system_power.json` for each system
- Point count ≥ 8, or ≥ 7 if you elected `C_max` or the benchmark is agentic
- Exactly one Offline point (none for agentic)
- Coverage of Ultra Low, Low, Medium and High Concurrency
- `C_max` declared and > 32
- Accuracy at all five required points (four for agentic), each passing the quality target
- Seed-set validity
- Any drafter used is on the approved list, and was approved at least two cohorts earlier

!!! warning "Use checker `v1.0.1.0` or later"
    The Offline, power, accuracy-coverage, steady-state and drafter checks arrived in `v1.0.1.0`
    (2026-09-23). An older checker passes submissions the server will reject. Check with
    `endpoints-submission-cli --version`, and upgrade with `pip install -U endpoints-submission-cli`.

!!! danger "A missing Offline point only warns locally"
    The checker can't tell an agentic benchmark from a single-turn one yet, so a submission with
    no `offline` declaration gets a **warning** (`offline-point-present`), not an error. For a
    non-agentic benchmark the rules treat that as a reject. Run with `--strict` or read the
    warnings. Don't take a clean exit code as proof you have an Offline point.

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
