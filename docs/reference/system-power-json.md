# `system_power.json`

The provisioned-power descriptor for one system. **One per system**, not per point: provisioned
power is fixed for the whole curve. Why it exists and what it's used for:
[What is MLPerf Endpoints?](../understand/what-is-mlperf-endpoints.md#normalised-by-provisioned-power).

!!! danger "Required, and authored by you"
    Every system needs one. A system without it, or with a file from which no total can be
    derived, fails `power-descriptor` and the submission is rejected. Put it at
    the top level of at least one run folder of the system; the builder places it at
    `results/<system>/system_power.json` in the bundle. Authored in
    [step 5](../workflow/author-disclosures.md#3-write-system_powerjson-for-each-system).

## Fields

The checker reads the shape below. Every field is optional on its own, but the file as a whole has
to yield a total.

| Field | Type | Description |
|---|---|---|
| `provisioned_power_w` | number | A declared total, in **watts**. If present it is used as-is and the component groups are ignored |
| `rack_power_w`, `rack_nodes`, `submitted_nodes` | number | Rack-level node scaling (§4.5.2.1): the published power of a full rack of `rack_nodes` nodes, scaled to the `submitted_nodes` you submit |
| `cpu` | group | Host CPUs |
| `accelerator` | group | GPUs, ASICs and similar |
| `compute` | group | Optional. CPU and accelerator as one figure, where the vendor publishes them that way. Replaces `cpu` and `accelerator` |
| `scale_up_network` | group | The high-bandwidth fabric between accelerators — NVLink switches, TPU ICI, UALink over Ethernet. Zero in a system with no switches |
| `scale_out_network` | group | Optional. Only for multi-node systems with a scale-out fabric. Reported, but not added to the total |
| `overhead_fraction` | number | Optional, and normally left out: the checker takes it from `cooling` in `system_desc.json` (below) |

Each **group** has a count, a rated power per unit in **watts**, and a link to a public
specification. The checker accepts the field names from rules §4.5.2, or a generic spelling on any
group:

| Group | Count | Power per unit |
|---|---|---|
| `cpu` | `num_cpu` | `tdp_per_cpu` |
| `accelerator` | `num_accelerator` | `tdp_per_accelerator` |
| `scale_up_network`, `scale_out_network` | `num_switches` | `tdp_per_switch` |
| Any group | `count` | `tdp_per_unit` |

The link goes in `link` or `public_specification`. Count what's installed, as provisioned, not the
chassis maximum.

!!! warning "Declare `cooling` in `system_desc.json`"
    The overhead fraction comes from the system description's `cooling` field, read at system
    level or from `node_types`: `0.30` if it says liquid (or water, or immersion), `0.50` if it says
    air. A system whose nodes are cooled differently gets `0.50`. Without a fraction there is no
    total, so `power-descriptor` fails. Passive cooling matches neither value; in that case state
    `overhead_fraction` here.

## How the total is computed

What the checker computes from the fields above. The power model it implements is in
[§4.5.2, Power Model][rules-4.5.2-power-model].

```
total_w  = provisioned_power_w                             if declared
         = rack_power_w × submitted_nodes / rack_nodes     else, if all three are given
         = major × (1 + overhead_fraction)                 otherwise

major    = cpu + accelerator + scale_up_network
           (compute replaces cpu + accelerator when given)

provisioned_power_kw = total_w / 1000
system_tps_per_kw    = system_tps / provisioned_power_kw
```

Scale-out networking isn't part of `major`. Rules §4.5.2 counts it inside the overhead fraction,
along with storage, power-supply overhead and cooling.

## Example

A liquid-cooled node with two CPUs, eight accelerators and one scale-up switch. Its
`system_desc.json` says the node is liquid-cooled, so the fraction is `0.30`:

```json
{
  "cpu":              { "num_cpu": 2,         "tdp_per_cpu": 350,         "link": "https://vendor.example/cpu-spec" },
  "accelerator":      { "num_accelerator": 8, "tdp_per_accelerator": 700, "link": "https://vendor.example/accel-spec" },
  "scale_up_network": { "num_switches": 1,    "tdp_per_switch": 3500,     "link": "https://vendor.example/switch-spec" }
}
```

`(700 + 5600 + 3500) × 1.30 = 12,740 W`, so 12.74 kW. A point with `system_tps` of 25,000 reports
`system_tps_per_kw` of about 1,962.

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

## Checker rules that read this file

| Rule | Checks | Severity |
|---|---|---|
| `power-descriptor` | Present for each system, and a total can be derived | Reject |
| `power-estimated` | Names the component groups left for MLCommons to fill in | Warn |
| `metric-consistency-tps-per-kw` | A stored `system_tps_per_kw` equals `system_tps / provisioned_power_kw` | Flag |

The bundle build also fails if two runs of the same system carry different `system_power.json`
contents.

??? note "Caveats"
    - **Scope.** Rules §4.5 applies normalization to Standardized CoP and CoN, makes it optional for
      RDI, and defers Serviced. The same section says every submission must include the file, and
      the checker requires it for every system regardless of division. Until that's reconciled,
      include it whatever your division. Tracked as **B11**.
    - **Pending ratification.** Tier definitions, overhead fractions, reference components and the
      metric's name and units are all marked subject to change.

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (d2d9da6) and
`mlcommons/endpoints-submission-cli@main` (a42a056, `v1.1.0.0`), 2026-10-02.*
