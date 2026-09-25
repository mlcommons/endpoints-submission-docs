# 7. Validate locally

> Produces: a clean checker report.

!!! note "Before you begin"
    - Completed [6. Register your runs](register-runs.md)
    - You have the run ID of every point

!!! danger "You only get one attempt at Week 0"
    The same checks run on the server after you submit, and a submission that fails any of them by
    the end of Week 0 is rejected ([Submission Rules §6.1][srules-6.1]). You can't patch it in
    place: you resubmit as an entirely new submission. Local validation is the cheap version of
    that.

## What you'll do

- Run [`submissions create --dry-run`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/submissions.md#submissions-create), which builds the bundle from your runs and checks it
  without creating anything
- Or run `endpoints-submission-cli check-submission` against a submission directory you
  assembled yourself
- Fix, re-register, re-run, repeat until clean

## Steps

### 1. Dry-run the real pipeline

Run [`submissions
create`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/submissions.md#submissions-create)
with `--dry-run`, the same classification you'll use in [step 8](submit.md) (division, scenario,
availability, publication cycle and publication mode, which you decided in [Before you
begin](before-you-begin.md)), and the run IDs from step 6. It takes the same path step 8 will,
without creating anything, so it's the closest local approximation to what Week 0 will do.

The full report, warnings included, goes to a `submission_checker_<timestamp>.log` file in the
current directory. If a check fails, fix the run folder, [register it
again](register-runs.md#re-register), and re-run with the new run ID.

### 2. Or check a directory you assembled

Use this for CI, or when you've laid out a submission yourself following [Submission package
layout](../reference/package-layout.md#the-submission-bundle). The command, its flags and exit codes
are in the checker's
[README](https://github.com/mlcommons/endpoints-submission-cli/blob/main/README.md#check-a-submission).

### 3. Read the report properly

Errors and warnings are not the same thing. The checker sets the severity, and it mostly follows the
failure actions in [§9.1 of the rules][rules-9.1]. Some rules §9.1 only flags are errors locally,
listed in [Why submissions get
rejected](../rules/rejection-reasons.md#flagged-by-the-rules-but-they-fail-the-local-check).

| | Meaning | Consequence |
|---|---|---|
| **Error** | A check that fails | Blocks `submissions create` |
| **Warning** | A check that is flagged rather than fatal | Does not block locally, but reviewers see it and may object |

!!! tip "Read the warnings, not just the exit code"
    The dry-run fails only on errors, so warnings pass silently unless you read the log.
    `submissions create --dry-run` has no strict mode; `endpoints-submission-cli check-submission
    <dir> --strict` treats warnings as errors. Warnings are where methodology objections come from.
    A point that merely *warns* on duration or region placement is exactly the kind of thing a
    reviewer files an objection about in Weeks 1–3, and an objection costs you far more than a
    re-run does now.

### 4. Know what is being checked

What each check looks at is in the checker's
[README](https://github.com/mlcommons/endpoints-submission-cli/blob/main/README.md#what-gets-checked).
Which of them reject rather than flag is in [§9.1 of the rules][rules-9.1], and the cross-walk from
checker rule ID to clause is in [Compliance checks](../reference/compliance-checks.md).

!!! warning "Use checker `v1.0.1.0` or later"
    The Offline, power, accuracy-coverage, steady-state and drafter checks arrived in `v1.0.1.0`
    (2026-09-23). An older checker passes submissions the server will reject. Check with
    `endpoints-submission-cli --version`, and upgrade with `pip install -U
    endpoints-submission-cli`.

!!! danger "A missing Offline point only warns locally"
    The checker can't tell an agentic benchmark from a single-turn one yet, so a submission with no
    `offline` declaration gets a **warning** (`offline-point-present`), not an error. For a
    non-agentic benchmark the rules treat that as a reject. Read the warnings, or run
    `check-submission --strict`. Don't take a clean exit code as proof you have an Offline point.

### 5. Use the programmatic API for CI

The checker can also run from Python; see its [Programmatic
API](https://github.com/mlcommons/endpoints-submission-cli/blob/main/README.md#programmatic-api).
Wiring it into CI so every change to your disclosure files is checked is worth the hour it takes.

## Next

→ [8. Submit](submit.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#validation-failures) and [Why submissions
get rejected](../rules/rejection-reasons.md).
