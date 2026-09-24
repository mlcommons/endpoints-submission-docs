# 5. Author the disclosure files

> Produces: `system_desc.json` and `point.yaml` in every run folder, `system_power.json` once per
> system, plus the shared `src/` and `docs/` content.

!!! note "Before you begin"
    - Completed [4. Run the measurement points](run-the-points.md)
    - You have one run folder per point
    - Your disclosure content is cleared for publication

!!! danger "No tool generates these files"
    `system_desc.json`, `point.yaml` and `system_power.json` are **hand-authored by you** and
    dropped into run folders before upload. The reference client does not write them. The
    submission CLI copies `point.yaml` into the bundle **exactly as written**. It doesn't derive it
    from `config.yaml` and does not fill in missing fields. Whatever you write is what gets
    submitted and what gets checked.

## What you'll do

- Write `point.yaml` for every measurement point
- Write `system_desc.json` for every measurement point
- Write `system_power.json` once per system
- Write the shared `src/<implementation>/README.md`
- Write the shared `docs/` disclosure content

## Where the files go

Each run folder needs `system_desc.json` and `point.yaml` at its top level. `system_power.json`
goes at the top level of at least one run folder per system:

```
<run-folder>/
├── system_desc.json                  # §8.2 — you author this
├── point.yaml                        # §8.3 — you author this
├── system_power.json                 # §4.5.2 — you author this, per system
├── performance/result_summary.json   # written by the client
├── accuracy/accuracy_results.json    # written by the client
├── config.yaml                       # written by the client (optional as of v1.0)
├── src/<implementation>/             # merged into the bundle's shared src/
└── documentation/                    # merged into the bundle's shared docs/
```

## Steps

### 1. Write `point.yaml` for each point

This is the measurement-point disclosure the checker validates. The fields it must declare, and
what each one means, are defined in [§8.3 of the rules][rules-8.3]. Work through that table field
by field; the [`point.yaml` reference](../reference/point-yaml.md) notes where the checker's
accepted values and field names differ from it.

Two fields need a decision rather than a lookup: `offline` and `dataset_type`.

!!! warning "Exactly one point carries `offline`"
    For a non-agentic benchmark, one point has to declare `offline: dedicated` or
    `offline: elected`. `elected` is only accepted on the point whose concurrency equals your
    declared `C_max`. The checker only **warns** when no point declares `offline`, because it
    can't yet tell an agentic benchmark from a single-turn one, but the rules reject a non-agentic
    submission without one.

    A dedicated Offline run sets `concurrency` to the size of the performance dataset and counts
    toward no region. The rules don't say what its `region` field should hold. `submitters_choice`
    passes the checker without a placement warning. Tracked as **C7** in
    [Open questions](../help/open-questions.md).

!!! warning "`dataset_type` does real work"
    The bundle builder must know whether a run is an accuracy or a performance run and **will not
    guess**. It reads `datasets[].type` from `config.yaml` first, then falls back to `point.yaml`'s
    `dataset_type`, but only when that value is exactly `Accuracy` or `Performance`. A value of
    `Accuracy + Performance` describes the *dataset*, not the run, and the build fails naming the
    run. The strictness is intentional: defaulting to "performance" used to file accuracy runs in
    the wrong place and quietly drop their results.

### 2. Write `system_desc.json` for each point

The hardware and software description. Since policies PR #119 there's no per-system file:
**every Pareto point carries its own copy**, and the checker verifies all points of a curve describe
the same system.

The fields are defined in [§8.2 of the rules][rules-8.2], and a copyable template is in
[§8.2.1][rules-8.2.1]. Checker behaviour and known gaps between the rules and the tooling:
[`system_desc.json` reference](../reference/system-desc-json.md).

!!! tip "`tps_utilization` is computed, not chosen"
    It is `reported_system_tps / max(reported_system_tps across the curve)`. The checker recomputes
    it against your own curve. You cannot fill this in until every point has run.

### 3. Write `system_power.json` for each system

This declares the system's provisioned power, which v1.0 divides throughput by. It's per
**system**, not per point, because provisioned power is fixed for the whole curve. The builder
lifts it from your run folders to `results/<system>/system_power.json` in the bundle.

Two ways to fill it:

- **From components.** Count and rated TDP for the CPUs, accelerators and scale-up switches, each
  with a link to a public spec sheet, plus the overhead fraction for your cooling: `0.30` liquid,
  `0.50` air.
- **Directly.** A single provisioned-power figure, if you have published documentation for the
  system as provisioned. For a range, use the upper bound.

```json
{
  "cpu":              { "count": 2, "tdp_per_unit": 350,  "link": "https://…" },
  "accelerator":      { "count": 8, "tdp_per_unit": 700,  "link": "https://…" },
  "scale_up_network": { "count": 1, "tdp_per_unit": 3500, "link": "https://…" },
  "overhead_fraction": 0.30
}
```

Count only what's actually installed. A half-populated node uses the populated counts, not the
chassis maximum. If a component runs below its rated TDP, you need public evidence of the lower
rating plus reproducible evidence of the cap, such as `nvidia-smi` or `rocm-smi` output.

Field-by-field detail, the power model, and the partial-rack rules:
[`system_power.json` reference](../reference/system-power-json.md).

!!! warning "Keep every copy identical"
    You can put the file in more than one run folder of the same system, but every copy must have
    the same contents. Two runs of one system that disagree fail the bundle build, because one
    system can only have one provisioned power.

!!! note "Leaving a value out doesn't mean leaving the file out"
    The file is required, and it has to state enough for a total to be derived. Any component group
    you leave blank is flagged `power-estimated`, and MLCommons fills it in from its own estimate,
    which the rules describe as deliberately conservative. The published result is then tagged
    **"MLC Estimated Power"**.

### 4. Write the shared `src/` content

`src/<implementation>/` (for example `vllm/`, `trtllm/`, `sglang/`) holds the endpoint interface
code, infrastructure and cluster setup, and client harness. **A `README.md` is required** in each
implementation directory, explaining how to build and launch the system under test and reproduce a
point.

This content is shared across the whole submission and written **once**. It isn't duplicated per
Pareto point. Adding or withdrawing a point must not require any change under `src/` or `docs/`.

### 5. Write the shared `docs/` content

- `software_disclosure.md` — serving framework with version and commit or release tag, accelerator
  compute library and build, driver version, operating system.
- `calibration.adoc` — required if you applied any weight transformation. Either describe the recipe
  in enough detail for an external team to reproduce it, or provide the scripts that implement it.
- Anything else a reviewer needs to follow your setup.

!!! warning "Disclosure obligations differ by division"
    Standardized requires full hardware, software and parallelism disclosure. Serviced requires the
    advertised model name and version, endpoint URL, pricing model and rates, and rate limits, but
    full rack hardware disclosure is optional. See
    [Requirements you must meet](../rules/requirements.md).

## Verify

The fastest check is the checker itself, which is [step 6](validate.md). Before that, confirm
mechanically that nothing is missing:

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
assembled submission root. A point whose pointers don't resolve is incomplete and the submission
is rejected.

## Next

→ [6. Validate locally](validate.md)

Problems? See [Troubleshooting](../help/troubleshooting.md#disclosure-files).
