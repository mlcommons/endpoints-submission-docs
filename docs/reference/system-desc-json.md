# `system_desc.json`

The hardware and software description of the system under test. **One per Pareto point**.

!!! danger "Not written by the reference client"
    You supply it and drop it into each run folder before upload. See
    [step 5](../workflow/author-disclosures.md).

The checker verifies that every point of a curve describes the **same** system.

## Fields and template

The fields, and what each one means, are defined in [rules §8.2][rules-8.2]. There are two ways to
produce the file:

- **Capture it with [`mlperf-sysinfo`](https://docs.mlcommons.org/mlperf-sysinfo/).** It reads the
  machine under test and writes `system_desc.json` using its `endpoints` profile. How to install and
  run it is in its docs.
- **Copy the template** in [§8.2.1][rules-8.2.1], which includes the nesting of `node_types` and
  `accelerator_info`, and fill it in.

!!! warning "`tps_utilization` is computed"
    The checker recomputes it against your own curve (`tps-utilization`).

!!! note "`link_config` was removed"
    Policies commit `b4ab404` dropped `link_config` from the §8.2 field table, but the §8.2.1
    template still carries it. Leave it empty. Tracked as **B7**.

!!! question "`disaggregated` is truncated upstream"
    The field definition in the source data dictionary is cut off mid-sentence and carries an
    upstream TODO to verify the full text with the dictionary owner. Confirm the intended semantics
    before relying on it.

!!! note "Field name drift"
    Most of the rules now say `system_desc.json`, but the §9.1 *Max concurrency declared* row and
    the Submission Rules still say `system_desc_id.json`, and the result-ID definition refers to a
    `benchmark_model` field. The tooling uses `system_desc.json` and `model_name`, and that's what
    the checker reads.

Provisioned power is **not** in this file. It goes in a separate, per-system
[`system_power.json`](system-power-json.md).

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
