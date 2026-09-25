# `system_power.json`

The provisioned-power descriptor for one system. **One per system**, not per point: provisioned
power is fixed for the whole curve. Why it exists and what it's used for:
[What is MLPerf Endpoints?](../understand/what-is-mlperf-endpoints.md#normalised-by-provisioned-power).

!!! danger "Required, and authored by you"
    Every system needs one. A system without it, or with a file from which no total can be
    derived, fails the submission checker. Put it at the top level of at least one run folder of
    the system; the builder places it at
    `results/<system>/system_power.json` in the bundle. Authored in
    [step 5](../workflow/author-disclosures.md#3-write-system_powerjson-for-each-system).

## Fields

The checker reads the shape below. Every field is optional on its own, but the file as a whole has
to yield a total.

| Field | Type | Description |
|---|---|---|
| `provisioned_power_w` | number | A declared total, in **watts**. If present it is used as-is and the component groups are ignored |
| `cpu` | group | Host CPUs |
| `accelerator` | group | GPUs, ASICs and similar |
| `scale_up_network` | group | The high-bandwidth fabric between accelerators — NVLink switches, TPU ICI, UALink over Ethernet. Zero in a system with no switches |
| `scale_out_network` | group | Optional. Only for multi-node systems with a scale-out fabric |
| `overhead_fraction` | number | Other components and cooling: `0.30` liquid-cooled, `0.50` air-cooled |

Each **group** has:

| Field | Type | Description |
|---|---|---|
| `count` | number | Units installed, as provisioned — not the chassis maximum |
| `tdp_per_unit` | number | Rated power per unit, in **watts** |
| `link` | string | A public specification backing the rating |

!!! warning "The rules and the checker name these differently"
    [Rules §4.5.2][rules-4.5.2-power-model] lists the fields as `num_cpu`, `tdp_per_cpu`, `num_accelerator`,
    `tdp_per_accelerator`, `num_switches` and `tdp_per_switch`. The released checker reads nested
    groups with `count`, `tdp_per_unit` and `link`, as above. The checker's spelling is what gets
    validated. Tracked as **B10** in [Open questions](../help/open-questions.md).

## How the total is computed

The formula is in [§4.5.2, Power Model][rules-4.5.2-power-model], and the checker follows it, with
one difference: it adds `scale_out_network` to the major components, while the rules' formula
counts scale-out networking inside the overhead fraction.

## Example

A liquid-cooled node with two CPUs, eight accelerators and one scale-up switch, in the shape the
checker reads:

```json
{
  "cpu":              { "count": 2, "tdp_per_unit": 350,  "link": "https://vendor.example/cpu-spec" },
  "accelerator":      { "count": 8, "tdp_per_unit": 700,  "link": "https://vendor.example/accel-spec" },
  "scale_up_network": { "count": 1, "tdp_per_unit": 3500, "link": "https://vendor.example/switch-spec" },
  "overhead_fraction": 0.30
}
```

## Evidence, partial systems and estimates

These are policy, and live in the rules:

- **Which sources count as evidence** for a power figure, and what a component running below its
  rated TDP has to show: [§4.5.2, Methodology][rules-4.5.2-methodology].
- **How to state the power of a partially populated system**, whether a partial rack, a node with
  empty accelerator slots, or a system under a TDP cap: [§4.5.2.1][rules-4.5.2.1]. Linear scaling
  is allowed only at whole-node granularity.
- **What MLCommons does with values you leave out**, and the **"MLC Estimated Power"** tag the
  result then carries: [§4.5.2][rules-4.5.2].

Where a published figure is a range, the rules use the upper bound.

The bundle build also fails if two runs of the same system carry different `system_power.json`
contents.

??? note "Caveats"
    - **Scope.** Rules §4.5 applies normalization to Standardized CoP and CoN, makes it optional for
      RDI, and defers Serviced. The same section says every submission must include the file, and
      the checker requires it for every system regardless of division. Until that's reconciled,
      include it whatever your division. Tracked as **B11**.
    - **Pending ratification.** Tier definitions, overhead fractions, reference components and the
      metric's name and units are all marked subject to change.

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
