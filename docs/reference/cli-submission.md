# Submission CLI — `endpoints-submission-cli`

Registers benchmark runs, assembles submission bundles, runs compliance checks and uploads to the
PRISM Submission API. Used in [step 7](../workflow/submit.md).

!!! note "Tracks upstream"
    This page tracks `docs/endpoints-cli/usage/` in `mlcommons/endpoints-submission-cli`. Run
    `--help` on any command for the authoritative flag list.

## Requirements

- Python 3.10+
- [`gh` CLI](https://cli.github.com/) — required for creating, updating and withdrawing submissions
- A PRISM API token in `mlc_…` format

## Installation

```bash
pip install endpoints-submission-cli
```

Ships both `endpoints-submission-cli` and [`submission-checker`](cli-checker.md).

## Authentication

| Environment variable | Default | Description |
|---|---|---|
| `PRISM_USER_API_TOKEN` | — | API key. Required unless `--token` is passed |
| `MLPERF_API_BASE_URL` | `https://api.mlcommons.org` | Override only for dev/staging |

```bash
export PRISM_USER_API_TOKEN=mlc_your_token_here
```

`--token` is available on every command and wins when both are set.

## Command tree

```
endpoints-submission-cli
├── runs
│   ├── list          List all runs
│   ├── create        Register a run from a local folder
│   ├── get           Fetch run details
│   ├── delete        Delete a run and its archive
│   ├── pin           Pin a run (prevent expiry)
│   └── unpin         Restore normal expiry
└── submissions
    ├── list          List all submissions
    ├── create        Create a submission from registered runs
    ├── create-local  Create from an already-assembled submission directory
    ├── get           Fetch submission details
    ├── update        Update run list or metadata
    ├── withdraw      Withdraw a submission
    └── remove-run    Remove a run from a submission
```

`-j` / `--json` is available on list and get commands for machine-readable output.

---

## `runs create`

Parses a local run folder, registers a run record, and uploads the folder as a `.tar.gz`.

```bash
endpoints-submission-cli runs create --path PATH [--expires-at DATETIME] [--pinned] [--test] [--dry-run]
```

| Flag | Required | Description |
|---|---|---|
| `--path PATH` | yes | Path to the local run folder |
| `--expires-at DATETIME` | no | ISO 8601 expiry. Defaults to server policy |
| `--pinned` | no | Pin immediately to prevent automatic expiry |
| `--test` | no | Mark as a test run, excluded from published reporting. **Fixed at creation** — no API to change it later |
| `--dry-run` | no | Print the parsed payload and exit without calling the API |

**Rollback:** if the archive upload fails after the record was created, the CLI deletes the record
to leave a clean state. If that delete also fails, the orphaned run ID is printed.

!!! note "`--test` is invisible in `runs list`"
    That view is served by the API's `RunSummary` schema, which does not carry `is_test`.
    `runs get` shows `Test Run │ Yes`, and `submissions get` marks a flagged run with a yellow `*`.

**Payload mapping:**

| Payload field | Source |
|---|---|
| `system_info` | `system_desc.json` (submitter-authored) |
| `config` | `config.yaml` |
| `result_summary` | `performance/result_summary.json` |
| `benchmark_version` | `result_summary.git_sha` |
| `started_at` / `finished_at` | `result_summary.test_started_at` + `duration_ns` |

Everything else in the folder, including `accuracy/` and `events.jsonl`, reaches the server inside
the uploaded archive rather than the JSON payload.

## `runs get`

```bash
endpoints-submission-cli runs get --run-id RUN_ID [--download-to DIR] [-j]
```

Renders a table plus a System Info table. `config` and `result_summary` are nested blobs omitted
from the table, so use `-j` to see them. With `--download-to`, the archive is saved as `<run-id>.tar.gz` and the
path is printed to **stderr**, so `runs get -j` stays pipeable into `jq`.

## `runs delete` / `pin` / `unpin`

```bash
endpoints-submission-cli runs delete --run-id RUN_ID
endpoints-submission-cli runs pin    --run-id RUN_ID
endpoints-submission-cli runs unpin  --run-id RUN_ID
```

!!! warning "Runs in an active submission cannot be deleted"
    Withdraw the submission first, then delete the run. Deletion removes the archive from storage
    first (a 404 is ignored), then the record from the database.

---

## `submissions create`

The full pipeline. Aborts before uploading anything if the checker finds errors.

```bash
endpoints-submission-cli submissions create \
  --division DIVISION --scenario SCENARIO --availability AVAILABILITY \
  --run-ids RUN_ID [--run-ids RUN_ID ...] \
  [--provisional] [--yes] [--publication-cycle CYCLE] \
  [--target-availability-date DATE] [--embargo-date DATETIME] [--dry-run]
```

| Flag | Required | Description |
|---|---|---|
| `--division` | yes | `standardized` \| `serviced` \| `rdi` |
| `--scenario` | yes | `cop` \| `con` |
| `--availability` | yes | `available` \| `preview` \| `rdi` |
| `--run-ids RUN_ID` | yes | Repeatable — one flag per run |
| `--provisional` | no | Request provisional publication. Prompts for confirmation |
| `--yes` / `-y` | no | Skip the provisional confirmation prompt |
| `--publication-cycle` | no | e.g. `2026-10-C1` |
| `--target-availability-date` | conditional | `YYYY-MM-DD`. **Required** with `--availability preview` |
| `--embargo-date` | no | ISO 8601 datetime. Without `--provisional`, holds the finalized result; with it, holds the provisional result **and** the start of review |

`--provisional` and `--embargo-date` together select one of the three publication modes. See
[step 7](../workflow/submit.md#2-create-the-submission) for the combinations.
| `--dry-run` | no | Assemble and check, print the layout, exit |

**What it does:**

1. Download all run archives
2. Assemble the submission folder
3. Run the [Submission Checker](cli-checker.md) — aborts with exit 1 on compliance errors
4. `POST /submissions`
5. Upload the bundle
6. `PATCH /submissions/{id}` to set `status=REVIEW_PENDING`

**Rollback:** if the bundle upload fails, the submission is automatically withdrawn. If step 6 alone
fails, both submission and bundle exist. It's a warning, not fatal, and the status can be set manually.

!!! note "The CLI does not open the review PR"
    `pr_url` and `pr_number` appear on the submission record once whatever opens it has set them.

## `submissions get`

```bash
endpoints-submission-cli submissions get --submission-id SUB_ID [-j]
```

Renders every field the API returns: classification, test flag, publication cycle, embargo date,
`Reviewers Assigned` (a count; reviewer identities are never exposed), checker/API/CLI versions, PR
references, and all lifecycle timestamps in order. Embedded runs follow in their own table.

## `submissions update`

```bash
endpoints-submission-cli submissions update --submission-id SUB_ID \
  [--run-ids RUN_ID ...] [--target-availability-date DATE] \
  [--publication-cycle CYCLE] [--embargo-date DATETIME]
```

With no flags it prints a warning and makes no API call.

**Metadata only** → a single `PATCH`, no rebuild.

**With `--run-ids`** → full rebuild: fetch current list, reject additions, PATCH, download remaining
archives, reassemble, re-check, re-upload. If anything after the PATCH fails, the run list is
restored.

!!! danger "`--run-ids` may only shrink the list"
    The Submission Rules no longer provide a post-submission window for adding measurement points,
    so a list containing a run the submission does not already have is **rejected** before anything
    is fetched or patched.

## `submissions remove-run`

```bash
endpoints-submission-cli submissions remove-run --submission-id SUB_ID --run-id RUN_ID
```

Registers the removal, then rebuilds, re-checks and re-uploads if runs remain. If none remain,
steps 2–4 are skipped with a warning. Rollback re-adds the run on failure.

!!! danger "Removed points cannot be replaced"
    Withdrawn points do **not** count toward the minimum point count, and that shortfall cannot be
    repaired by adding another point. There is no `add-run`. If a submission needs a different set of
    runs, create a new one.

## `submissions withdraw`

```bash
endpoints-submission-cli submissions withdraw --submission-id SUB_ID
```

Marks the submission `WITHDRAWN` and deletes the stored bundle. Archive deletion is best-effort: a
failure is a warning and does not change the exit code. The CLI does not close the review PR.

## `submissions create-local`

Creates a submission from an **already-assembled** submission directory rather than from registered
runs. Takes `--path` (org-level directory) plus the same classification flags. `--test` sets the
test flag on every run it registers, so a test submission leaves no untagged runs behind.

## Status lifecycle

See [Submission states](submission-states.md).

*Last verified against: `mlcommons/endpoints-submission-cli@main` (f25f71e) and
`mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
