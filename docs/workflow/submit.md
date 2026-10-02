# 7. Submit

> Produces: an uploaded bundle, a submission ID, and a peer-review PR in the private MLCommons
> review repository.

!!! note "Before you begin"
    - Completed [6. Validate locally](validate.md) with exit code 0
    - You have decided on publication status and publication mode
    - `gh` is installed and authenticated

## What you'll do

- Register each run, collecting a run ID
- Create the submission from those run IDs
- Confirm it landed

## Steps

### 1. Register each run

```bash
endpoints-submission-cli runs create --path ./results/point-c64
# → Run created: d5d9873e-5eca-4f8d-a487-4be1cb8b440c
```

This parses the run folder, registers a run record, and uploads the folder as a `.tar.gz`. Repeat
for every point including the accuracy run, and keep the returned run IDs.

Useful flags: `--dry-run` prints the parsed payload without calling the API; `--pinned` prevents
automatic expiry; `--expires-at` sets a custom expiry; `--test` marks a run as excluded from
published reporting.

!!! warning "The folder layout is the only one accepted"
    `result_summary.json` must be under `performance/`, not at the top level. A flat layout is
    rejected with *"missing required file(s): performance/result_summary.json"*.

If the archive upload fails after the run record was created, the CLI deletes the record
automatically to leave a clean state. If that cleanup also fails, it prints the orphaned run ID for
you to remove.

### 2. Create the submission

```bash
endpoints-submission-cli submissions create \
  --division standardized \
  --scenario cop \
  --availability available \
  --run-ids <run-id-1> \
  --run-ids <run-id-2> \
  …
```

| Flag | Required | Notes |
|---|---|---|
| `--division` | yes | `standardized` / `serviced` / `rdi` |
| `--scenario` | yes | `cop` / `con` |
| `--availability` | yes | `available` / `preview` / `rdi` |
| `--run-ids` | yes | Repeat the flag once per run |
| `--provisional` | no | Publish before review completes, tagged *peer review pending* |
| `--target-availability-date` | conditional | **Required** with `--availability preview` |
| `--embargo-date` | no | ISO 8601. What it holds back depends on `--provisional`, below |
| `--publication-cycle` | no | e.g. `2026-10-C1` |
| `--dry-run` | no | Assemble and check only |

What the command actually does: downloads the run archives, assembles the bundle, **runs the
checker** (aborting on errors before anything is uploaded), registers the submission, uploads the
bundle, and sets status to `REVIEW_PENDING`.

If the bundle upload fails, the submission is withdrawn automatically to leave a clean state.

The two publication flags combine into the three publication modes:

| Flags | Mode | What happens |
|---|---|---|
| neither | Confidential review | Review starts when checks pass; published at the first cohort after finalization |
| `--embargo-date` only | Confidential review, embargoed | Review starts when checks pass; the finalized result is held until the date, up to 60 days after review completes |
| `--provisional` | Provisional publication | Published tagged *peer review pending*; review starts at that point |
| `--provisional --embargo-date` | Provisional, embargoed | Nothing public, and **no review**, until the date |

!!! danger "The mode is irrevocable"
    The publication mode can't be changed after submission. Submitters who don't opt in to
    provisional publication can't request it later. Anyone quoting a *peer review pending* result,
    including the press, must carry the MLCommons footnote stating results are preliminary and
    subject to change.

!!! warning "Preview commits you to a deadline"
    `--availability preview` requires a target availability date within **180 days** of first
    publication, and commits you to re-submitting as Available at that point. Miss it without an
    extension and the result is **invalidated and removed**, not archived. See
    [Publication status](../rules/publication-status.md).

### 3. Points are fixed at creation

!!! danger "There is no way to add a point after submitting"
    The post-submission window for adding measurement points has been removed from the Submission
    Rules. There is no `add-run` command, and `submissions update --run-ids` **rejects** any list
    that would add a run. The list may only shrink.

    You may withdraw a faulty point during peer review with `submissions remove-run`. But withdrawn
    points do not count toward the minimum point count, and that shortfall **cannot be repaired**.
    Withdrawing the Offline point, or one of the five accuracy points, is just as final. If a
    submission needs a different set of runs, create a new one.

This is why [step 3](plan-your-curve.md) suggests planning a spare point.

## Verify

```bash
endpoints-submission-cli submissions get --submission-id <submission-id>
```

Confirm:

- **Status** is `REVIEW_PENDING`
- **Division, scenario and availability** are what you intended
- **Publication cycle** and **embargo date** are as expected
- **Reviewers Assigned** shows a count — reviewer identities are never exposed by the API
- The embedded runs table lists every point you meant to include

Also keep the checker log the command wrote (`submission_checker_<timestamp>.log`).

!!! note "`pr_url` may be empty at first"
    The CLI no longer opens the review pull request itself. `pr_url` and `pr_number` appear on the
    submission record once whatever opens it has set them.

## Next

→ [8. After you submit](after-submission.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#submission-failures).
