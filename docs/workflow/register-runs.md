# 6. Register your runs

> Produces: one run ID per run folder, each with its folder uploaded to MLCommons.

!!! note "Before you begin"
    - Completed [5. Author the disclosure files](author-disclosures.md), including its Verify check
    - You have your PRISM API key from [step 1](register.md)

A submission isn't built from folders on your disk. It's built from **runs**, which are records on
the MLCommons server that each hold one uploaded run folder. You register every folder from [step
4](run-the-points.md) as a run, then [step 8](submit.md) creates the submission from the run IDs.

## What you'll do

- Give the CLI your API key and check it authenticates
- Preview and register each folder, and note the run IDs
- Pin the runs you intend to submit
- Confirm the list

How each command works, with its flags and output, is in the CLI's [`runs` command
docs](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md).
This page covers what a submission needs from them.

## Steps

### 1. Set your token {#set-your-token}

Every command authenticates with your PRISM API key. Set it as described in
[Authentication](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/getting-started.md#authentication),
then check it with [`runs
list`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md#runs-list).
Treat the key as a credential: don't commit it or paste it into an issue.

### 2. Preview and register each folder

Preview each folder with [`runs create
--dry-run`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md#runs-create),
then register it with `runs create`. Register every folder: each concurrency point, the dedicated
Offline run if you have one, and any accuracy run you ran as its own folder. A `--mode both` run
keeps performance and accuracy in one folder, so it's one run. Keep the returned IDs, because [step
8](submit.md) takes them as `--run-ids`.

The folder layout the CLI expects is in [Submission package layout](../reference/package-layout.md).

!!! warning "Decide on `--test` before you register"
    `--test` excludes a run from published reporting, and it can't be changed after the run is
    created. Use it for rehearsals, never for runs you mean to publish.

### 3. Pin the runs you'll submit

Runs expire under a server policy that isn't published (**C12** in [Open
questions](../help/open-questions.md)), so pin the ones you intend to submit, at creation or
afterwards with [`runs
pin`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md#runs-pin).

### 4. If you change a folder after registering it {#re-register}

The uploaded archive is a copy, and a run can't be updated. Register the folder again, use the new
run ID, and delete the old run with [`runs
delete`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md#runs-delete).

## Verify

[`runs
list`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md#runs-list)
shows one row per folder, with the concurrencies from your plan in [step 3](plan-your-curve.md).
[`runs
get`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md#runs-get)
on each run shows `Test Run: No`; `runs list` doesn't show the test flag.

## Next

→ [7. Validate locally](validate.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#registering-runs).
