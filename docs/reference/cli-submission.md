# Submission CLI — `endpoints-submission-cli`

Registers benchmark runs, assembles submission bundles, runs compliance checks and uploads to the
PRISM Submission API. Used in [step 6](../workflow/register-runs.md), [step
7](../workflow/validate.md) and [step 8](../workflow/submit.md).

Installation, authentication, and every command and flag are documented in
[`mlcommons/endpoints-submission-cli`](https://github.com/mlcommons/endpoints-submission-cli):

- [README](https://github.com/mlcommons/endpoints-submission-cli/blob/main/README.md)
- [Getting started](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/getting-started.md)
- [`runs` commands](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/runs.md)
- [`submissions` commands](https://github.com/mlcommons/endpoints-submission-cli/blob/main/docs/endpoints-cli/usage/submissions.md)

Run `--help` on any command for the authoritative flag list. You need a [PRISM API
key](../understand/eligibility/prism-api-key.md) for every command.

The package also ships the checker, as `endpoints-submission-cli check-submission`. Its checks are
described in the README's
[`submission-checker`](https://github.com/mlcommons/endpoints-submission-cli/blob/main/README.md#submission-checker)
section, but that command isn't installed (**B16** in [Open questions](../help/open-questions.md)).
How `--provisional` and `--embargo-date` combine into the publication modes is in [step
8](../workflow/submit.md#create-the-submission). Status values are in [Submission
states](submission-states.md).

There's no command to add a run to a submission; see [step
8](../workflow/submit.md#points-fixed-at-creation).

*Last verified against: `mlcommons/endpoints-submission-cli@main` (f25f71e) and
`mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
