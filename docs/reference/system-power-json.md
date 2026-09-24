# `system_power.json`

The provisioned-power descriptor for one system. **One per system**, not per point: provisioned
power is fixed for the whole curve. Why it exists and what it's used for:
[What is MLPerf Endpoints?](../understand/what-is-mlperf-endpoints.md#normalised-by-provisioned-power).

!!! danger "Required, and authored by you"
    Every system needs one. A system without it, or with a file from which no total can be
    derived, fails `power-descriptor` and the submission is rejected. No tool writes it. Put it at
    the top level of at least one run folder of the system; the builder places it at
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
    Rules §4.5.2 lists the fields as `num_cpu`, `tdp_per_cpu`, `num_accelerator`,
    `tdp_per_accelerator`, `num_switches` and `tdp_per_switch`. The released checker reads nested
    groups with `count`, `tdp_per_unit` and `link`, as above. The checker's spelling is what gets
    validated. Tracked as **B10** in [Open questions](../help/open-questions.md).

## How the total is computed

```
major    = cpu + accelerator + scale_up_network (+ scale_out_network)
           where each group = count × tdp_per_unit

total_w  = provisioned_power_w                    if declared
         = major × (1 + overhead_fraction)        otherwise

provisioned_power_kw = total_w / 1000
system_tps_per_kw    = system_tps / provisioned_power_kw
```

Rules §4.5.2 also allows CPU and accelerator power to be declared as one combined value where
a vendor publishes them that way.

## Example

A liquid-cooled node with two CPUs, eight accelerators and one scale-up switch:

```json
{
  "cpu":              { "count": 2, "tdp_per_unit": 350,  "link": "https://vendor.example/cpu-spec" },
  "accelerator":      { "count": 8, "tdp_per_unit": 700,  "link": "https://vendor.example/accel-spec" },
  "scale_up_network": { "count": 1, "tdp_per_unit": 3500, "link": "https://vendor.example/switch-spec" },
  "overhead_fraction": 0.30
}
```

`(700 + 5600 + 3500) × 1.30 = 12,740 W`, so 12.74 kW. A point with `system_tps` of 25,000 reports
`system_tps_per_kw` of about 1,962.

## Evidence the rules accept

| Accepted | Not accepted |
|---|---|
| A spec sheet on the vendor's website | Media speculation |
| A disclosure in an academic or technical conference or publication | Third-party social media |
| A statement to press at a keynote or on an earnings call | Industry analyst blogs, videos and reports |
| Other public statements officially sanctioned by the submitting organisation | Anything not stated by a representative of the submitting organisation |

A component running **below its rated TDP** needs public evidence of the lower rating, such as an
alternative SKU or a listed operating mode, plus evidence of the cap reproducible by an audit,
for example `nvidia-smi` or `rocm-smi` output. Custom and low-volume SKUs carry the same burden.

## Partially populated systems

| Configuration | How to state its power |
|---|---|
| `Y` of `N` identical nodes in a rack | Published power for that configuration, or `P_rack × Y / N` from the published rack figure |
| A node with some accelerator slots empty | Published power for that configuration, or the component groups with only the installed parts counted. **Not** linear scaling: host CPU, memory, NICs and PSU overhead don't shrink with accelerator count |
| A node or rack under a TDP cap | Published power for that configuration, or the groups with the capped values, plus the below-TDP evidence above |

Where a published figure is a range, use the upper bound: a rack rated 132–140 kW is 140 kW.

## What happens to values you leave out

MLCommons fills them in, using the architecture, core count and other details in your system
description to pick a proxy: Intel and AMD parts for x86 CPUs, Arm AGI and Neoverse for Arm,
Broadcom Tomahawk for Ethernet switching, and public NVLink power figures for NVLink. The rules describe these estimates as deliberately
conservative. The published result is then tagged **"MLC Estimated Power"**.

If you disagree with an estimate, you have to point to a better public source or publish the figure
yourself. Non-public information is considered only at MLCommons's discretion.

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

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
