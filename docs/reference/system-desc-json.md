# `system_desc.json`

The hardware and software description of the system under test. **One per Pareto point**, since
policies PR #119 there is no shared per-system file.

!!! danger "Authored by you"
    Not written by the reference client. You author it and drop it into each run folder before
    upload. Authored in [step 5](../workflow/author-disclosures.md).

The checker verifies that every point of a curve describes the **same** system
(`system-description-consistency`).

## Fields and template

The fields, and what each one means, are defined in [rules §8.2][rules-8.2]. The JSON template,
including the nesting of `node_types` and `accelerator_info`, is in [§8.2.1][rules-8.2.1]. Copy
the template from there rather than from a derived page, so you pick up changes to it.

A few fields need more care than their one-line definition suggests:

!!! warning "`tps_utilization` is computed, not chosen"
    The checker recomputes it against your own curve (`tps-utilization`). You cannot fill it in until
    every point has run.

!!! note "`link_config` was removed"
    Policies commit `b4ab404` dropped `link_config` from the §8.2 field table, but the §8.2.1
    template still carries it. Leave it empty. Tracked as **B7**.

!!! question "`disaggregated` is truncated upstream"
    The field definition in the source data dictionary is cut off mid-sentence and carries an
    upstream TODO to verify the full text with the dictionary owner. Confirm the intended semantics
    before relying on it.

## Checker rules that read this file

| Rule | Checks |
|---|---|
| `system-description-present` | Every point has a `system_desc.json` |
| `system-description-valid` | It parses against the `SystemDescription` schema |
| `system-description-consistency` | Every point of a curve describes the same system |
| `model-name-valid` | `model_name` is one of the round's supported models |
| `model-name-consistency` | It matches the results directory name |
| `max-concurrency-declared` | `max_supported_concurrency` present and > 32 |
| `tps-utilization` | Equals `system_tps / max(system_tps)` over the point's own curve |

!!! note "Field name drift"
    Most of the rules now say `system_desc.json`, but the §9.1 *Max concurrency declared* row and
    the Submission Rules still say `system_desc_id.json`, and the result-ID definition refers to a
    `benchmark_model` field. The tooling uses `system_desc.json` and `model_name`, and that's what
    the checker reads.

Provisioned power is **not** in this file. It goes in a separate, per-system
[`system_power.json`](system-power-json.md).

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
