# 8. After you submit

> Produces: a finalized, published result, as long as you keep responding to reviewers.

!!! note "Before you begin"
    - Completed [7. Submit](submit.md)
    - Status is `REVIEW_PENDING`
    - **Someone is on the hook to respond within 3 business days for the next six weeks**

Don't treat this as an afterthought. It's easier to lose a submission here by not replying to
reviewers than by failing a technical check at Week 0.

## What happens, and when

| Phase | Window | Your obligation |
|---|---|---|
| Automated compliance | Week 0 | Nothing — but a failure rejects the submission outright |
| Peer review | Weeks 1–3 | Respond to every objection within **3 business days** |
| Objection resolution | Weeks 4–6 | Provide a fix or formal resolution within **3 business days** |
| Dispute resolution | From Week 6, ~5 weeks | File a written statement within **10 business days** of notification |

Timelines anchor to the cohort your submission first appears in, not the day you uploaded.

## The response clock

!!! danger "Silence withdraws your submission"
    Penalties accrue automatically from the day an objection is filed. The review chair enforces
    them without a motion, and they're **cumulative and non-reversible**. Responding after a
    threshold does not undo the penalty already incurred.

    | Business days without your response | Penalty |
    |---|---|
    | 3 | Finalization delayed by **1 cohort** |
    | 6 | Delayed by **2 cohorts** |
    | 10 | Submission is **withdrawn** |

Business-day counting accounts for local public holidays in your primary operating jurisdiction.

Your initial response must either **acknowledge the issue and include a schedule for resolution**
(this can extend into the objection resolution window), or **contest it with a counter-argument
and supporting evidence**. A holding reply with no schedule and no argument is not a response.

After you respond, the objector has 2 business days to accept, retract, or carry the objection
forward. In the resolution window, objector silence for 3 business days is treated as
acknowledgment and the objection is automatically retracted.

## What reviewers are looking at

A peer review covers, at minimum: whether results are in a reasonable range for the hardware and
software, whether your reproducibility instructions are clear and followable, the benchmark
methodology, and the content of your system description JSON.

Human reviewers focus on what automation cannot check:

- Whether the curve shape is physically plausible: throughput rising with concurrency to
  saturation, then levelling off
- Whether metric distributions look manipulated, such as suspiciously uniform TTFT across very
  different concurrency levels
- Whether the TTFT-triggering fragment is a genuine part of the response rather than leading
  whitespace emitted to stop the clock
- Whether content is duplicated across response fields or padded to inflate token counts
- Whether warmup used performance-dataset samples. Reviewers may cross-check your retained logs
- For the Offline point, whether reordering stayed inside one pass over the dataset. Batches made
  of repeated copies of the same sample point to sorting across passes
- Whether the system description matches what actually ran
- If you used speculative decoding, whether the disclosed drafter matches the approved list
- Cross-submission consistency for the same hardware platform

## Objection types and what they cost you

| Type | Severity |
|---|---|
| Compliance failure | High — may require withdrawal |
| Methodology | High |
| Reproducibility | High, but must exceed the variability margin to be actionable |
| Validity of results | High |
| Division rules | Medium — usually **reclassification**, not withdrawal |
| Availability | Medium — usually reclassification |
| Suspect or incomprehensible results | Medium — triggers investigation |
| Cosmetic | Low — does not block finalization |

Reproducibility margins: up to **10%** variability when an independent party re-runs during review;
within **5%** when re-running on the exact same system. These apply to `system_tps` and
`tps_per_user` only.

!!! warning "Latency cannot carry an objection on its own"
    The throughput margins do **not** apply to `ttft_p90_ms` or any latency percentile. A fixed
    percentage band is not a sound test for a tail statistic. Until the working group ratifies a
    method, a reproducibility objection may not rest on latency alone. Latency evidence can support
    an objection based primarily on throughput or accuracy.

**Accuracy always has to pass.** No margin, at any stage.

## What you may and may not change

**During peer review**, after compliance checks have passed, you may update run and submission
metadata **only when the review committee requests it**, where insufficient information, code or
instructions were provided, or a material flaw must be rectified. Any improvement to performance
metrics must be justified and explained to the committee.

Freely correctable: documentation, READMEs, system description metadata, source and configuration
files that do not match the actual run settings, software version information, calibration
write-ups.

!!! danger "Results may not be changed during review"
    If a measurement point is wrong, your options are to **withdraw that point**
    (`submissions remove-run`) or withdraw the whole submission. Withdrawn points do not count
    toward the minimum point count, and the shortfall cannot be repaired by adding a replacement.

**After finalization**, corrections to published results are not permitted at all. An error means
invalidating the affected points or the submission. Non-result corrections need review-chair
approval and publish with a change log.

## Finalization and publication

Once every objection is resolved or retracted, the *peer review pending* tag is removed and the
result is finalized. If everything resolves before the end of Week 3 you qualify for **early
finalization** and do not wait for the resolution window to close.

The result then publishes in the next cohort for which it clears the one-business-day alignment
window, or on your embargo date if you declared one. Published results carry: result ID, submitter
and system description, benchmark model, division, publication status, the step-function Pareto
curve, and `system_tps`, `tps_per_user` and `ttft_p90_ms` at each point. A dedicated Offline run is
plotted as your throughput ceiling and labelled **Offline**; an elected one is a label on your
`C_max` point. If MLCommons had to fill in any of your power figures, the result is tagged
**"MLC Estimated Power"**.

## Your obligations after publication

- **Retain the system.** Keep the benchmarked system and its configuration available for a
  possible audit until the result settles, which is the later of the next audit vote or **90
  days** after finalization.
- **Keep it as submitted if you're nominated for audit.** Committee members can nominate your
  result for the quarterly audit vote during the **4 weeks after publication**. From the moment
  it's nominated you must keep the system in its submitted configuration, even if that runs past
  the 90 days. If the vote passes you over, the obligation ends. If you're selected, or drawn in
  the random audit, it lasts until the audit is complete. See
  [Audits](../understand/how-submission-works.md#audits).
- **Keep a CoN endpoint reachable.** Standardized CoN submissions must keep the endpoint URL
  accessible for at least **90 days** after publication, with a point of contact for access requests.
- **Honour a Preview commitment.** 180 days from first publication to achieve Available status and
  re-submit. One extension of up to 60 days may be granted, requested at least 30 days before
  expiry. See [Publication status](../rules/publication-status.md).
- **Use the right name.** Unqualified "MLPerf Endpoints" means Standardized. Serviced and RDI
  results must use their qualified names, and Preview results must carry the "Preview" qualifier.

## Verify

```bash
endpoints-submission-cli submissions get --submission-id <submission-id>
```

You are done when status reads `FINALIZED`, and published when it reads `PUBLISHED`. Full state
list: [Submission states](../reference/submission-states.md).

## Next

Nothing. You've finished the workflow. If something went wrong, see
[Troubleshooting](../help/troubleshooting.md) or [Getting support](../help/support.md).
