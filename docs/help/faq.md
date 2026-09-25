# FAQ

Process and policy questions that are not errors. For errors, see
[Troubleshooting](troubleshooting.md).

!!! note "An FAQ entry is a documentation bug"
    If a question keeps getting asked here, the workflow page that should have answered it is at
    fault. The fix is to improve that page and delete the entry, not to grow this one.

## Getting started

**Do I have to be an MLCommons member to submit?**
:   **Unconfirmed.** The rules require the [MLCommons CLA](../understand/eligibility/cla-process.md) from the *individual* making the
    submission ([Submission Rules §5.1][srules-5.1]), but no clause makes [organisational
    membership](../understand/eligibility/membership.md) a precondition. Ask MLCommons before
    planning around either answer. See [Membership, PRISM and
    eligibility](../understand/eligibility/index.md) and **A1** in [Open
    questions](open-questions.md).

**Does everyone in my organisation need to sign the CLA?**
:   No. Only the individual making the submission signs it
    ([Submission Rules §5.1][srules-5.1]).

**Is there a registration deadline?**
:   No. The general MLPerf eight-week advance registration doesn't apply
    ([Submission Rules §5.1][srules-5.1]). Registration is having a [PRISM API
    key](../understand/eligibility/prism-api-key.md).

**When is the submission deadline?**
:   There isn't one; submissions are rolling. What matters is the **cohort cutoff**: automated
    checks must pass at least one business day before a publication date ([Submission Rules
    §4.3][srules-4.3]).

## Scope of a submission

**Can I submit more than one model?**
:   One submission is one Pareto curve: one system, one benchmark model, one dataset. Multiple
    models means multiple submissions.

**How many points do I need?**
:   For a non-agentic benchmark, 8 to 32: `1 + 3 + 3` fixed-concurrency points plus an Offline
    point. If you elect your `C_max` point as the Offline result, 7 is enough. Agentic benchmarks
    need 7 and have no Offline point. See [§5.3 of the rules][rules-5.3] and [Plan your Pareto
    curve](../workflow/plan-your-curve.md).

**Do I really need a separate Offline run?**
:   No. You can elect your `C_max` point as the Offline result instead, provided you have a point
    at exactly `C_max`. You give up whatever extra throughput an unpaced run might have shown, and
    save a run plus its accuracy validation. See [§5.7.2 of the rules][rules-5.7.2] and [step
    3](../workflow/plan-your-curve.md#6-decide-how-to-meet-the-offline-requirement).

**Can I add points after submitting?**
:   **No.** The post-submission window for adding measurement points was removed. There is no
    `add-run`, and `submissions update --run-ids` rejects any list that would add one. The technical
    rules still mention the old window; see **B2** in [Open questions](open-questions.md).

**Can I remove a bad point?**
:   Yes, with `submissions remove-run` during peer review. But withdrawn points do **not** count
    toward the minimum ([Submission Rules §8.1][srules-8.1]) and the shortfall **cannot be
    repaired** by adding a replacement. This is why planning a spare point matters.

**Do I need an accuracy run per point?**
:   No. You need one at five points: the four mandatory region points and the Offline point, or
    four for agentic benchmarks ([§5.3 of the rules][rules-5.3]). How each must pass, single-turn
    versus multi-turn, is in [§4.3][rules-4.3].

## Rules

**Is response caching allowed?**
:   No. Every request must execute the forward pass
    ([§2.9.9 Q1 of the rules][rules-2.9.9]).

**Is KV-cache reuse across queries allowed?**
:   **Yes**, on a per-query salted token stream. This is the main difference from MLPerf Inference.
    See [§2.9.5 of the rules][rules-2.9.5] and [Model
    equivalence](../rules/model-equivalence.md#kv-cache-the-big-difference).

**Can I use a different serving framework than the reference?**
:   Yes, if it conforms to the rest of the rules and meets the Available definition
    ([§2.9.9 Q3 of the rules][rules-2.9.9]).

**Can I quantize?**
:   Yes. Post-training quantization is permitted under the conditions in
    [§2.9.3 of the rules][rules-2.9.3].

**Can I use speculative decoding?**
:   Only with a drafter on the benchmark's approved list. No list has been published yet, so for
    now the answer is no for every benchmark, and the checker rejects any point that uses it. See
    [§2.9.4 of the rules][rules-2.9.4] and **C10** in [Open questions](open-questions.md).

**Can I disable speculative decoding on some points?**
:   Yes. The drafter is fixed across the curve, but its configuration can vary per point, including
    off ([§2.9.4][rules-2.9.4]). A drafter that ships in the canonical checkpoint must stay in your
    derived checkpoint ([§2.9.3][rules-2.9.3]).

**Do I have to report power?**
:   For Standardized, yes, as provisioned power rather than a meter reading
    ([§4.5 of the rules][rules-4.5]). The checker wants a `system_power.json` for every system
    whatever your division (**B11** in [Open questions](open-questions.md)). See [Power
    normalization](../rules/requirements.md#power-normalization).

**Does passing the accuracy gate make an optimisation legal?**
:   No. Accuracy is **necessary, not sufficient** ([§2.9.8 of the rules][rules-2.9.8]).

## Publication

**Can I publish before review finishes?**
:   Yes, by opting in to provisional publication at submission time. **The choice is
    irrevocable.** See [Submission Rules §6.2.3][srules-6.2.3].

**Can I delay publication?**
:   Yes, with an embargo date declared at submission. What it holds back depends on your
    publication mode; see [Submission Rules §4.2][srules-publication-embargo] and
    [§6.2][srules-6.2].

**Can I be audited?**
:   Yes. From the moment you're nominated you must keep the system as submitted. See
    [Submission Rules §10][srules-10] and [Audits](../understand/how-submission-works.md#audits).

**What happens if my Preview system does not become available in time?**
:   The result is **invalidated and removed** at the next cohort, unless you got the one extension
    allowed. See [Submission Rules §7.3.5][srules-7.3.5] and [§7.3.6][srules-7.3.6].

**Can I relabel a Preview result as Available without re-running?**
:   Only if the hardware and software configuration did not change materially. If it did, you must
    re-run ([Submission Rules §7.3.7][srules-7.3.7]).

**Can I fix an error in a published result?**
:   No. The affected points or the whole submission are invalidated instead
    ([Submission Rules §8.1][srules-8.1]).

## Review

**Who reviews my submission?**
:   The review committee for your cohort, plus one designated reviewer assigned at random from
    outside your organisation. See [Submission Rules §2.1][srules-2.1] and [§2.6][srules-2.6].

**Is a competitor reviewing me a conflict of interest?**
:   No, and neither are CSP/OEM/ODM partnerships
    ([Submission Rules §2.4][srules-2.4]).

**How fast do I have to respond to an objection?**
:   **3 business days**, with local public holidays exempt. Penalties escalate to withdrawal at 10
    business days and are non-reversible ([Submission Rules §6.3][srules-6.3]).

**Can I improve my results during review?**
:   Only **when the review committee requests it**, and any improvement to performance metrics must
    be justified ([Submission Rules §6.3][srules-updating-submissions-during-peer-review]).

**What if someone objects after review closes?**
:   Only on limited grounds, and reproducibility isn't one of them: a reproducibility concern
    becomes an audit nomination instead. See [Submission Rules §6.6][srules-6.6].

**How long does my result stay challengeable?**
:   Until the later of the next audit vote or **90 days** after finalization. You must keep the
    system available for audit until then, and longer if you're nominated ([Submission Rules
    §6.6][srules-scope-and-standing-for-late-concerns]).

## Naming and messaging

**What do I call my result?**
:   Unqualified "MLPerf Endpoints" means **Standardized**. Serviced, RDI and Preview results carry a
    qualifier. See [§2.2.3][rules-2.2.3], [§2.3.2][rules-2.3.2] and [§2.4.2][rules-2.4.2] of the
    rules, and [Submission Rules §7.3.8][srules-7.3.8].

**Do I have to say anything when quoting a provisional result?**
:   Yes, the MLCommons footnote saying results are preliminary
    ([Submission Rules §6.2.3][srules-6.2.3]).
