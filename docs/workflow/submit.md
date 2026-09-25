# 8. Submit

> Produces: an uploaded bundle, a submission ID, and a peer-review PR in the private MLCommons
> review repository.

!!! note "Before you begin"
    - Completed [7. Validate locally](validate.md) with exit code 0
    - You have the run IDs from [6. Register your runs](register-runs.md)
    - You have decided on publication status and publication mode

## What you'll do

- Create the submission from your run IDs
- Confirm it landed

## Steps

### 1. Create the submission {#create-the-submission}

Run [`submissions create`][cli-submissions-create] with your classification flags and one
`--run-ids` per run. The command, its flags, and what it does step by step are in the CLI docs. It
runs the checker again and aborts before uploading anything if there are errors.

The two publication flags select one of the publication modes in [Submission Rules
§6.2][srules-6.2], which says when review starts and when each mode publishes:

| Flags | Mode |
|---|---|
| neither | Confidential review |
| `--embargo-date` only | Confidential review, embargoed |
| `--provisional` | Provisional publication |
| `--provisional --embargo-date` | Provisional, embargoed. Nothing is public, and **review doesn't start**, until the date |

!!! danger "The mode is irrevocable"
    You can't change the publication mode after submitting, so you can't opt in to provisional
    publication later ([§6.2][srules-6.2]). Anyone quoting a *peer review pending* result has to
    carry the MLCommons *preliminary* footnote.

!!! warning "Preview commits you to a deadline"
    `--availability preview` needs a target availability date within **180 days** of first
    publication. Miss it without an extension and the result is **invalidated and removed**
    ([Submission Rules §7.3][srules-7.3]). See [Publication status](../rules/publication-status.md).

### 2. Points are fixed at creation {#points-fixed-at-creation}

!!! danger "There is no way to add a point after submitting"
    The Submission Rules no longer have a window for adding measurement points, and the CLI can't
    add a run to a submission ([`submissions update`][cli-submissions-update]). You can withdraw a
    faulty point during peer review with [`submissions remove-run`][cli-submissions-remove-run], but
    withdrawn points don't count toward the minimum ([Submission Rules §8.1][srules-8.1]), and that
    shortfall can't be repaired. If a submission needs a different set of runs, create a new one.

This is why [step 3](plan-your-curve.md) suggests planning a spare point.

## Verify

[`submissions get`][cli-submissions-get] shows status `REVIEW_PENDING`, the division, scenario,
availability, publication cycle and embargo date you intended, and every run you meant to include.

## Next

→ [9. After you submit](after-submission.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#submission-failures).
