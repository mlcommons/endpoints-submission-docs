# 9. After you submit

> Produces: a finalized, published result, as long as you keep responding to reviewers.

!!! note "Before you begin"
    - Completed [8. Submit](submit.md)
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

[![MLPerf Endpoints submission and review cycle, tracing four example submissions through each
publication mode](../assets/review-cycle.svg)](../assets/review-cycle.svg)

*Four example submissions, one per publication mode, from [§4.5 of the Submission
Rules][srules-4.5]. Select the image to open it full size.*

Timelines anchor to the cohort your submission first appears in, not the day you uploaded. The full
schedule is in [§6.5 of the Submission Rules][srules-6.5], and the dispute deadlines in
[§9.2][srules-9.2].

## The response clock

!!! danger "Silence withdraws your submission"
    Penalties start automatically once an objection is filed and don't reverse if you reply late:
    finalization slips one cohort after 3 business days without a response, two after 6, and the
    submission is **withdrawn** after 10. The schedule, and how business days are counted, is in
    [Submission Rules §6.3][srules-6.3].

A reply only counts if it either acknowledges the issue with a schedule for resolution or contests
it with evidence ([§6.3][srules-6.3]). A holding reply with no schedule and no argument is not a
response. What the objector owes you in return is in [§6.4][srules-6.4].

## What reviewers are looking at

The minimum scope of a review is in [Submission Rules §2.6][srules-scope-of-review]. Human reviewers
concentrate on what automation can't check, listed in [§9.2 of the rules][rules-9.2]: curve shape,
suspicious metric distributions, TTFT and token-count gaming, warmup data, Offline pass boundaries,
and whether your system description matches what ran.

## Objection types and what they cost you

Every objection has a type, and the type sets its severity ([Submission Rules §6.8][srules-6.8]).
Division and availability objections usually end in **reclassification** rather than withdrawal, and
cosmetic ones don't block finalization.

A reproducibility objection has to show a deviation beyond the throughput margins in
[Reproducibility Expectations][srules-reproducibility-expectations]: 10% for an independent re-run,
5% on the exact same system.

!!! warning "Latency cannot carry an objection on its own"
    The margins don't apply to `ttft_p90_ms` or any other latency percentile, and until the working
    group ratifies a method, a reproducibility objection can't rest on latency alone (same section).

**Accuracy always has to pass.** No margin, at any stage.

## What you may and may not change

During peer review you can update run and submission metadata only when the review committee asks
([Updating submissions during peer review][srules-updating-submissions-during-peer-review]).
Non-result content such as documentation, system description metadata and configs that don't match
the run can be corrected; results can't ([Submission Rules §8.1][srules-8.1]).

!!! danger "Results may not be changed during review"
    If a measurement point is wrong, your options are to **withdraw that point** (`submissions
    remove-run`) or withdraw the whole submission. Withdrawn points do not count toward the minimum
    point count. §8.1 says that may mean submitting additional points, but there's no way to add a
    point to a submission ([step 8](submit.md#points-fixed-at-creation)), so in practice the
    shortfall can't be repaired.

After finalization, a wrong result means invalidating the affected points or the submission, and
non-result corrections need review-chair approval (same section).

## Finalization and publication

Once every objection is resolved or retracted, the result is finalized, early if that happens before
the end of Week 3 ([§6.3][srules-6.3]). It's queued for the next available cohort
([§6.3][srules-6.3]), or held until your embargo date if you declared one ([§4.2][srules-4.2]).

What the published entry shows is listed in [Submission Rules §7.6][srules-7.6], and how the Offline
point is plotted in [§5.7.3 of the rules][rules-5.7.3]. If MLCommons had to fill in any of your
power figures, the result is tagged **"MLC Estimated Power"**.

## Your obligations after publication

- **Retain the system** for a possible audit until the result settles: the later of the next audit
  vote or **90 days** after finalization
  ([Scope and Standing for Late Concerns][srules-scope-and-standing-for-late-concerns]).
- **Keep it as submitted if you're nominated for audit.** Nominations are open for 4 weeks after
  publication, and from nomination the system has to stay in its submitted configuration, even
  past the 90 days
  ([§10.2][srules-10.2]).
  See [Audits](../understand/how-submission-works.md#audits).
- **Keep a CoN endpoint reachable** for at least **90 days** after publication, with a point of
  contact
  ([§7.2.5][srules-7.2.5]).
- **Honour a Preview commitment:** Available and re-submitted within 180 days of first
  publication, with one possible extension
  ([§7.3][srules-7.3]).
  See [Publication status](../rules/publication-status.md).
- **Use the right name.** Unqualified "MLPerf Endpoints" means Standardized; Serviced, RDI and
  Preview results carry qualified names
  ([§2.2.3][rules-2.2.3],
  [§7.3.8][srules-7.3.8]).

## Verify

```bash
endpoints-submission-cli submissions get --submission-id <submission-id>
```

You are done when status reads `FINALIZED`, and published when it reads `PUBLISHED`. Full state
list: [Submission states](../reference/submission-states.md).

## Next

Nothing. You've finished the workflow. If something went wrong, see
[Troubleshooting](../help/troubleshooting.md) or [Getting support](../help/support.md).
