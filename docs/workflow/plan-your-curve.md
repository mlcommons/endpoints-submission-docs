# 3. Plan your Pareto frontier

> Produces: the list of concurrency levels you'll run, checked as valid before you spend any
> accelerator time.

!!! note "Before you begin"
    - Completed [2. Install the tools](install-tools.md)
    - You know roughly the highest concurrency your system serves usefully

!!! warning "Do this before you run anything"
    Region boundaries are calculated from your own minimum and maximum concurrency, so which
    concurrency levels count as valid depends on a number you pick. Running your points first and
    computing regions afterwards is the most common way to discover you have missed a required
    region.

## What you'll do

- Choose `C_max`, your Maximum Supported Concurrency
- Compute your region boundaries with the checker's `regions` command
- Pick at least 7 concurrency levels that satisfy the [`1 + 3 + 3` coverage rule][rules-5.3]
- Decide how you'll meet the Offline requirement: a dedicated run, or electing your `C_max` point
- Sanity-check the count against the 32-point cap

## The coverage rule

A non-agentic submission needs at least 8 and at most 32 measurement points, structured `1 + 3 + 3 +
1`. The minimum is 7 if you elect your `C_max` point as the Offline result, or if the benchmark is
agentic. Where each point has to go, and the freedom you have in placing the rest, is in [§5.3 of
the rules][rules-5.3]. All seven concurrency points must be distinct. The cap is in
[§5.6][rules-5.6], and the Offline point counts toward it.

## Steps

### 1. Choose `C_max`

`C_max` is the highest concurrency you choose to benchmark. It sets the top of the High Concurrency
region, so it's the extent of your published frontier. It must be **greater than 32**. Beyond that the
choice is yours; the rules explain what to weigh under [Maximum Supported Concurrency in
§5.4][rules-5.4-cmax].

!!! warning "`C_max` ≤ 33 needs working-group approval"
    At that point all three concurrency regions collapse to roughly one level each. You must notify
    the working group with written justification, and they may request more information before
    accepting the submission. See Boundary Edge Cases under [Concurrency Regions in
    §5.4][rules-5.4-concurrency].

### 2. Understand what sets `C_min`

`C_min` is the top of the [Ultra Low Concurrency region][rules-5.4-ultra-low], which runs from 1
to `C_min`, and it has to be in 1–32. You set it one of two ways:

- **By default** it's your lowest fixed-concurrency point, and that point is your mandatory Ultra Low
  point.
- **With `c_min_override`** in `system_desc.json` you declare it. A point at that concurrency must
  exist, but any of your points in 1–`C_min` can be the mandatory Ultra Low point, and the others
  there count as submitter's-choice points.

The override rules (type, range, the same value on every point) are under [Maximum Supported
Concurrency in §5.4][rules-5.4-cmax].

!!! note "The override needs checker `v1.5.1.0` or later"
    Earlier releases ignore `c_min_override` and take `C_min` from your lowest point, so their
    boundaries and coverage results won't match what you declared.

!!! warning "`C_min` moves every other boundary"
    Without an override, dropping your lowest point moves every region boundary, and points that
    were legal may no longer be. With one, you can't add, remove or change it once peer review
    starts ([Submission Rules §8.1][srules-8.1]). Decide `C_min` before you run.

### 3. Compute the boundaries

Beyond Ultra Low Concurrency, the space up to `C_max` is divided into three equal regions in log-2
space. The reference algorithm is in [§5.5 of the rules][rules-5.5]. To calculate it easily, use the
tool:

```bash
python -c "from submission_checker.cli import main; main()" regions --max-concurrency 1024 --min-concurrency 16
```

<div class="result" markdown>

For `C_min = 16`, `C_max = 1024`:

| Region | Range |
|---|---|
| Ultra Low Concurrency (1 to `C_min`) | 1 – 16 |
| Low Concurrency | 17 – 26 |
| Medium Concurrency | 27 – 117 |
| High Concurrency | 118 – 1024 |
| Margin (10% above `C_max`) | 1025 – 1127 |

</div>

The tool labels the Ultra Low Concurrency band "Low Latency".

Pre-computed boundaries for common combinations are in [Appendix B of the rules][rules-appendix-b].

### 4. Choose your points

The rules give a worked 7-point set for three system sizes, under [Concurrency Regions in
§5.4][rules-5.4-concurrency]. Use them as a starting shape, with two caveats. They predate the
Offline point, so add a dedicated Offline run on top or elect the `C_max` point. And the large-scale
and mid-range sets stop short of their `C_max`, which is fine for region coverage but causes trouble
when you [decide how to meet the Offline
requirement](#6-decide-how-to-meet-the-offline-requirement).

### 5. Note the 10% margin

The High Concurrency region carries a **10% margin** above `C_max`, extending the valid upper bound
to `ceil(1.10 × C_max)` ([Concurrency Regions in §5.4][rules-5.4-concurrency]).

!!! warning "The margin does not satisfy coverage"
    A point in the margin is valid, but the margin is its own region. It does **not** count as your
    required High Concurrency point. The margin also no longer serves its original purpose: it
    existed to allow adding points after submission, and that window has been removed. See [step
    8](submit.md#points-fixed-at-creation).

### 6. Decide how to meet the Offline requirement

Skip this step if your benchmark is agentic.

For every other benchmark you need one Offline result. [§5.7.2 of the rules][rules-5.7.2] gives two
ways to supply it: a **dedicated Offline run**, or **electing your `C_max` point** as the Offline
result.

## Verify

Write your planned levels down and name the region each one falls in, using the boundaries you
computed above. Then check the plan against the minimum submission requirements in [§5.3 of the
rules][rules-5.3] and the point-count, region-coverage and Offline checks in [§9.1][rules-9.1].

!!! tip "Budget a spare"
    Withdrawn points do **not** count toward the minimum, and the shortfall cannot be repaired by
    adding a replacement point later. Planning one or two points more than the minimum buys you the
    ability to lose one during review without losing the submission.

## Next

→ [4. Run the measurement points](run-the-points.md)
