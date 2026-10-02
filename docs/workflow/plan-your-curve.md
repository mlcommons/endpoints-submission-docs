# 3. Plan your Pareto curve

> Produces: the list of concurrency levels you'll run, checked as valid before you spend any
> accelerator time.

!!! note "Before you begin"
    - Completed [2. Install the tools](install-tools.md)
    - You know roughly the highest concurrency your system serves usefully

!!! danger "Do this before you run anything"
    Region boundaries are calculated from your own minimum and maximum concurrency, so which
    concurrency levels count as valid depends on a number you pick. Running your points first and
    computing regions afterwards is the most common way to discover you have missed a required
    region. Each missing point means another 1,200-second run plus warmup.

## What you'll do

- Choose `C_max`, your Maximum Supported Concurrency
- Compute your four region boundaries with `submission-checker regions`
- Pick at least 7 concurrency levels that satisfy the `1 + 3 + 3` coverage rule
- Decide how you'll meet the Offline requirement: a dedicated run, or electing your `C_max` point
- Sanity-check the count against the 32-point cap

## The coverage rule

A non-agentic submission needs a **minimum of 8** and a **maximum of 32** measurement points,
structured `1 + 3 + 3 + 1`:

| Points | Placement |
|---|---|
| 1 mandatory | In the **Ultra Low Concurrency** region — concurrency 1 to 32 inclusive, fixed for every submission |
| 3 mandatory | One each in **Low**, **Medium** and **High** Concurrency |
| 3 submitter's choice | Anywhere in those same three concurrency regions |
| 1 mandatory **Offline** | Every query available at once rather than a fixed concurrency. See [step 6](#6-decide-how-to-meet-the-offline-requirement) |

The minimum drops to **7** in two cases: you elect your `C_max` point as the Offline result, so
there's no separate Offline run, or the benchmark is **agentic**. Agentic benchmarks have no
Offline point at all, and including one is an error.

There are **no spacing requirements**. Cluster points around an inflection, highlight a sweet spot,
or spread them evenly. All three discretionary points can go in one region. The Offline point
counts toward the 32-point cap.

## Steps

### 1. Choose `C_max`

`C_max` is the highest concurrency at which you choose to benchmark. It defines the upper bound of
the High Concurrency region and therefore the extent of your published curve.

There is no compliance test forcing a particular value, but the choice is consequential and visible:

- `C_max` must be **greater than 32**.
- `C_max` should be **much greater** than your minimum concurrency.
- A larger `C_max` shows more of your scaling story but stretches the High Concurrency region.

!!! warning "`C_max` ≤ 33 needs working-group approval"
    At that point all three concurrency regions collapse to roughly one level each. You must notify
    the working group with written justification, and they may request more information before
    accepting the submission.

### 2. Understand what sets `C_min`

The Ultra Low Concurrency region is fixed at 1–32 for everyone. `C_min`, the value the region
calculation uses, is the lowest concurrency level in your own submission, and has to fall in 1–32.

!!! note "Derived, not declared"
    In v1.0 the checker **derives** `C_min` from your submitted points rather than reading a declared
    value. Practical consequence: your lowest point silently changes every other region boundary. If
    you later drop your lowest point, the boundaries move and points that were legal may no longer
    be. Decide your lowest point early and keep it.

You're encouraged but not required to measure at **concurrency 1**, the single-user
baseline, and the number most often quoted in comparisons.

### 3. Compute the boundaries

Beyond Ultra Low Concurrency, the space up to `C_max` is divided into **three equal regions in
log-2 space**:

```
I = log2(C_max - C_min) / 3

Low Concurrency    : C_min + 1          → round(C_min + 2^I)
Medium Concurrency : low_end + 1        → round(C_min + 2^(2I))
High Concurrency   : med_end + 1        → C_max
```

Boundaries round half-to-even (banker's rounding). Log spacing is used because the difference
between concurrency 1 and 10 matters far more than between 1000 and 1010.

Don't calculate this by hand. Use the tool:

```bash
submission-checker regions --max-concurrency 1024 --min-concurrency 16
```

<div class="result" markdown>

For `C_min = 16`, `C_max = 1024`:

| Region | Range |
|---|---|
| Ultra Low Concurrency | 1 – 16 |
| Low Concurrency | 17 – 26 |
| Medium Concurrency | 27 – 116 |
| High Concurrency | 117 – 1024 |
| 10% margin | 1025 – 1127 |

</div>

More pre-computed combinations are in [Metrics and regions](../reference/metrics-and-regions.md).

### 4. Choose your points

Worked examples straight from the rules:

| System | `C_min` | `C_max` | A valid set of 7 fixed-concurrency points |
|---|---|---|---|
| Large scale | 32 | 8,192 | `{32, 40, 200, 500, 1000, 2000, 4096}` |
| Mid-range | 16 | 1,024 | `{16, 24, 64, 96, 128, 256, 1000}` |
| Smaller | 1 | 256 | `{1, 4, 16, 32, 64, 128, 256}` |

These are the rules' own examples, and they predate the Offline point. Add a dedicated Offline run
on top, or elect the `C_max` point. The first two sets also stop short of their `C_max` (8,192 and
1,024), which is fine for region coverage but causes trouble at [step 6](#6-decide-how-to-meet-the-offline-requirement).

### 5. Note the 10% margin

The High Concurrency region carries a **10% margin** above `C_max`, extending the valid upper bound
to `ceil(1.10 × C_max)`.

!!! warning "The margin does not satisfy coverage"
    A point in the margin is valid, but the margin is its own region. It does **not** count as your
    required High Concurrency point. The margin also no longer serves its original purpose: it
    existed to allow adding points after submission, and that window has been removed. See
    [step 7](submit.md#3-points-are-fixed-at-creation).

### 6. Decide how to meet the Offline requirement

Skip this step if your benchmark is agentic.

For every other benchmark you need one Offline result, and there are two ways to supply it.

Either way, **put one of your points at exactly `C_max`**. Both options are defined relative to "the
`C_max` point", and the rules' own example sets above don't have one.

**Option 1: a dedicated Offline run.** A separate run in which the client makes the whole
performance dataset available at once and the system drains it as fast as it can. This is your
eighth point. Two constraints tie it to your `C_max` point:

| Quantity | Must hold |
|---|---|
| Throughput | `system_tps(Offline) ≥ 0.98 × system_tps(C_max point)` |
| Concurrency | `concurrency(Offline) ≥ C_max` |

The 2% is there to absorb run-to-run noise. It isn't a target. An Offline run that comes in below
your `C_max` point usually means the Offline run didn't saturate the system. A failure is flagged
for reviewers rather than rejected outright. Note that the checker can only compare throughput when
a point sits at exactly `C_max`. Without one it skips that half of the check without saying so.

The Offline point's `concurrency` isn't yours to choose: it's the number of queries in one pass over
the performance dataset. It doesn't count toward any region, and it can't serve as your `C_max`
point.

**Option 2: elect your `C_max` point.** Declare that your `C_max` point already is the highest
throughput your system reaches. That point stays an ordinary fixed-concurrency point and is also
reported as the Offline result. There's no second run, the 2% margin doesn't apply, and your
minimum is 7 points. You're giving up whatever extra throughput a dedicated run might have shown,
which is why the rules take your word for it. The checker rejects an `elected` declaration on any
point whose concurrency isn't your declared `C_max`.

!!! tip "Which to pick"
    If you expect an unpaced run to beat your `C_max` point, run Option 1, since that's the number
    that gets plotted as your throughput ceiling. If your `C_max` point is already saturating the
    system, Option 2 saves a full run and its accuracy validation.

!!! question "If `C_max` is larger than your dataset"
    A dedicated Offline run reports the dataset size as its concurrency, and that has to be at least
    `C_max`. With a `C_max` above the dataset size you can't satisfy both. The working group hasn't
    decided what happens in that case. If it applies to you, ask before you run, or use Option 2.
    Tracked as **C7** in [Open questions](../help/open-questions.md).

## Verify

Write your planned levels down and check each one against the computed boundaries before running.
A practical check: for the set you chose, confirm you can name which region each point satisfies,
and that Low, Medium and High each have at least one.

Then confirm your plan against the cap:

- At least 8 points, or 7 if you're electing `C_max` or the benchmark is agentic? At most 32,
  Offline included?
- At least one point at concurrency ≤ 32?
- `C_max` > 32, and one point at exactly `C_max`?
- For a dedicated Offline run: is your performance dataset at least `C_max` queries?
- Can you name which five points will carry accuracy results (four for agentic)?

!!! tip "Budget a spare"
    Withdrawn points do **not** count toward the minimum, and the shortfall cannot be repaired by
    adding a replacement point later. Planning one or two points more than the minimum buys you the
    ability to lose one during review without losing the submission.

## Next

→ [4. Run the measurement points](run-the-points.md)
