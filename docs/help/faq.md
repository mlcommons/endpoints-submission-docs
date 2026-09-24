# FAQ

Process and policy questions that are not errors. For errors, see
[Troubleshooting](troubleshooting.md).

!!! note "An FAQ entry is a documentation bug"
    If a question keeps getting asked here, the workflow page that should have answered it is at
    fault. The fix is to improve that page and delete the entry, not to grow this one.

## Getting started

**Do I have to be an MLCommons member to submit?**
:   **Unconfirmed.** The rules require the MLCommons CLA from the *individual* making the
    submission, and describe registering at MLCommons Member Central with an organisation email. No
    clause states that organisational membership is a precondition. Ask MLCommons before planning
    around either answer. See [Membership, PRISM and eligibility](../understand/eligibility.md) and
    **A1** in [Open questions](open-questions.md).

**Does everyone in my organisation need to sign the CLA?**
:   No. The requirement applies to the individual making the submission.

**Is there a registration deadline?**
:   No. The eight-week advance registration requirement from the general MLPerf rules is explicitly
    overridden for Endpoints. Registration is having a PRISM API key.

**When is the submission deadline?**
:   There isn't one. Submissions are rolling. What matters is the **cohort cutoff**: automated
    checks must pass at least one business day before the 1st or 3rd Wednesday publication date.

## Scope of a submission

**Can I submit more than one model?**
:   One submission is one Pareto curve: one system, one benchmark model, one dataset. Multiple
    models means multiple submissions.

**How many points do I need?**
:   For a non-agentic benchmark, 8 to 32: `1 + 3 + 3` fixed-concurrency points plus an Offline
    point. If you elect your `C_max` point as the Offline result, 7 is enough. Agentic benchmarks
    need 7 and have no Offline point. See [Plan your Pareto curve](../workflow/plan-your-curve.md).

**Do I really need a separate Offline run?**
:   No. You can elect your `C_max` point as the Offline result instead, provided you have a point
    at exactly `C_max`. You give up whatever extra throughput an unpaced run might have shown, and
    save a run plus its accuracy validation. See
    [step 3](../workflow/plan-your-curve.md#6-decide-how-to-meet-the-offline-requirement).

**Can I add points after submitting?**
:   **No.** The post-submission window for adding measurement points was removed. There is no
    `add-run`, and `submissions update --run-ids` rejects any list that would add one.

**Can I remove a bad point?**
:   Yes, with `submissions remove-run` during peer review. But withdrawn points do **not** count
    toward the minimum and the shortfall **cannot be repaired** by adding a replacement. This is
    why planning a spare point matters.

**Do I need an accuracy run per point?**
:   Not at every point, but at five of them: the four mandatory region points (Ultra Low, Low,
    Medium and High Concurrency) and the Offline point. Agentic benchmarks need four. All use the
    same endpoint configuration, weights and software stack as the performance runs. For
    single-turn benchmarks every result must pass, and each run goes at its point's concurrency
    immediately after that point's performance run; for multi-turn benchmarks only the average must
    pass.

## Rules

**Is response caching allowed?**
:   No. Returning a cached response verbatim to a matching request is prohibited. Every request must
    execute the forward pass.

**Is KV-cache reuse across queries allowed?**
:   **Yes.** This is the main difference from MLPerf Inference. It counts as a serving optimisation, not
    response caching, because the forward pass still runs on a per-query salted token stream. See
    [Model equivalence](../rules/model-equivalence.md).

**Can I use a different serving framework than the reference?**
:   Yes. Arbitrary frameworks and runtimes are permitted provided they conform to the rest of the
    rules: model equivalence, no benchmark detection, no input-based optimization. The framework
    must satisfy the Available definition.

**Can I quantize?**
:   Yes. PTQ is the standard permitted transformation, subject to four conditions: published
    calibration set only, publicly described, passes the accuracy gate, disclosed in the YAML.

**Can I use speculative decoding?**
:   Only with a drafter on the benchmark's approved list. No list has been published yet, so for
    now the answer is no for every benchmark, and the checker rejects any point that uses it. See
    [Model equivalence](../rules/model-equivalence.md#speculative-decoding).

**Can I disable speculative decoding on some points?**
:   Yes. The **drafter** must be the same across the curve, but its *configuration* may vary per
    point, including disabling speculation entirely. A drafter that ships in the canonical
    checkpoint must still be present in your derived checkpoint. You can't strip it out.

**Do I have to report power?**
:   For Standardized, yes: a `system_power.json` per system is required, and results are also
    published as `system_tps_per_kw`. It's optional for RDI and not yet defined for Serviced. It's
    provisioned power from spec-sheet ratings, not a meter reading. See
    [Power normalization](../rules/requirements.md#power-normalization).

**Does passing the accuracy gate make an optimisation legal?**
:   No. Accuracy is **necessary, not sufficient**. A submission that hits the quality target while
    breaking one of the rules (pruning weights, removing experts, fine-tuning a drafter) is not
    model equivalent.

## Publication

**Can I publish before review finishes?**
:   Yes, by opting in to provisional publication at submission time. Results carry a *peer review
    pending* tag and every public reference must carry the MLCommons footnote. Review starts when
    the result goes public. **The choice is irrevocable**. You can't request it later.

**Can I delay publication?**
:   Yes, with an embargo date declared at submission. Under confidential review, the finalized
    result is held for up to 60 days after review completes, and review itself isn't delayed. Under
    provisional publication, the embargo holds back the tagged result and the start of review with
    it. The date can be changed afterwards, but the change is broadcast to all review committee
    members.

**Can I be audited?**
:   Yes. There are two audits a quarter: one drawn at random, one chosen by committee vote from
    nominations made in the 4 weeks after publication. From the moment you're nominated you must
    keep the system as submitted. See [Audits](../understand/how-submission-works.md#audits).

**What happens if my Preview system does not become available in time?**
:   The result is **invalidated and removed** at the next cohort, not archived. One extension of up
    to 60 days may be granted if requested at least 30 days before expiry. A second will be denied.

**Can I relabel a Preview result as Available without re-running?**
:   Only if the hardware and software configuration did not change materially. If it did, you must
    re-run.

**Can I fix an error in a published result?**
:   No. Corrections to finalized results are not permitted. The affected points or the whole
    submission are invalidated instead. Non-result corrections need review-chair approval and publish
    with a change log.

## Review

**Who reviews my submission?**
:   The committee for your cohort, drawn from organisations with a finalized Endpoints result in the
    preceding 6 months or 12 cohorts. Plus one designated reviewer assigned at random, excluding your
    own organisation.

**Is a competitor reviewing me a conflict of interest?**
:   No, explicitly. The model is built on competitors reviewing each other. CSP/OEM/ODM partnerships
    are not conflicts either. Conflicts are limited to direct financial interest, employment
    relationships and similar.

**How fast do I have to respond to an objection?**
:   **3 business days**, with local public holidays exempt. Penalties escalate to withdrawal at 10
    business days and are non-reversible.

**Can I improve my results during review?**
:   You may only update run and submission metadata **when the review committee requests it**. Any
    improvement to performance metrics must be justified and explained to the committee.

**What if someone objects after review closes?**
:   Late objections are permitted only on availability, validity, model-equivalence and
    division-rule grounds, and only from review committee members. Reproducibility is **not**
    eligible. That becomes an audit nomination instead.

**How long does my result stay challengeable?**
:   Until the later of the next audit vote or **90 days** after finalization. Your obligation to
    retain the system for a possible audit runs to the same point, and longer if you're nominated
    or selected for audit.

## Naming and messaging

**What do I call my result?**
:   Unqualified "MLPerf Endpoints" means **Standardized**. Serviced and RDI results must use
    "MLPerf Endpoints Serviced" and "MLPerf Endpoints RDI". Preview results must carry the "Preview"
    qualifier. Omitting it violates MLCommons usage guidelines.

**Do I have to say anything when quoting a provisional result?**
:   Yes. Include the MLCommons footnote saying results are preliminary and peer review is pending.
    That applies to you, MLCommons, press and third parties alike.
