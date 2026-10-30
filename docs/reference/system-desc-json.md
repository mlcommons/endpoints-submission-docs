# `system_desc.json`

The hardware and software description of the system under test. **One per Pareto point**.

!!! danger "Not written by the reference client"
    You supply it and drop it into each run folder before upload. See [step
    5](../workflow/author-disclosures.md).

The checker verifies that every point of a frontier describes the **same** system.

## Fields and template

The fields, and what each one means, are defined in [rules §8.2][rules-8.2]. There are two ways to
produce the file:

- **Capture it with [`mlperf-sysinfo`](https://docs.mlcommons.org/mlperf-sysinfo/).** It reads the
  machine under test and writes `system_desc.json` using its `endpoints` profile. How to install and
  run it is in its docs.
- **Copy the template** in [§8.2.1][rules-8.2.1], which includes the nesting of `node_types` and
  `accelerator_info`, and fill it in.

!!! note "Availability fields"
    `availability_url` is required when `"system_availability_status": "Available"`
    ([Submission Rules §7.3.7][srules-7.3.7]). `target_availability_date`, an ISO 8601 date, is
    required when it is `Preview` ([Submission Rules §7.3.4][srules-7.3.4]).

!!! warning "`tps_utilization` is computed"
    The checker recomputes it against your own frontier (`tps-utilization`).

!!! warning "`cooling` sets your power overhead"
    The checker uses `cooling` to pick the overhead fraction for
    [`system_power.json`](system-power-json.md): `0.30` for liquid-cooled, `0.50` for air-cooled.
    Leave it empty and `power-descriptor` fails. It must also agree with the `cooling` in
    `system_power.json` ([E.2][rules-e2]).

!!! note "No `model_name` here"
    The benchmark model name goes in `point.yaml` ([§8.3][rules-8.3]). §8.2 no longer lists it, and
    the checker ignores a `model_name` left in this file.

!!! question "`disaggregated` is truncated upstream"
    The field definition in the source data dictionary is cut off mid-sentence and carries an
    upstream TODO to verify the full text with the dictionary owner. Confirm the intended semantics
    before relying on it.


Provisioned power is **not** in this file. It goes in a separate, per-system
[`system_power.json`](system-power-json.md).

*Last verified against: `mlcommons/endpoints_policies@main` (1818269) and
`mlcommons/endpoints-submission-cli@main` (a42a056, `v1.1.0.0`), 2026-10-02.*
