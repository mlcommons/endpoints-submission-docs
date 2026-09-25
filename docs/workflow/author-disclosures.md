# 5. Author the disclosure files

> Produces: `system_desc.json` and `point.yaml` in every run folder, `system_power.json` once per
> system, plus the shared `src/` and `docs/` content.

!!! note "Before you begin"
    - Completed [4. Run the measurement points](run-the-points.md)
    - You have one run folder per point
    - Your disclosure content is cleared for publication

!!! danger "The reference client doesn't write these files"
    `system_desc.json`, `point.yaml` and `system_power.json` are yours to supply, dropped into run
    folders before upload. The reference client does not write them, and only `system_desc.json` has
    a capture tool (see [below](#2-write-system_descjson-for-each-point)). The submission CLI copies
    `point.yaml` into the bundle **exactly as written**. It doesn't derive it from `config.yaml` and
    does not fill in missing fields. Whatever you write is what gets submitted and what gets
    checked.

## What you'll do

- Write `point.yaml` for every measurement point
- Write `system_desc.json` for every measurement point
- Write `system_power.json` once per system
- Write the shared `src/<implementation>/README.md`
- Write the shared `docs/` disclosure content

## Where the files go

Each run folder needs `system_desc.json` and `point.yaml` at its top level. `system_power.json` goes
at the top level of at least one run folder per system:

```
<run-folder>/
├── system_desc.json                  # §8.2   — you supply (mlperf-sysinfo can capture it)
├── point.yaml                        # §8.3   — you author
├── system_power.json                 # §4.5.2 — you author, once per system
├── config.yaml                       # client — optional as of v1.0
├── performance/
│   └── result_summary.json           # client — performance phase
├── accuracy/
│   └── accuracy_results.json         # client — accuracy phase
├── src/
│   └── <implementation>/             # merged into the bundle's shared src/
│       └── README.md                 # you author — required
└── documentation/                    # merged into the bundle's shared docs/
    ├── software_disclosure.md        # you author
    └── calibration.adoc              # you author — only if you transformed weights
```

## Steps

### 1. Write `point.yaml` for each point

This is the measurement-point disclosure the checker validates. The fields it must declare, and what
each one means, are defined in [§8.3 of the rules][rules-8.3]. Work through that table field by
field; the [`point.yaml` reference](../reference/point-yaml.md) notes where the checker's accepted
values and field names differ from it.

Two fields need a decision rather than a lookup: `offline` and `dataset_type`.

!!! warning "Exactly one point carries `offline`"
    For a non-agentic benchmark, one point has to declare `offline: dedicated` or `offline:
    elected`. `elected` is only accepted on the point whose concurrency equals your declared
    `C_max`. The checker only **warns** when no point declares `offline`, because it can't yet tell
    an agentic benchmark from a single-turn one, but the rules reject a non-agentic submission
    without one.

    A dedicated Offline run sets `concurrency` to the size of the performance dataset and counts
    toward no region ([§5.7.1][rules-5.7.1], [§5.7.2][rules-5.7.2]). The rules don't say what its
    `region` field should hold. `submitters_choice` passes the checker without a placement warning.
    Tracked as **C7** in [Open questions](../help/open-questions.md).

!!! warning "`dataset_type` does real work"
    The bundle builder must know whether a run is an accuracy or a performance run and **will not
    guess**. It reads `datasets[].type` from `config.yaml` first, then falls back to `point.yaml`'s
    `dataset_type`, but only when that value is exactly `Accuracy` or `Performance`. A value of
    `Accuracy + Performance` describes the *dataset*, not the run, and the build fails naming the
    run. The strictness is intentional: defaulting to "performance" used to file accuracy runs in
    the wrong place and quietly drop their results.

### 2. Write `system_desc.json` for each point

The hardware and software description. Since policies PR #119 there's no per-system file: **every
Pareto point carries its own copy**, and the checker verifies all points of a curve describe the
same system.

The fields are defined in [§8.2 of the rules][rules-8.2]. Either capture the file with
[`mlperf-sysinfo`](https://docs.mlcommons.org/mlperf-sysinfo/), which reads the machine under test
using its `endpoints` profile, or copy the template in [§8.2.1][rules-8.2.1] and fill it in. Checker
behaviour and known gaps between the rules and the tooling: [`system_desc.json`
reference](../reference/system-desc-json.md).

!!! tip "`tps_utilization` is computed, not chosen"
    It is `reported_system_tps / max(reported_system_tps across the curve)`. The checker recomputes
    it against your own curve. You cannot fill this in until every point has run.

### 3. Write `system_power.json` for each system

This declares the system's provisioned power, which v1.0 divides throughput by. It's per **system**,
not per point, because provisioned power is fixed for the whole curve. The builder lifts it from
your run folders to `results/<system>/system_power.json` in the bundle.

The [power model][rules-4.5.2-power-model] and the ways to fill the file are in [§4.5.2 of the
rules][rules-4.5.2]: either component counts and TDPs, each linked to a public spec sheet, with the
overhead fraction for your cooling, or a single published provisioned-power figure. The component
form looks like this:

```json
{
  "cpu":              { "count": 2, "tdp_per_unit": 350,  "link": "https://…" },
  "accelerator":      { "count": 8, "tdp_per_unit": 700,  "link": "https://…" },
  "scale_up_network": { "count": 1, "tdp_per_unit": 3500, "link": "https://…" },
  "overhead_fraction": 0.30
}
```

Count only what's actually installed. Partly populated systems and components run below their rated
TDP have their own rules in the same section, including the evidence you need for a cap.
Field-by-field detail and checker behaviour: [`system_power.json`
reference](../reference/system-power-json.md).

!!! warning "Keep every copy identical"
    You can put the file in more than one run folder of the same system, but every copy must have
    the same contents. Two runs of one system that disagree fail the bundle build, because one
    system can only have one provisioned power.

!!! note "Leaving a value out doesn't mean leaving the file out"
    The file is required even if you leave values out ([Component Template in
    §4.5.2][rules-component-template-system_powerjson]). The checker flags each blank component
    group `power-estimated`, MLCommons fills it in with a deliberately conservative estimate, and
    the published result is tagged **"MLC Estimated Power"**. The file still has to yield a total:
    with no `provisioned_power_w` and no group that has both a count and a TDP, the checker fails
    `power-descriptor`.

### 4. Write the shared `src/` content

`src/<implementation>/` (for example `vllm/`, `trtllm/`, `sglang/`) holds the endpoint interface
code, cluster setup and client harness, with a **required `README.md`** on how to build, launch and
reproduce a point. Like `docs/`, it's written **once** for the whole submission, not per point
([§8.1][rules-8.1]).

### 5. Write the shared `docs/` content

- `software_disclosure.md` — the software components listed in
  [§8.4][rules-8.4].
- `calibration.adoc` — required if you applied any weight transformation
  ([§3.3][rules-3.3]).
  The recipe has to be reproducible from the write-up or the scripts you provide
  ([§2.2.1][rules-2.2.1]).
- Anything else a reviewer needs to follow your setup.

!!! warning "Disclosure obligations differ by division"
    Standardized requires full hardware, software and parallelism disclosure; Serviced has its own
    list and makes full rack hardware optional. See [§2.7][rules-2.7] and, for Serviced,
    [§2.3.1][rules-2.3.1]. Summary: [Requirements you must meet](../rules/requirements.md).

## Verify

The full check is the checker, in [step 7](validate.md). Before that, confirm mechanically that
nothing is missing:

```bash
for d in run-folders/*/; do
  for f in system_desc.json point.yaml; do
    [ -f "$d$f" ] || echo "MISSING: $d$f"
  done
done
ls run-folders/*/system_power.json            # at least one per system
grep -lE '^offline: *(dedicated|elected)' run-folders/*/point.yaml   # one folder, none if agentic
```

Then confirm your `shared_src` and `shared_docs` values name directories that will exist under the
assembled submission root. A point whose pointers don't resolve is incomplete and the submission is
rejected.

## Next

→ [6. Register your runs](register-runs.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#disclosure-files).
