<!--
  PROVENANCE SNAPSHOT — do not edit.
  Upstream : endpoints_policies/endpoints_submission_rules.md
  Repo     : mlcommons/endpoints_policies @ v1.0_rules_dev@6b0b1ef
  Captured : 2026-09-24
  Purpose  : diff this against upstream to find what drifted since the docs were written.
-->

# MLPerf® Endpoints Submission Rules

*MLPerf Endpoints Rules Task Force — Version 1.0 Draft — 2026-05-05*

> [!NOTE]
> This document supplements and, where specified, overrides the
> [MLPerf® General Submission Rules](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc)
> for the MLPerf Endpoints benchmark suite.
> Sections not explicitly overridden here inherit from the general rules without modification.

---

## Table of Contents

1. [Basics](#1-basics)
2. [Review Committee](#2-review-committee)
   - [2.1 Structure](#21-structure)
   - [2.2 Review Chair](#22-review-chair)
   - [2.3 Reviewer Obligations](#23-reviewer-obligations)
   - [2.4 Conflict of Interest](#24-conflict-of-interest)
   - [2.5 Confidential and Not Precedent Setting](#25-confidential-and-not-precedent-setting)
   - [2.6 Peer Review Process](#26-peer-review-process)
3. [Operating Principles](#3-operating-principles)
4. [Schedule](#4-schedule)
   - [4.0 Submission Milestones](#40-submission-milestones)
   - [4.1 Rolling Submission Model](#41-rolling-submission-model)
   - [4.2 Publication Cohorts and Embargo](#42-publication-cohorts-and-embargo)
   - [4.3 Submission-to-Publication Alignment](#43-submission-to-publication-alignment)
   - [4.4 Benchmark Roadmap](#44-benchmark-roadmap)
   - [4.5 Review Cycle Example](#45-review-cycle-example)
   - [4.6 Seed Rotation](#46-seed-rotation)
5. [Submission](#5-submission)
   - [5.1 Registration](#51-registration)
   - [5.2 How to Submit](#52-how-to-submit)
   - [5.3 Late Submissions](#53-late-submissions)
   - [5.4 Licensing](#54-licensing)
   - [5.5 Submission Content](#55-submission-content)
   - [5.6 system\_desc\_id.json Metadata](#56-system_desc_idjson-metadata)
   - [5.7 Logging Requirements](#57-logging-requirements)
   - [5.8 Compliance Testing](#58-compliance-testing)
6. [Review](#6-review)
   - [6.1 Automated Compliance (Week 0)](#61-automated-compliance-week-0)
   - [6.2 Publication Modes](#62-publication-modes)
   - [6.3 Peer Review (Weeks 1–3)](#63-peer-review-weeks-13)
   - [6.4 Objection Resolution (Weeks 4–6)](#64-objection-resolution-weeks-46)
   - [6.5 Review Timeline Summary](#65-review-timeline-summary)
   - [6.6 Late Objections (Post Week 6)](#66-late-objections-post-week-6)
   - [6.7 Filing Objections](#67-filing-objections)
   - [6.8 Types of Objections](#68-types-of-objections)
   - [6.9 No Dedicated Review Meetings](#69-no-dedicated-review-meetings)
   - [6.10 Visibility of Results During Review](#610-visibility-of-results-during-review)
   - [6.11 Withdrawing Results](#611-withdrawing-results)
7. [Publication](#7-publication)
   - [7.1 Results Categories](#71-results-categories)
   - [7.2 Available](#72-available)
   - [7.3 Preview](#73-preview)
   - [7.4 RDI (Research, Development, or Internal)](#74-rdi-research-development-or-internal)
   - [7.5 Open Question: Custom SKU Classification](#75-open-question-custom-sku-classification-custom-sku)
   - [7.6 Results Table Content](#76-results-table-content)
8. [Post-Publication](#8-post-publication)
   - [8.1 Pareto Updates](#81-pareto-updates)
   - [8.2 Corrections](#82-corrections)
   - [8.3 Withdrawal](#83-withdrawal)
   - [8.4 Terms of Use](#84-terms-of-use)
   - [8.5 Issues Discovered After Publication](#85-issues-discovered-after-publication)
9. [Dispute Resolution](#9-dispute-resolution)
   - [9.1 Scope](#91-scope)
   - [9.2 Escalation Path](#92-escalation-path)
   - [9.3 Remedies](#93-remedies)
   - [9.4 Appeal](#94-appeal)
   - [9.5 Status](#95-status)
10. [Audit Process](#10-audit-process)
    - [10.1 Audit Quota](#101-audit-quota)
    - [10.2 Audit Votes](#102-audit-votes)
    - [10.3 Random Audit Selection](#103-random-audit-selection)
    - [10.4 Audit Nomination on Reproducibility Grounds](#104-audit-nomination-on-reproducibility-grounds)
    - [10.5 Audit Compliance and Resolution Rules](#105-audit-compliance-and-resolution-rules)
11. [Appendices](#11-appendices)

---

## 1. Basics

These rules define the submission, review, and publication process for the MLPerf Endpoints benchmark suite. MLPerf Endpoints inherits the general principles of the [MLPerf® General Submission Rules](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc) — including transparency, mandatory peer review, and objection-based quality control — but replaces the batch submission model with a *rolling submission model* suited to the faster cadence of Gen AI inference benchmarking.

Where this document conflicts with the MLPerf General Submission Rules, this document takes precedence for MLPerf Endpoints submissions.

Technical requirements — including benchmarks, metrics, the pareto collection methodology, and division-specific rules — are defined in the companion [MLPerf Endpoints Rules](endpoints_rules.md) document. Publication status categories and availability criteria are defined in [§7](#7-publication) of this document.

> [!NOTE]
> **Rule Stability.** These rules are *tentative* until the **v1.0** submission round opens on **2026-10-12**, after which rolling submission begins. Sections explicitly marked **`[TENTATIVE — Subject to change after 2026-10-12]`** in either this document or in [`endpoints_rules.md`](endpoints_rules.md) are the most likely to evolve, based on submitter feedback from the v0.7 round (which closed on 2026-06-26) and on working-group discussion. See [§4.0 Submission Milestones](#40-submission-milestones) for the full milestone table.

The review process is designed to:

- **Publish results quickly.** Submitters and the industry should not have to wait months for results to appear. The rolling model targets rapid, bi-weekly publication.
- **Align publication to embargo dates.** Submitters may wish to have results become public on particular dates to align with launches, conferences, and other events.
- **Scale with workload.** Each submission may contain tens of measurement points (pareto curves), significantly more than a single-number inference submission. The process must handle this without proportionally increasing reviewer burden.
- **Avoid dedicated review meetings.** Reviews are conducted asynchronously via GitHub issues. Meetings are called only when disputes cannot be resolved offline.
- **Maintain credibility.** Early publication of results before peer review is completed is clearly labeled and carries mandatory disclaimers.

---

## 2. Review Committee

### 2.1 Structure

The review committee for a given cohort consists of representatives from organizations that have participated in MLPerf Endpoints within the preceding **6 months or 12 cohorts, whichever is longer**, counting backward from the first day of the publication month.

"Participated" means the organization has at least one MLPerf Endpoints result that has completed the full review process and been published without a "peer review pending" tag (i.e., finalized results). Results that were published but are still carrying the "peer review pending" tag do not count toward participation eligibility. Organizations whose only finalized results within the lookback period were subsequently withdrawn are not eligible for committee membership.

> **Example:** A submission published in the 2026-12-C1 cohort draws its review committee from organizations that submitted results published on or after 2026-06-01.

### 2.2 Review Chair

A standing review chair is selected every quarter. The chair does not need to be a member of the review committee and does not need to have participated in MLPerf Endpoints — they should be knowledgeable about MLPerf benchmarking processes and preferably neutral (i.e., not affiliated with an active Endpoints submitter). One co-chair is elected or nominated from the review committee every 3 months on a rotating basis.

> [!NOTE]
> **[WG Open Item]** — Should the review chair be required to be an MLCommons member? Arguments for: ensures familiarity with MLCommons processes and governance; provides accountability. Arguments against: limits the pool of qualified neutral candidates. The working group must decide before the first submission round.

Responsibilities of the chair and co-chair include:

- Monitoring the review pipeline and ensuring timely progress through review phases.
- Calling review committee meetings when disputes escalate (see [§9 Dispute Resolution](#9-dispute-resolution)).
- Certifying that submissions have completed the review process.
- Recusing themselves from review of their own organization's submissions (if applicable).
- Overseeing peer review assignment (see [§2.6](#26-peer-review-process)).

### 2.3 Reviewer Obligations

Review committee members are expected to participate in peer review of submissions within their area of expertise. Beyond the assigned reviews defined in [§2.6](#26-peer-review-process), there is no mandatory minimum number of reviews per member, but the committee is collectively responsible for ensuring all submissions receive adequate scrutiny within the review window.

### 2.4 Conflict of Interest

Committee members must recuse themselves from reviewing their own organization's submissions. Members may also voluntarily recuse on any other grounds. Recusals are recorded in the submission's GitHub issue thread.

For clarity: being a competitor is not a conflict of interest. The peer review model is inherently built on competitors reviewing each other's work. Similarly, CSP/OEM/ODM partnership relationships between the reviewer's and submitter's organizations do not constitute a conflict of interest. Conflicts of interest are limited to situations such as direct financial interest in a submission's outcome, employment relationships, or other circumstances that would compromise a reviewer's objectivity beyond normal competitive dynamics.

Submitters or other review committee members may raise conflict of interest concerns about a reviewer at any time during the review process. The review chair will evaluate the concern and determine whether recusal is warranted.

#### Neutral Members

Several processes in these rules are staffed by **neutral members**: the dispute resolution panel ([§9.2](#92-escalation-path)), the objection review panel ([§8.5](#85-issues-discovered-after-publication)), and audit-nomination screening ([§10.4](#104-audit-nomination-on-reproducibility-grounds)).

A neutral member is a person with **minimal conflict of interest in the matter at hand**, assessed against the criteria above: no direct financial interest in its outcome, no employment or equivalent relationship with either party, and no involvement in preparing or reviewing the submission in question. Consistent with the paragraph above, ordinary competitive relationships and CSP/OEM/ODM partnerships do **not** disqualify a person from serving as a neutral member.

Neutral members **need not be members of the review committee**, and need not belong to the MLPerf Endpoints working group. Where the committee cannot supply enough conflict-free members — because the parties to a dispute between them account for much of the committee, or because the subject matter is narrow — the review chair may appoint neutrals from outside it.

Neutral members **should** have practical experience with AI benchmarking and with MLPerf in particular: familiarity with submission and review practice, with the divisions and publication status categories, and with the measurement methodology. This is a preference rather than a requirement. Where the two cannot both be satisfied, freedom from conflict takes precedence over subject-matter experience.

The review chair records the basis on which each neutral member was selected, in the submission's issue thread or in the dispute record. Either party may raise a conflict of interest concern about a proposed neutral member, which the chair evaluates as above.

### 2.5 Confidential and Not Precedent Setting

*Inherits from [General Submission Rules §2.4](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#confidential-and-not-precedent-setting) without modification.*

### 2.6 Peer Review Process

Ensuring all submissions are rigorously evaluated and adhere to the rules and policies of MLPerf is critical to maintaining the integrity of MLPerf. Any member of the review committee is entitled to conduct peer review of any submission and to verify that it meets the requirements of the MLPerf Endpoints policy documents. To ensure that every submission is reviewed by at least one member, each new submission is additionally assigned a designated reviewer. Subsequent updates to the pareto ([§8.1](#81-pareto-updates)) are evaluated by the same assigned reviewer.

An assigned review must be completed before the close of the peer review window (end of Week 3; see [§6.3](#63-peer-review-weeks-13)). Assigned reviews are the one exception to [§2.3](#23-reviewer-obligations): members carry no minimum review quota, but an assignment once received must be completed.

#### Reviewer Assignment

- The submission CLI randomly selects a reviewer from the eligible pool when the submission is received, before the peer review period begins. The review chairs oversee assignment and may override it as described below.
- The submitting organization is excluded from the pool for its own submissions, per [§2.4 Conflict of Interest](#24-conflict-of-interest).
- Once a member has been assigned a review, they are removed from the selection pool until every other member of the pool has completed at least one assigned review. A member may opt back into the pool earlier, once their current assignment is complete. If the rotation cannot advance because assignments remain outstanding, the review chairs may reset it.
- Review chairs may reassign a review if the assigned reviewer recuses themselves, or if the chairs determine the reviewer lacks the resources or experience to review the particular submission.
- Review chairs may request a member to review more than one submission, depending on the member's availability.

#### Failure to Complete an Assigned Review

An organization that does not complete an assigned review by the close of the peer review window is removed from the review committee. It becomes eligible for committee membership again once it has a newly finalized MLPerf Endpoints result — a published result no longer carrying the "peer review pending" tag, per the participation definition in [§2.1](#21-structure).

#### Scope of Review

A peer review should cover, at minimum:

- Whether the results are within the expected and reasonable range for the hardware and software used.
- Whether the reproducibility instructions are clear and easy to follow.
- The benchmark methodology.
- The content of the JSON files in the `systems` directory.

Reviewers should open a GitHub issue for any problem they find or any question they have, per [§6.7 Filing Objections](#67-filing-objections). Where a submission contains more results than a reviewer can cover, they should focus on the subset they can handle, prioritizing high-performing results and those that compete against other submissions. This list is not exhaustive — any other issue noticed in a submission should be raised the same way.

Reproducing results is not required. Where a reviewer does attempt reproduction, inconsistencies are assessed against the margins defined in [§6.6 Reproducibility Expectations](#reproducibility-expectations).

---

## 3. Operating Principles

*Inherits from [General Submission Rules §3](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#operating-principles) without modification.*

MLPerf's purpose is to produce fair and useful benchmark results.

The MLPerf Endpoints review committee reserves the right to depart from these rules and/or exclude submissions that conflict with this purpose with a two-thirds (rounded up) vote by the submitters.

The role of the review process is to ensure fairness of submissions, not to litigate details in an effort to disqualify competitors.

---

## 4. Schedule

### 4.0 Submission Milestones

> [!NOTE]
> **Rule Stability** (cross-reference [§1 Basics](#1-basics)). These rules are *tentative* until the v1.0 round opens. Submitters should expect rules — particularly sections marked `[TENTATIVE — Subject to change after 2026-10-12]` in [`endpoints_rules.md`](endpoints_rules.md) — to evolve up to that point, based on submitter feedback from v0.7.

| Date | Milestone | Notes |
|---|---|---|
| **2026-06-26** | MLPerf Endpoints **v0.7** submission deadline (Submission Round 1) | First endpoint submission round. Rules in this document and in [`endpoints_rules.md`](endpoints_rules.md) apply. |
| **2026-07-31** | MLPerf Inference **v6.1** submission deadline | Traditional MLPerf Inference round (separate process, governed by [inference_rules.adoc](https://github.com/mlcommons/inference_policies/blob/master/inference_rules.adoc) and the [General Submission Rules](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc)). Endpoints does **not** gate on this date; it is listed here for awareness. |
| **2026-10-12** | MLPerf Endpoints **v1.0** submission round opens (Submission Round 2) + start of rolling submission | Opening of the v1.0 round. Rules were revised after v0.7 closed, based on submitter feedback. Rolling submission ([§4.1](#41-rolling-submission-model)) begins on this date for v1.0 and beyond. |

Until the v1.0 round opens, the rules in this document and in [`endpoints_rules.md`](endpoints_rules.md) may be revised by working-group consensus or by the operating-principles 2/3 vote (inherited from [General Submission Rules §3](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#operating-principles)). Rule revisions targeting v1.0 follow the same process. The sections most likely to change are those marked `[TENTATIVE — Subject to change after 2026-10-12]`.

### 4.1 Rolling Submission Model

MLPerf Endpoints uses a **rolling submission model**. Submitters may submit on any day of the month — there is no fixed submission deadline. Each submission is timestamped upon receipt and enters the review pipeline immediately.

This section replaces the batch submission schedule defined in the MLPerf General Submission Rules §4 for MLPerf Endpoints submissions.

### 4.2 Publication Cohorts and Embargo

MLCommons publishes results on a bi-weekly cadence, on the **1st and 3rd Wednesday of each month at 8:00 AM Pacific Time**.

Each cohort is identified as:

- `YYYY-MM-C0` — published on the 1st Wednesday of the month at 8:00 AM PT.
- `YYYY-MM-C1` — published on the 3rd Wednesday of the month at 8:00 AM PT.

> **Example:** A submission received on a Monday could appear in the next Wednesday's cohort at the earliest, subject to passing automated compliance checks and the 1 business-day alignment window (see [§4.3](#43-submission-to-publication-alignment)).

#### Publication Embargo

Submitters may request that MLCommons hold publication of their results until a specific embargo date, declared at the time of submission.

- **Confidential review with embargoed publication** ([§6.2.2](#622-confidential-review-with-embargoed-publication)): results are not public until review is complete. The embargo date delays public release of the finalized result, and may be up to **60 days after the completion of review**. Review itself is unaffected.
- **Provisional publication** ([§6.2.3](#623-provisional-publication)): the embargo date delays when the "peer review pending" result first becomes publicly visible, and may be set to any date before finalization of results. Because peer review begins at provisional publication, this embargo also defers the start of review.

Rules for embargoed submissions:

- The embargo date must be declared in the submission metadata.
- Under [§6.2.2](#622-confidential-review-with-embargoed-publication) the full review process proceeds normally under the embargo — the embargo delays only public release. Under [§6.2.3](#623-provisional-publication) the embargo additionally defers the start of peer review, since review begins at provisional publication.
- The embargo date **may be changed after submission**, but any change must be broadcast immediately to all review committee members.
- All review committee members are informed of the embargo date.
- Results under embargo are published on the requested embargo date and are not tied to the regular cohort schedule.

### 4.3 Submission-to-Publication Alignment

A submission is eligible for the next cohort if it passes automated compliance checks (see [§6.1 Automated Compliance](#61-automated-compliance-week-1)) at least 1 business day before the Wednesday publication date. Submissions that do not clear automated checks in time roll to the following cohort.

All review timelines, update windows, and objection deadlines are anchored to the **cohort in which a submission first appears** — not the raw submission date.

### 4.4 Benchmark Roadmap

*Inherits from [General Submission Rules §4.3](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#benchmark-roadmap-schedule) without modification.*

### 4.5 Review Cycle Example

The figure below traces four submissions entering the pipeline shortly after the rolling submission window opens on **Monday 12 October 2026**, one in each publication mode of [§6.2](#62-publication-modes). It shows how automated compliance, peer review, objection resolution, provisional publication, embargo, dispute resolution, and cohort publication interact across calendar time. The relevant cohorts are `2026-10-C1` (21 Oct), `2026-11-C0` (4 Nov), `2026-11-C1` (18 Nov), `2026-12-C0` (2 Dec), `2026-12-C1` (16 Dec), `2027-01-C0` (6 Jan) and `2027-01-C1` (20 Jan).

![MLPerf Endpoints Submission and Review Cycle](review_cycle.svg)

**Scenario A — confidential review, no objections.** A submitter files on 12 October, the day the window opens. Automated compliance passes on 13 October and peer review begins ([§6.1](#61-automated-compliance-week-0)). No objections are filed, so at the close of Week 3 on 3 November the submission qualifies for early finalization ([§6.3](#63-peer-review-weeks-13)) and publishes in the **2026-11-C0** cohort on 4 November. Nothing was publicly visible before that date.

**Scenario B — provisional publication under embargo ([§6.2.3](#623-provisional-publication)).** A submitter files on 22 October, opts in to provisional publication, and declares an embargo through 30 October. Automated compliance passes on 23 October, but **peer review does not begin there**: under this mode review starts at provisional publication, so the embargo holds both. On 30 October the embargo lifts, the "peer review pending" result becomes publicly visible, and Week 1 begins. An objection is filed on 5 November; the submitter responds within three business days on 10 November. Peer review closes on 20 November and the objection is resolved on 25 November, inside the Week 4–6 window. The result is finalized and published in the **2026-12-C0** cohort on 2 December, at which point the pending tag is removed.

**Scenario C — unresolved at Week 6, escalation to dispute resolution.** A submitter files on 26 October. An objection filed on 10 November is still open when the objection resolution window closes on 8 December, so escalation is automatic ([§9.2](#92-escalation-path)). The chair certifies the open objection and names the panel on 10 December, written statements are due on 24 December, the panel convenes by 31 December, and a binding decision issues on 14 January. The submission does not finalize while the dispute is open; it publishes in the **2027-01-C1** cohort on 20 January. Had the process not concluded, the 8-week backstop would have required the chair to decide on the record by 2 February.

**Scenario D — confidential review with embargoed publication ([§6.2.2](#622-confidential-review-with-embargoed-publication)).** A submitter files on 19 October under confidential review and declares a publication embargo through 8 December. Automated compliance passes on 20 October and peer review begins immediately — the embargo does not defer review in this mode. No objections are filed, so the submission reaches early finalization on 10 November. The finalized result is then **held**: it is not published at the next cohort, and it is never publicly visible carrying a "peer review pending" tag. It is released on the embargo date, 8 December, which is 28 days after review completed and so within the 60-day limit of [§4.2](#42-publication-cohorts-and-embargo).

> [!NOTE]
> **[WG Open Item — embargo versus cohort cadence]** Scenarios B and D both publish on a date that is not a cohort date, which [§4.2](#42-publication-cohorts-and-embargo) permits: embargoed results "are published on the requested embargo date and are not tied to the regular cohort schedule". In Scenario B the effect is more than off-cadence — a submission clearing compliance on 23 October would not otherwise reach a cohort until 4 November, so an embargo to 30 October makes the result public *earlier* than it could have been without one. The working group should decide whether an embargo date may precede the submission's next eligible cohort, and whether off-cadence publication is intended for one, both, or neither mode.

### 4.6 Seed Rotation

A **seed set** is the collection of seeds published by MLCommons that control the reference client's sources of run-to-run non-determinism for a cohort. These seeds drive the random number generators the client uses for benchmarking (request-issue / sample order, and the per-query salt). The seed set is an *extensible collection* — additional seeds may be introduced in future versions without changing this rule. This mirrors MLPerf Inference, where MLCommons rotates the LoadGen seeds (`qsl_rng_seed`, `sample_index_rng_seed`, `schedule_rng_seed`) every submission round.

MLCommons refreshes the seed set **once every two publication cohorts**. Its relationship to the cohort has two distinct parts — a window during which a *new* submission may **adopt** a set, and the lifetime for which a submission stays **bound** to the set it adopted. Keeping these separate is what lets a rolling submission keep growing without seed rotation ever cutting it short.

- **Publication and adoption window.** MLCommons publishes a new seed set every two publication cohorts ([§4.2](#42-publication-cohorts-and-embargo)), keyed by the cohort ID (`YYYY-MM-C0` / `YYYY-MM-C1`) in which it is published. Each published seed set is available for **adoption by new submissions for four consecutive cohorts** — its publication cohort and the following three cohorts — and is then dropped from the sets available for adoption. Because refresh occurs every two cohorts and each set remains adoptable for four, **two seed sets are normally available for adoption**. For example, a set published in cohort `N` is adoptable in cohorts `N` through `N+3`; the next set is published in `N+2`, and the first set is dropped when cohort `N+4` begins. The adoption window governs only which set a *new* submission may bind to; it does **not** expire the seed set of a submission already in flight (see *Binding lifetime* below).

- **Binding at first submission.** A submission binds to exactly **one** seed set when it first appears, chosen from the sets in its adoption window. The adopted seed set and the targeted cohort MUST be recorded in the submission ([`endpoints_rules.md` §8.3](endpoints_rules.md#83-measurement-point-yaml)) so a reviewer or auditor can reproduce the run and the seeded-RNG integrity check ([`endpoints_rules.md` §2.1.1](endpoints_rules.md#211-client-on-prem-cop)) can confirm the client used the published seeds without modification.

- **Binding lifetime.** Once a submission binds to a seed set, that set stays valid **for that submission for the full applicable Pareto-update window** ([§8.1](#81-pareto-updates)), even after the set's adoption window has closed and newer sets have been published. Every measurement point added later MUST use the bound seed set. Because the binding is fixed at first submission and does not expire with rotation, seed rotation never forces an in-flight run to be re-executed, an embargo of up to 60 days ([§4.2](#42-publication-cohorts-and-embargo)) never invalidates a submission, and changing the length of the Pareto-update window does not change seed-set adoption or binding. A *new* submission (as distinct from an update to an existing one) must always adopt a set within its current adoption window — an expired set may not be adopted afresh — but that set remains valid for every submission already bound to it.

- **Comparability.** All submissions bound to the same seed set are directly comparable. Because a submission keeps its seed set for its full update window, two submissions being compared may hold different seed sets; such comparison is permitted on the assumption that seed choice has a negligible effect on measured performance.

---

## 5. Submission

### 5.1 Registration

*Overrides General Submission Rules §5.1 (the eight-week advance registration requirement does not apply to rolling submissions).*

The **individual making the submission** must have signed the relevant MLCommons CLA. The CLA requirement applies to that individual — it is not required that every member of their organization has signed. 

Register with PRISM by creating your account at MLCommons Member Central using your organization email id. Once a membership account has been created, you will be granted API creation access and sent an email notifying that, after which, you can sign in and create an API key with Service Scope MLPerf Endpoints from your API Keys dashboard.

### 5.2 How to Submit

Submissions are made via the `endpoints-submission-cli`, authenticated with the PRISM API key from §5.1 (set as `PRISM_USER_API_TOKEN` or passed via `--token`).

**Workflow:**

1. **Run the benchmark.** For each Pareto point, run `inference-endpoint benchmark` with config.yml having `system_info` section(if you want to automatically capture the system description). Each invocation writes a local run folder containing system info, configuration, and result summaries.

2. **Register each run.** `endpoints-submission-cli runs create --token <PRISM_USER_API_TOKEN> --path <folder>` uploads the run to MLCommons storage and returns a `run_id`.

3. **Create the submission.** `endpoints-submission-cli submissions create --run-ids ... --division <...> --scenario <...> --availability <...>` assembles the bundle from the registered runs and submits it. Please refer to the detailed README for the [run](https://github.com/mlcommons/endpoints-submission-cli/blob/endpointsubcli/docs/endpoints-cli/usage/runs.md)) and [submission](https://github.com/mlcommons/endpoints-submission-cli/blob/endpointsubcli/docs/endpoints-cli/usage/submissions.md) command structures.

**What happens on `submissions create`:**

- The Submission Checker runs locally on the assembled bundle. On failure, nothing is uploaded; the submitter corrects the issues and resubmits.
- On pass, the bundle is uploaded to MLCommons storage, the submission is assigned a publication cycle (next 1st or 3rd Wednesday with ≥1 business day of buffer), and a peer-review PR is automatically opened in the private MLCommons review repository. Reviewers and the submitting organization are granted access; objections are filed as PR comments and resolved per §6.
- On finalization, results are published in the public MLPerf Endpoints results repository and shown in the visualizer on the next publication-cycle date.

The submission must include all materials required by the [MLPerf Endpoints Rules](endpoints_rules.md) document. Detailed CLI usage — including amendments, withdrawal, and run lifecycle — is documented in the MLPerf Endpoints [Submission Guide](https://docs.google.com/document/d/1mgJrGqilxG9Kn_tzVFifSQQnuUWjMihVLMp_WNZRVRM/edit?usp=sharing).

### 5.3 Late Submissions

The "late submissions" provisions of the General Submission Rules §5.3 do not apply to MLPerf Endpoints, as there is no fixed submission deadline.

### 5.4 Licensing

*Inherits from [General Submission Rules §5.4](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#licensing) without modification.*

All submissions of code must be made under the MLCommons CLA. Per the CLA, all submissions of code will be Apache 2 compatible. Third party libraries need not be Apache 2 licensed.

### 5.5 Submission Content

A submission must contain the following:

- Metadata for the system under test (`system_desc_id.json`).
- Pareto curve YAML configuration files (one per measurement point).
- Run result artifacts for each measurement point (log files, metric summaries).
- Accuracy validation run artifacts.
- Code that implements the benchmark endpoint interface.
- Metadata describing the system-implementation combination tested.

Full content and directory structure requirements are defined in [MLPerf Endpoints Rules §7](endpoints_rules.md#7-submission-requirements).

### 5.6 `system_desc_id.json` Metadata

The complete list of fields is defined in [MLPerf Endpoints Rules §8.2](endpoints_rules.md#82-system-description-system_desc_idjson), which is the canonical source.

### 5.7 Logging Requirements

Results logs must be produced by the MLPerf Endpoints reference client using the `ConcurrencyScheduler` load pattern. Per-run artifacts must conform to the MLPerf Endpoints data dictionary.

### 5.8 Compliance Testing

Submitters must run the automated compliance validator prior to submission. The compliance validator checks pareto structure, run requirements, accuracy, and metadata as described in [MLPerf Endpoints Rules §9](endpoints_rules.md#9-compliance-validation).

---

## 6. Review

### 6.1 Automated Compliance (Week 0)

Automated compliance checks are executed immediately after submission. The checks must complete within **one calendar week (Week 0)**, but may complete much sooner — as quickly as one day depending on submission size and queue depth. These checks verify:

- All required submission materials are present (YAML configurations, result artifacts, system descriptions).
- The pareto curve satisfies minimum point count and region coverage requirements (minimum 7 points structured as 1 + 3 + 3; maximum 32 points total).
- Run durations, minimum query counts, load patterns, and streaming configuration meet requirements.
- Accuracy validation results are included and pass the quality target.
- Metric consistency: the valid per-response TPOT distribution is non-empty with a finite, strictly positive P90 and its normalized millisecond value is used to derive `tps_per_user = 1000 / tpot_p90_ms`.
- Configuration consistency across measurement points (same model, same endpoint, same software stack).

The full list of automated checks is defined in [MLPerf Endpoints Rules §9](endpoints_rules.md#9-compliance-validation).

**If a submission fails any automated check by the end of Week 0, it is rejected.** The submitter is notified of the specific failures and may correct the issues and resubmit as a new submission. Rejected submissions do not enter the peer review phase. The peer review period begins as soon as all automated checks have passed — except for submissions using provisional publication, where it begins at provisional publication (see [§6.2](#62-publication-modes)).

### 6.2 Publication Modes

Every submission declares one of three publication modes at submission time. The choice is **irrevocable after submission**.

| Mode | Public before finalization | Peer review begins | Finalized results published |
|---|---|---|---|
| **A. Confidential review** *(default)* | No | When automated checks pass | First cohort after finalization |
| **B. Confidential review, embargoed publication** | No | When automated checks pass | On the declared embargo date |
| **C. Provisional publication** | Yes — tagged "peer review pending" | At provisional publication | Tag removed at finalization |

#### 6.2.1 Confidential Review (default)

Results and artifacts are visible to the review committee and to other submitters, but are not published publicly until review is complete — all objections resolved, none pending. Results are published in the first cohort after finalization. Peer review begins as soon as automated compliance passes ([§6.1](#61-automated-compliance-week-0)).

#### 6.2.2 Confidential Review with Embargoed Publication

A submitter who does not want provisional publication, but does want to control the date on which finalized results become public, may declare a **publication embargo**. Review is unaffected by the embargo: it is confidential and it begins and runs exactly as in [§6.2.1](#621-confidential-review-default). Only the public release of the **finalized** result is held.

- The embargo date is declared at submission and may be up to **60 days after the completion of review** ([§4.2](#42-publication-cohorts-and-embargo)).
- No result is ever publicly visible carrying a "peer review pending" tag under this mode.
- If review completes before the embargo date, the result is finalized on schedule and held, then published on the embargo date.
- If review is still running when the embargo date passes, the embargo has no further effect and the result is published at the first cohort after finalization.

This mode suits a submitter aligning publication to a launch, conference or earnings date who does not want preliminary numbers in public beforehand.

#### 6.2.3 Provisional Publication

Submitters may **opt in** to provisional publication at the time of submission. If a submitter opts in:

- Results are published before peer review completes, carrying a **"peer review pending"** tag. This allows submitters to reference new results in time-sensitive contexts — such as keynote presentations, product launches, and press briefings — without waiting for the full review cycle.
- The review committee is informed of the opt-in at the start of the review cycle.
- **Peer review begins at provisional publication**, not when automated compliance passes. Reviewers and the public see the result at the same time.

**Embargo under provisional publication.** A submitter who opts in may additionally declare an **embargo date** — a hold on when the "peer review pending" result first becomes publicly visible. The embargo date may be any date before the finalization of results. Because peer review begins at provisional publication, an embargo under this mode **also defers the start of peer review**, and therefore defers finalization by the same amount. See [§4.2](#42-publication-cohorts-and-embargo) for general embargo rules, including how to change the embargo date after submission.

> [!IMPORTANT]
> Any reference to results carrying the "peer review pending" tag — by MLCommons, submitters, press, or third parties — must include the standard MLCommons footnote stating that results are **preliminary and subject to change** pending peer review. The exact footnote text is defined in the MLPerf Results Messaging Guidelines.

### 6.3 Peer Review (Weeks 1–3)

Once automated compliance checks pass, the submission enters the peer review phase, which runs through the end of Week 3. Review committee members may:

- Examine the submission materials, run logs, and configuration files.
- File objections as GitHub issues on the submission repository (see [§6.7 Filing Objections](#67-filing-objections)).
- Request clarification from the submitter via the issue thread.

**No new objections may be filed after the close of the peer review window (end of Week 3).** Objections not filed during this window are not eligible for the objection resolution phase and may only be raised through the late objection process (see [§6.6](#66-late-objections-post-week-6)).

**Response timelines within the peer review window:**

- Once an objection is filed, the **submitter must post an initial response within 3 business days**, counting from the day the objection is filed. Business day counting accounts for local public holidays in the submitter's primary operating jurisdiction — days falling on a local holiday do not count against the window. The response must either acknowledge the issue and include a **schedule for resolution** — which may extend into the objection resolution window if needed — or contest the objection with a counter-argument and supporting evidence.
- After the submitter responds, the **objecting party has 2 business days** to acknowledge the response, retract the objection, or indicate that the issue remains unresolved and will carry into the objection resolution window. The objecting party may also provide a **schedule or timeline** for testing and validating the proposed resolution, in which case the objection remains open until that validation is complete or the stated timeline has elapsed.

**Submitter non-response penalties.** Failure to respond to a filed objection triggers automatic penalties based on elapsed business days since the objection was filed. Business day counting follows the same local holiday rule as the response window. These penalties apply throughout both the peer review and objection resolution windows and are enforced by the review chair without requiring a separate motion.

| Business days elapsed without submitter response | Penalty |
|---|---|
| 3 business days | Results finalization is automatically delayed by **1 publication cohort**. |
| 6 business days | Delay increases to **2 publication cohorts**. |
| 10 business days | The submission is **withdrawn**. The submitter may correct the issues and resubmit as a new submission. |

Penalties are cumulative and non-reversible — responding after a penalty threshold has been crossed does not remove the penalty, though subsequent response may prevent further escalation. The review chair must notify the submitter and all review committee members when a penalty threshold is crossed.

**Early finalization.** Objections may be fully resolved during the peer review window. If all objections are resolved or retracted before the end of Week 3, the submission is eligible for **early finalization** — it does not need to wait for the close of the objection resolution window. The review chair certifies early finalization and the submission is queued for the next available cohort.

#### Updating submissions during peer review

After compliance checks pass for a submission, submitters may only update run and submission metadata when requested by review committee. Updates are restricted to cases where insufficient information, code, or instructions was provided, or a material flaw is discovered that must be rectified. Any improvement to performance metrics must be justified and explained to the review committee. Submitters are encouraged to use the GitHub Web UI for making changes. Any and all changes are synced up with the database, and should shortly be viewable in the visualizer.

### 6.4 Objection Resolution (Weeks 4–6)

From Week 4 through Week 6, all filed objections carried over from the peer review window must be resolved. All objections must reach resolution; escalation to dispute resolution is reserved for cases where the parties cannot agree on the facts or interpretation.

**Response timelines within the resolution window:**

- For each open objection entering the resolution window, the **submitter must provide a fix or formal resolution response within 3 business days**, accounting for local holidays, consistent with the resolution schedule declared during peer review. Non-response penalties from [§6.3](#63-peer-review-weeks-13) continue to apply.
- After the submitter's resolution response, the **objecting party has 2 business days** to either acknowledge resolution, explicitly explain — with specificity — why the response is not sufficient to close the objection, or provide a **schedule or timeline** for testing and validating the resolution. Silence after 3 business days is treated as acknowledgment of resolution and the objection is automatically retracted.
- If the review chair determines that a resolution timeline is not being met, they may intervene to set a binding deadline or call a resolution meeting.

**Meeting escalation.** If an objection is not close to resolution **12 business days after it was first filed**, or if any open objection is entering Week 5 of the review cycle, the review chairs may call a meeting with the relevant parties to expedite and close the issue. Attendance at such a meeting is expected of both the objector and the submitter.

An objection is considered resolved when:

- The submitter addresses the concern to the objecting party's satisfaction, or
- The review chair determines the objection is not substantiated, or
- The objecting party retracts the objection, or
- The objecting party does not respond within 3 business days of the submitter's resolution response.

Unresolved objections at the end of Week 6 are escalated to the dispute resolution process. Escalation is **automatic**: the review chair certifies which objections remain open and refers them, without requiring a motion from either party. A submission with an escalated objection does not finalize until the dispute concludes. The process and its deadlines are set out in [§9.2](#92-escalation-path).

Once all objections are resolved or retracted, the "peer review pending" tag is removed and the submission's results are finalized.

### 6.5 Review Timeline Summary

| Phase | Window | Key Actions |
|---|---|---|
| Automated Compliance | Week 0 (up to 1 week; may complete in as little as 1 day) | Automated checks run. Pass → advances to peer review immediately. Fail by end of Week 0 → **rejected**; submitter may resubmit. Results remain confidential by default; provisional publication if submitter opted in (subject to embargo). |
| Peer Review | Weeks 1–3 | Committee reviews; objections filed via GitHub (no new objections after end of Week 3); submitter has 3 business days (local holidays exempt) to respond with a resolution schedule; non-response penalties: +1 cohort at 3 biz days, +2 cohorts at 6, withdrawn at 10; objector has 2 business days to respond or provide a validation schedule. If all objections resolved before end of Week 3 → eligible for early finalization. |
| Objection Resolution | Weeks 4–6 | Open objections must be resolved; same non-response penalty schedule applies; objector has 2 business days to explain insufficiency or provide validation schedule (silence = objection retracted after 3 days); chairs may call meeting if objection is 12+ biz days old or entering Week 5; submission finalized or escalated to dispute resolution. |
| Dispute Resolution | From end of Week 6, ~5 weeks | Escalation is automatic for objections still open at Week 6; the submission does not finalize. Chair certifies and names a panel within 2 business days; written statements within 10; panel convenes within 15; recommendation +5; binding decision +5. One investigation extension. 8-week backstop, after which the chair decides on the record. Appeal within 14 days. |
| Late Objections ⚠️ | Post Week 6 | Availability, validity, model-equivalence, and division-rule objections only, via the dispute resolution process. Reproducibility is not eligible. **[WIP — pending WG approval]** |

### 6.6 Late Objections (Post Week 6)

> [!WARNING]
> **[WIP — Pending Working Group Approval]** — The late objection policy is under active discussion and has not yet been ratified by the working group. The grounds, process, and time limits described below are a current proposal and are subject to change.

After Week 6, late objections may be raised only on the following grounds:

- **Availability** — the system does not meet the availability status claimed at submission.
- **Validity** — specific metric values are found to be incorrect or inconsistent with known hardware capabilities.
- **Model equivalence** — the submission does not meet the model-equivalence rules of [Endpoints Rules §2.9](endpoints_rules.md#29-model-equivalence-rules-standardized-division): for example a prohibited weight transformation, an undisclosed approximation, or a drafter, sparsity, or KV-cache configuration that was never declared.
- **Division rules** — the submission does not meet the requirements of the division under which it was published, or was published in the wrong division.

Reproducibility remains **ineligible** as a late objection ground; reproducibility objections must be filed within the peer review window. A member with a reproducibility concern about a published result instead makes their case by **nominating the submission for audit** under [§10.4](#104-audit-nomination-on-reproducibility-grounds).

> [!NOTE]
> **Why these grounds extend past Week 6.** Model-equivalence and division-rule violations are frequently not discoverable during the review window. Under [§6.10](#610-visibility-of-results-during-review), code and submission artifacts are visible only to the review committee and other submitters until finalization, so for many parties the first opportunity to examine them arises after Week 6.

As with objections raised during review, the remedy for a division or availability misclassification is normally reclassification rather than withdrawal ([§6.8](#68-types-of-objections)).

Late objections are handled through the dispute resolution process (see [§9](#9-dispute-resolution)).

#### Scope and Standing for Late Concerns

The limits in this subsection apply both to late objections under §6.6 and to audit nominations under [§10.4](#104-audit-nomination-on-reproducibility-grounds).

**Standing.** Only members of the review committee ([§2.1](#21-structure)) may raise a late objection or nominate a submission for audit. An organization that is not on the committee for the cohort in question may bring its concern to a committee member. [§8.5](#85-issues-discovered-after-publication) remains open to any MLCommons member, but only for allegations of direct fraud or misrepresentation.

**Time window.** The window runs from **finalization** — the point at which all objections are resolved and the "peer review pending" tag is removed ([§6.4](#64-objection-resolution-weeks-46)) — not from provisional publication. A result is open to late objection and audit nomination until the **later** of:

- the next **audit vote** following finalization ([§10.2](#102-audit-votes)), or
- **90 calendar days** after finalization.

Once that point passes the result is settled, and is no longer subject to late objection or audit nomination.

Anchoring to finalization gives every submission the same exposure period. Anchoring to first publication would penalize submitters who opt in to provisional publication ([§6.2](#623-provisional-publication)), whose results appear weeks earlier and would therefore settle sooner than an otherwise identical confidential submission.

The 90-day floor guarantees a minimum challenge period regardless of where in the audit-vote cycle a result lands, while the audit-vote ceiling keeps the period bounded. It matches the 90-day endpoint-accessibility requirement for Standardized CoN submissions ([§7.2.5](#725-division-specific-available-requirements)), so that the endpoint remains reachable for as long as the result can be challenged. A submitter's obligation to retain the benchmarked system and its configuration for a possible audit runs to the close of this window and no further — except where the submission is nominated or selected for audit, in which case retention runs as set out in [§10.2](#102-audit-votes).

> *Example:* A result finalized 10 days before an audit vote does not settle at that vote — 90 days have not elapsed — and settles on day 90. A result finalized 100 days before the next vote remains open until that vote.

**Supersession by a more recent submission.** Where the submitter has a more recent **finalized** result for the same benchmark model on a *similar system*, that result is taken as the reference for evaluating a reproducibility or validity concern, and supersedes the concern as it applies to the older submission. A similar system is one with the same accelerator model and count, the same host and interconnect topology, and a software stack differing only in component versions. The submitter identifies the superseding result; the review chair confirms it meets these criteria.

Supersession does **not** apply to availability, division-rule, or model-equivalence concerns — a later compliant submission is evidence about performance, not a cure for an earlier misclassification or equivalence violation — nor where the concern alleges intentional misrepresentation ([§8.5](#85-issues-discovered-after-publication)).

#### Reproducibility Expectations

Perfect reproducibility of results cannot be reasonably expected and must not be used to block publication unless the deviation is egregious. Due to natural variability in silicon, machine configuration, setup, power delivery, cooling, and thermal state, some run-to-run variation is expected.

**Throughput metrics.** The margins below apply to `system_tps`, and to `tps_per_user` derived from it ([Endpoints Rules §4.1](endpoints_rules.md#41-primary-metrics)):

- **Up to 10%** variability is expected and allowed when an independent party re-runs the benchmark during the review period. Large-scale submissions (hundreds of accelerators) may exhibit higher variance and should be assessed with proportionally greater tolerance.
- **Within 5%** when re-running on the **exact same system** — for example during an audit.

**Latency metrics.** These margins do **not** apply to `ttft_p90_ms`, or to any other latency percentile. A fixed percentage band is not a sound test for a percentile statistic:

- A percentile is a substantially noisier estimator than a mean, and its sampling error depends on the number of completed queries at the measurement point.
- TTFT distributions are right-skewed and heavy-tailed, so a band that is generous for a mean can be punishing for a tail statistic.
- The absolute scale spans orders of magnitude across the pareto — tens of milliseconds in the Low Latency region, seconds at high concurrency — so a single relative band is simultaneously too tight at one end and too loose at the other.
- For Client-over-Network submissions, measured TTFT includes public Internet latency that the submitter does not control, and whose run-to-run variation may exceed any submitter-attributable difference.

**Interim rule.** Until the working group ratifies a method, a reproducibility objection may not rest on latency metrics alone. Latency evidence may be offered in support of an objection whose primary basis is throughput or accuracy, and a reviewer may raise an unexplained latency discrepancy as a *Suspect or Incomprehensible Results* objection ([§6.8](#68-types-of-objections)).

**Accuracy must always pass** — the accuracy quality target is a hard gate with no variability allowance, both during automated compliance and throughout the review period.

> [!NOTE]
> **[WG Decision Required]** — The 10% and 5% throughput margins are current proposals and require ratification before they can be enforced.
>
> **A method for latency metrics has yet to be developed.** The per-metric **histogram** the reference client writes with each run is the natural input: it carries the whole distribution rather than a single percentile, and needs no new instrumentation. Three prerequisites must be settled before it can support cross-run comparison — bin edges specified by the reference client and identical across every run and measurement point (log-spaced, given the range latency spans across the pareto); per-bin counts retained so that the sample size is recoverable; and coverage restricted to steady state, excluding warmup per [Endpoints Rules §6.3.2](endpoints_rules.md#632-discard-policy). Given those, candidate tests include agreement across several quantiles rather than one, a distributional distance over the binned counts (Wasserstein, or a chi-square over bins), and bootstrap confidence intervals resampled from the histogram. The histogram's own resolution supplies a principled noise floor: a difference smaller than one bin width at the quantile under test is not actionable. Separating the network component for CoN submissions remains desirable, and [Endpoints Rules §2.1.1](endpoints_rules.md#211-client-on-prem-cop) already requires a measured baseline for CoP.
>
> **Artifact prerequisite.** [Endpoints Rules §8.1](endpoints_rules.md#81-directory-structure) does not currently require the histogram to be retained: the per-point artifacts are `result_summary.json` with aggregate metrics and percentiles. Any method resting on the histogram requires it to be added as a required artifact, with its binning fixed in the data dictionary referenced by [§5.7](#57-logging-requirements).
>
> The working group should also decide whether large-scale submissions warrant a distinct throughput threshold.

### 6.7 Filing Objections

Objections must be filed as GitHub issues on the submission repository before the end of Week 3. Each objection must:

- Cite the specific offending material (log excerpts, configuration values, reproduction attempts).
- Reference the applicable rule or section number.
- Include a `by <org>` tag and an `against <org>` tag.

Multiple organizations may append their `by <org>` to an existing objection. An objector who determines their objection is in error may remove their `by <org>` tag. All objections with no `by <org>` tags at the close of the peer review window will be closed.

### 6.8 Types of Objections

Objections filed during peer review must be categorized as one of the following types. All objections must cite specific evidence (log excerpts, configuration values, reproduction attempts) and reference the applicable rule or section.

| Type | Description | Severity |
|---|---|---|
| **Compliance Failure** | Submission does not meet stated rules (point count, region coverage, run duration, load pattern, etc.). | High — may require withdrawal. |
| **Methodology** | Disagreement with how the benchmark was configured or executed (e.g., dataset handling, warmup procedure). | High. |
| **Reproducibility** | Results cannot be reproduced by an independent party or appear statistically implausible. A reproducibility objection must demonstrate deviation beyond the allowed variability margin for the metric in question (see [§6.6 Reproducibility Expectations](#reproducibility-expectations)); the throughput margins do not apply to latency metrics, and an objection may not rest on latency alone until a method is ratified. Minor deviations within the expected range are not grounds for blocking publication. Accuracy failures are always a valid reproducibility objection regardless of margin. Reproducibility objections must be filed during the peer review window (through the end of Week 3) and are not eligible as late objections; after finalization, a reproducibility concern is pursued by nominating the submission for audit under [§10.4](#104-audit-nomination-on-reproducibility-grounds). | High — but must exceed the allowed variability margin to be actionable. |
| **Validity of Results** | Specific metric values appear incorrect, inconsistent, or incompatible with known hardware capabilities. | High. |
| **Division Rules** | Submission placed in wrong division, or system does not meet division requirements (availability, API compliance, etc.). The review committee may allow the submitting organization to reclassify to the correct division rather than withdraw. | Medium. |
| **Availability** | System claimed as Available or Preview does not meet the availability requirements at the stated date. The review committee may allow the submitting organization to reclassify (e.g., from Available to Preview or RDI) rather than withdraw. | Medium. |
| **Suspect or Incomprehensible Results** | Results are anomalous or unexplainable but no specific rule violation can be identified. Triggers an investigation. | Medium. |
| **Cosmetic** | Formatting errors, metadata issues, or presentation problems that do not affect scores. Does not block finalization. | Low. |

### 6.9 No Dedicated Review Meetings

Standing review meetings are not required. All peer review is conducted asynchronously via GitHub issues. Meetings are convened only when:

- An objection is escalated to the dispute resolution process.
- A submitter or committee member explicitly requests a meeting to resolve a specific issue.
- The review chair determines that a meeting would materially accelerate resolution.

Meeting requests must include a written agenda and specific questions to be addressed.

### 6.10 Visibility of Results During Review

*Overrides [General Submission Rules §6.1](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#visibility-of-results-and-code-during-review).*

| Group | During Review (Weeks 1–6) | After Finalization |
|---|---|---|
| Review committee | All results, all code, all run artifacts. | All results, all code, all run artifacts. |
| Submitters | All results, all code, all run artifacts. | All results, all code, all run artifacts. |
| Public | Results carrying the "peer review pending" tag only (for submissions that have opted in to provisional publication, once any declared embargo has lifted). No access to code or submission artifacts. | All results, all code, all submission artifacts. |

For submissions that have not opted in to provisional publication (see [§6.2](#623-provisional-publication)), the public has no visibility until results are finalized.

### 6.11 Withdrawing Results

A submission may be withdrawn at any time up until finalization. Post-finalization withdrawals are handled per [§8.3 Withdrawal](#83-withdrawal).

---

## 7. Publication

MLCommons publishes all results per the bi-weekly cadence defined in [§4.2 Publication Cohorts and Embargo](#42-publication-cohorts-and-embargo). After publication, code and results are public and free for use under the MLPerf Terms of Use.

### 7.1 Results Categories

*Overrides [General Submission Rules §7.3](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#results-categories).*

Results are divided into three publication status categories based on the availability of the hardware and software components at the time of submission.

| Category | Hardware | Software |
|---|---|---|
| **Available** | All components meet the four-point availability test at submission time. | Available software stack. |
| **Preview** | Does not yet qualify as Available; submitter commits to Available status within 180 days. | Available except software supporting substantially new hardware. |
| **RDI** (Research, Development, or Internal) | Does not meet Available or Preview requirements. | Does not meet Available or Preview requirements. |

An RDI component may not be submitted as Available or Preview until the cohort after next, or **221 days** after first publication as RDI, whichever is longer.

### 7.2 Available

A system is **Available** if all of its components that substantially determine ML performance meet the following four-point test at the time of submission:

| # | Criterion | Requirements |
|---|---|---|
| 1 | **Pricing** | Pricing is available — either publicly advertised or available upon request. |
| 2 | **Shipment** | The component or system has been shipped to or rented by at least one third party. |
| 3 | **Public Evidence** | There is externally verifiable public evidence that the component is actually shipping or available to purchase *today* — not merely announced, previewed, or soft-launched. See [§7.2.1](#721-public-evidence--proof-of-shipment-not-announcement) below. |
| 4 | **Reasonably Available** | The component or system is reasonably available for purchase or rent by additional third parties by the submission date. See [§7.2.2](#722-reasonably-available) below. |

**Assessing availability — preponderance of evidence.** Availability is a judgment call and no single data point is conclusive. The review committee evaluates availability based on the totality of available positive and negative signals. Signals indicating a product is likely available include: a sales representative actively willing to take orders; pricing on the vendor's website or obtainable on request; customer case studies referencing the product as deployed; official press releases confirming the product is *shipping*. In case of conflicting information, a vendor is not penalized for uneven availability across geographies or channels, provided availability exists in at least one market.

#### 7.2.1 Public Evidence — Proof of Shipment, Not Announcement

Evidence must prove the component is actually shipping or orderable *today* — not just announced. The following are **not sufficient** on their own:

- Announcements at a conference, trade show, or keynote.
- Product listings on a marketing or "coming soon" website.
- Soft-launch press releases or blog posts describing a future availability date.
- A "Preview" or "coming soon" entry in a cloud provider's product catalog.
- A stated or anticipated launch date without accompanying proof of current availability.

Evidence that *is* sufficient includes:

- Clearly listed availability on the vendor's official product or pricing page showing a live, functional call to action for qualifying customers (e.g., "Buy now," "Order," "Get started," or equivalent purchasing language — these are examples; other terms indicating an active, orderable listing are acceptable).
- Public claims by the vendor explicitly stating the product is "now shipping," "generally available," or "in production" — in a press release, earnings call, official blog post, or equivalent public statement.
- Publicly verifiable fulfillment data (a cloud provider's instance type appearing in live pricing and availability APIs, orderable by any qualifying customer).
- Third-party review units or press loaner systems confirmed by the recipient.
- Public purchase orders or customer shipment confirmations available in securities filings or regulatory disclosures.

For **component vendors** (accelerators, ASICs, memory, networking): the criterion is assessed from the component's position in the supply chain. The component must be available to ship to system integrators, OEMs, and ODMs under standard commercial terms.

**Custom SKUs.** See [§7.5](#75-custom-sku-classification-custom-sku) for the rules governing custom SKUs and hardware in large-scale production that is not available to all comparable customers.

**Vendor's own GA definition.** If the submitting organization has a formal internal process that defines when a product is generally available (e.g., "open shipping status," "phase 4 exit," "FCS"), the submitted system must satisfy that internal definition *in addition to* all criteria above.

#### 7.2.2 Reasonably Available

"Reasonably available" means the system or component is purchasable or rentable by any qualifying customer on commercially reasonable terms, including:

1. **Non-discrimination.** Competitors must not be blocked from purchasing or renting the system. All software, drivers, firmware, and support services made available to any customer must be made available to any other qualifying customer, including direct competitors, on equivalent terms.
2. **Access conditions.** Access to rent or purchase may be subject to conditions common to generally available products (financial qualifications, size of customer, support burden, export restrictions) but is not otherwise restricted — no "early access" approval requirements.
3. **Supply and lead times.** Supply and lead times are subject to market conditions and demand at the time of purchase; they are not governed by these rules and are not required to be fixed or guaranteed. MLPerf reproducibility does not confer any priority slot or allocation right — a reviewer or auditor wishing to reproduce results is subject to the same supply and demand constraints as any other customer. Individual components and entire systems may be subject to supply-demand constraints appropriate to their scale and market.

However, it is allowed for the qualifying pre-submission rentals or purchases to have been made with restrictions such as "early access" approval.

#### 7.2.3 Available Software Stack

An **Available** system must use an **Available** software stack — the set of software components that substantially determine ML performance but are not in the uploaded source code (e.g., the inference framework, ML accelerator library, kernel-level drivers).

| Category | Examples | Availability Rule |
|---|---|---|
| **ML frameworks** | PyTorch, TensorFlow, JAX | Any commit in an official public repository. Open PRs that add architecture support are permitted, provided the PR is publicly accessible. |
| **Inference servers** | TensorRT-LLM, vLLM, SGLang, Triton | Open-source: any public commit + accessible open PRs. Commercial: official release or labeled beta in the formal release sequence (see below). |
| **Accelerator compute libraries** | cuDNN, cuBLAS, NCCL, ROCm HIP | Official release or publicly available labeled beta in the formal release sequence. One-off private binary drops shared only with specific customers do not qualify. |
| **Hardware drivers** | CUDA driver, ROCm driver, NIC firmware | Official release or publicly available beta, downloadable through a standard vendor channel at the time of submission. |
| **Quantization / optimization tools** | bitsandbytes, AutoAWQ, llama.cpp, GPTQ | Same rules as ML frameworks if open-source; same rules as accelerator compute libraries if closed-source binary. |
| **Custom OS / kernel patches** | Custom Linux kernel builds, BIOS/firmware | Exempt if unmodified commodity software. Custom patches that substantially affect ML performance must be in an upstream-merged commit or publicly accessible PR. Private patches are not permitted. |
| **Submission code / custom kernels** | Custom CUDA kernels, Flash Attention variants | Not part of the software stack — must be included in the submitted source code package. |

**Labeled beta qualification.** A closed-source binary may qualify as Available if it is a labeled beta in the formal release sequence, provided: (1) the vendor has publicly committed that the optimizations will be included in a future official release; (2) the beta is offered to customers as a standard step in the release process — not a one-off private engagement; and (3) the beta is publicly downloadable at the time of submission. A release candidate satisfying these criteria qualifies; an engineering sample under NDA distributed to selected partners does not.

Software must be available at the time of **submission**. If a software component becomes unavailable between submission and publication, the submitter must notify the review committee. The committee may allow the submission to proceed if an equivalent public release is available, or may require reclassification.

#### 7.2.4 Available Models

The model weights and tokenizer used in the submission must be available through commercially accessible channels (e.g., a public model hub under a commercially permissive license, a vendor model download portal, or inclusion in the software product).

Model weights distributed under licenses that prohibit commercial deployment do not qualify the system as Available. Such submissions must be classified as Preview or RDI.

#### 7.2.5 Division-Specific Available Requirements

**Standardized division.** Hardware, software stack, and model must all meet §7.2.1–7.2.4. For CoN submissions, the endpoint URL must remain accessible for at least **90 days** after publication to support reproducibility verification; submitters must provide a point of contact for access requests during this period.

**Serviced division.** The endpoint must be at **full GA tier** — not a "Public Preview," "Open Beta," or "Generally Available in Preview" tier, even if publicly accessible. Preview-tier endpoints typically lack SLA commitments, may be subject to breaking changes, and may be withdrawn without notice; they do not represent the provider's full production commitment. Where the provider's own internal GA definition draws a distinction between preview-accessible and production-GA, the production-GA designation is required.

The **endpoint URL submitted for benchmarking must be the same endpoint that any paying customer uses in production.** Submitters may not use a capacity-reserved, dedicated, or otherwise privileged endpoint not available to the general public (e.g., a dedicated inference cluster standing up only for the benchmark run, or an internally hosted mirror with relaxed rate limits). Submissions found to have used non-public or specially provisioned endpoints are subject to withdrawal regardless of when the discovery is made.

The endpoint's **terms of service must permit benchmarking** by third parties. Any MLCommons member in good standing must be able to independently access the endpoint under standard terms and attempt to reproduce the published results. A Serviced submission whose ToS prohibits competitive benchmarking or automated performance testing does not qualify for Available status. Submitters must confirm at submission time that no provision of their ToS, acceptable use policy, or rate-limiting policies would prevent a member from conducting a good-faith reproducibility test.

**RDI division.** RDI division submissions carry RDI publication status and cannot qualify as Available or Preview. If the system subsequently becomes commercially available, the submitter must create a new Standardized or Serviced submission.

### 7.3 Preview

A **Preview** system does not qualify as Available at the time of submission, but the submitter commits to making it Available within **180 days** of its first publication in MLPerf Endpoints, and commits to re-submitting as Available at that time.

Preview results are published with a **"Preview — Available by [date]"** tag. If the system does not achieve Available status within the 180-day window, the result is automatically invalidated.

#### 7.3.1 Preview Window and Clock

- **Clock start:** The date of the cohort in which the result *first appears* — not the raw submission date.
- **Clock duration:** 180 calendar days from the clock start.
- **Clock anchor:** Anchored to the first publication date. Pareto updates, corrections, or system description amendments do not reset the clock.

*Example:* A result first published in the 2026-04-C1 cycle (April 30, 2026) has an availability deadline of October 27, 2026.

#### 7.3.2 Software Waiver for Preview

The Available software stack requirement (§7.2.3) is waived for software components necessary to support **newly developed hardware** that substantially determines ML performance (e.g., a new ML accelerator). "Newly developed" means the hardware was not Available as of the previous cohort and was not submitted as Preview in that cohort. All other software stack components must still meet the Available requirements.

#### 7.3.3 Performance Continuity Requirement

When a Preview submission transitions to Available, the re-submitted result must achieve equal or better performance compared to the Preview result.

**Throughput metrics.** For `system_tps` and `tps_per_user`, a degradation of up to **5%** is accepted, to account for variance inherent to endpoints workloads — the high-interactivity region of the throughput–latency curve is sensitive to load-generation noise, and large-scale systems exhibit higher run-to-run variance than traditional batch inference. The tolerance applies to each such metric independently.

**Latency metrics.** The 5% tolerance does **not** apply to `ttft_p90_ms` or to any other latency percentile, for the reasons set out in [§6.6 Reproducibility Expectations](#reproducibility-expectations): a fixed percentage band is not a sound test for a percentile statistic. Until the working group ratifies a comparison method for latency, a Preview-to-Available transition is not blocked on latency alone. A material and unexplained latency regression is instead raised as an objection during the re-submission's own review window, subject to the same interim rule.

> [!NOTE]
> **[WG Decision Required]** — The 5% throughput margin has been approved by the Task Force. The latency comparison method is open, and is shared with [§6.6](#reproducibility-expectations); until it is settled, this section is enforceable on throughput metrics only.

#### 7.3.4 Declaration Requirements

Submitters claiming Preview status must, at submission time:

1. Set `"availability_status": "preview"` in the system description.
2. State a `"target_availability_date"` within the 180-day window as an ISO 8601 date.
3. Identify **with specificity** which components are not yet available — e.g., *"The X100 GPU is expected to begin customer shipments in Q3 2026."* Vague statements such as *"targeting H2 2026"* are not sufficient.

#### 7.3.5 Preview Tracker and Expiration

MLCommons maintains a **Preview Availability Tracker** — a public document listing all active Preview results, their first publication dates, target availability dates, and days remaining. It is updated with each cohort.

At expiration of the 180-day window:

- **Available re-submission published:** Preview result is superseded by the Available result.
- **No re-submission, no extension:** Preview result is **invalidated** and removed at the next cohort. Invalidated results are not archived — they are removed.
- **Approved extension in force:** Result remains under Preview status for the extended period.

#### 7.3.6 Extensions

A **one-time extension of up to 60 calendar days** may be granted by the review chair, subject to:

- Request submitted at least **30 days before the expiration date**.
- Written explanation of the delay and a revised, specific availability date within the extended window.
- Only one extension is permitted per submission. A second extension will be denied; the result will be invalidated. The submitter may re-submit under RDI.

#### 7.3.7 Transition to Available

When a Preview system achieves commercial availability:

1. The submitter notifies the review committee via a GitHub issue on the submission thread.
2. The submitter makes a new Available submission with updated system description (`"availability_status": "available"`, `"availability_url"` pointing to a public product or ordering page).
3. The new submission follows the standard automated compliance and peer review process.
4. If the hardware or software configuration changed materially between Preview and GA, the submitter must re-run the benchmarks. Relabeling an existing Preview result as Available without re-running is not permitted if the configuration changed.
5. Upon successful review, the Available result is published in the next cycle and the Preview result is retired.

#### 7.3.8 Permitted Uses of Preview Results

Preview results may be referenced in public communications subject to the standard MLCommons footnote:

> *"MLPerf™ name and logo are trademarks of MLCommons. Results are preliminary — peer review pending. For more information, see mlcommons.org."*

All external references must use the qualified designation **"MLPerf Endpoints Preview."** Omitting the "Preview" qualifier when referencing an unfinalized result is a violation of MLCommons usage guidelines.

---
### 7.4 RDI (Research, Development, or Internal)

An **RDI** system contains one or more components that do not meet the Available or Preview criteria. There is no commitment or timeline associated with RDI status.

#### RDI Cooling-Off Period

An RDI component may not be submitted as Available or Preview until the later of:

- The cohort after next (i.e., at least two cohorts after the RDI submission), or
- **221 days** after first publication as RDI.

This cooling-off period prevents misuse of RDI status to pre-publish results on unavailable hardware and then immediately reclassify as Available.

### 7.5 Custom SKU Classification \[CUSTOM-SKU\]

#### Custom SKUs That Qualify as Available

Vendors, OEMs, and ODMs may qualify custom SKUs as **Available**, including SKUs manufactured exclusively for specific large-volume customers (e.g., custom silicon variants produced at scale for a hyperscaler or strategic customer). A custom SKU qualifies as Available if all three conditions are met:

1. The SKU has shipped to at least one customer.
2. The SKU is available to purchase by similar customers (e.g., other hyperscalers or large-volume customers) at a volume determined by the vendor, with customizations available upon request if warranted by the customer's requirements.
3. The differences between the custom SKU and any standard SKU are clearly and publicly documented by the vendor.

#### Open Question: Hardware Not Orderable by Any Comparable Customer

> [!NOTE]
> **[WG Open Item]** — How should we classify hardware that is in production at hyperscalers or strategic customers but is **not available** to any comparable customer for purchase?
>
> These systems do not qualify as **Available** under the custom SKU path above (they fail criterion 2 — no comparable customer can order them), and do not qualify as **Preview** (there may be no commitment to general availability). However, they are not prototypes or research hardware in the traditional RDI sense — the silicon is in production and shipping at volume.
>
> **Proposed options under working group consideration:**
>
> 1. **Classify as RDI** — current default under these rules. Applies the 221-day cooling-off for future Available resubmission.
> 2. **Create a new "Production" tier** — hardware that is in production but not generally orderable. This tier would sit between Available and RDI, without a cooling-off period, and with appropriate disclosure requirements.
>
> Until a decision is made, hardware shipping only to select strategic customers with no comparable ordering path defaults to **RDI** classification.

### 7.6 Results Table Content

Each results publication includes:

- Result ID, as defined in [MLPerf Endpoints Rules §8.5](endpoints_rules.md#85-result-id).
- Submitter organization and system description.
- Benchmark model.
- Division (`Standardized` / `Serviced` / `RDI`).
- Publication status (`Available` / `Preview` / `RDI`).
- Pareto curve in step-function representation.
- Key metrics at each submitted concurrency level: **System Tokens/Second** (`system_tps`), **TPS/User** (`tps_per_user`), **TTFT P90** (`ttft_p90_ms`).
- "Peer review pending" tag where applicable.

---

## 8. Post-Publication

#### Versioning and Historical Record

MLCommons maintains a complete historical record of all versions of every pareto curve. Each version corresponds to the state of the submission at a given cohort.

- The **active results page** always displays the latest finalized version of each pareto curve.
- **Older versions** of the pareto (prior to a point being superseded by a newer measurement) remain accessible and can be displayed on request, allowing users to compare performance across time.
- All historical versions are aligned to cohorts: the record shows which points were active in each `YYYY-MM-C0` / `YYYY-MM-C1` cohort.
- Superseded points are clearly labeled in the historical view with the cohort in which they were replaced.

### 8.1 Corrections

The types of corrections permitted depend on when the error is discovered.

**During peer review (weeks 1–6):**

Corrections to non-result content are permitted and encouraged:

- Documentation, README files, and system description metadata may be updated freely.
- Source code and configuration files may be corrected to fix discrepancies (e.g., a config file that does not match the actual run settings).
- Software version information and calibration writeups may be amended.

**Results may not be changed** during the peer review window. If a measurement point is found to be incorrect:

- The submitter may **withdraw** the specific faulty pareto point(s). Withdrawn points are removed from the published results and do not count toward the minimum point requirement (which may require additional points to be submitted to remain compliant).
- Alternatively, the submitter may **withdraw the entire submission** (see [§8.3](#83-withdrawal)).

**After finalization:**

Corrections to published results are **not permitted**. Errors in finalized results are handled as follows:

- If an error is discovered by the submitter, they must notify the review chair immediately.
- If the error affects the validity of one or more pareto points, those points — or the entire submission — may be **invalidated** by the review chair following the same process as a post-publication objection (see [§8.5](#85-issues-discovered-after-publication)).
- Invalidated points are removed from the active results page but remain in the historical archive with an "Invalidated" designation and a description of the error.
- Non-result corrections (documentation, metadata) after finalization require review chair approval and are published with a clear change log.

### 8.2 Withdrawal

A submitter may voluntarily withdraw their submission at any time.

- **Before finalization** (while results still carry the "peer review pending" tag): The submission is removed entirely from both the active results page and the historical archive. No record of the provisional publication is retained.
- **After finalization:** Results are removed from the active results page but remain in the historical archive with a "Withdrawn" designation.

### 8.3 Terms of Use

Any use of published results in connection with the MLPerf trademark must follow the [MLPerf Results Messaging Guidelines](https://github.com/mlcommons/policies/blob/master/MLPerf_Results_Messaging_Guidelines.adoc) and any relevant policies at https://mlcommons.org/en/policies/.

### 8.4 Issues Discovered After Publication

This section is limited to allegations of **direct fraud or misrepresentation** — a submission that knowingly reports results it did not achieve, materially misstates the system under test, or conceals a material fact from reviewers. Post-publication concerns that do not allege fraud are handled as late objections ([§6.6](#66-late-objections-post-week-6)) or audit nominations ([§10.4](#104-audit-nomination-on-reproducibility-grounds)), subject to the standing and time-window limits of [§6.6](#scope-and-standing-for-late-concerns). This section carries no time limit and is not subject to supersession by a later submission.

Any MLCommons member may raise a fraud or misrepresentation allegation via email to any MLCommons WG chair. An objection review panel — minimally the review chair plus two **neutral members** ([§2.4](#24-conflict-of-interest)) — will screen the allegation. If rejected at this stage, the chair will respond to the objector with the reasoning.

Otherwise, the chair will designate an investigator with no conflict of interest to produce a brief report confidential to the committee, which will include a response from the submitter of the disputed result.

Possible investigation outcomes:

1. The objection is not valid.
2. The result is moved to a lower division or availability status for noncompliance with the rules.
3. The result is removed due to intentional cheating or misrepresentation.

---

## 9. Dispute Resolution

> [!NOTE]
> The formal dispute resolution mechanism is under active development by the Rules Task Force and will be ratified separately. This section describes the framework that will govern MLPerf Endpoints.

### 9.1 Scope

The dispute resolution process handles:

- Objections that cannot be resolved through the standard peer review process ([§6.4](#64-objection-resolution-weeks-46)).
- Late objections on any of the grounds listed in [§6.6](#66-late-objections-post-week-6) — availability, validity, model equivalence, and division rules.
- Disagreements about rule interpretation.
- Findings referred from an audit ([§10](#10-audit-process)).

### 9.2 Escalation Path

**Trigger.** At the close of Week 6 the review chair certifies which objections remain open and escalates them. Escalation is automatic — neither party needs to request it, and no motion is required. Objections resolved or retracted during Weeks 1–6 do not reopen.

**Status of the submission during a dispute.** A submission with an escalated objection **does not finalize** until the dispute concludes. Results already published provisionally remain visible and keep the "peer review pending" tag; results under confidential review remain unpublished. Where an escalated objection is confined to identifiable measurement points, the chair may certify the remainder of the submission for finalization and hold only the disputed points — provided the submission still satisfies the minimum point and region-coverage requirements without them.

**Panel.** The chair convenes a panel consisting of the objecting party, the submitter, and at least two **neutral members** as defined in [§2.4](#24-conflict-of-interest). The parties present evidence; only the neutral members deliberate and recommend.

**Timeline.** Business days are counted as in [§6.3](#63-peer-review-weeks-13), with the same local-holiday rule.

| Step | Deadline |
|---|---|
| Chair certifies the open objections, notifies both parties and the committee, and names the panel | Within **2 business days** of the close of Week 6 |
| Each party files a written statement of position with supporting evidence | Within **10 business days** of notification |
| Panel convenes — as a meeting or an asynchronous review, at the chair's discretion | Within **15 business days** of notification |
| Neutral members issue a recommendation: uphold, dismiss, or investigate further | Within **5 business days** of the panel convening |
| Chair issues a binding decision | Within **5 business days** of the recommendation |

Where the recommendation is to investigate further, the chair appoints an investigator with no conflict of interest, and the remaining deadlines restart from delivery of the investigator's report. An investigation may extend the process **once only**.

**Non-participation.** If the submitter does not file a written statement by its deadline, the non-response penalties of [§6.3](#63-peer-review-weeks-13) continue to accrue, and at 10 business days past the deadline the submission is withdrawn. If the objecting party does not file, the objection lapses and is treated as retracted; where it was the only escalated objection, the submission proceeds to finalization.

**Backstop.** If the process has not concluded within **8 weeks** of escalation, the chair decides on the record then available. A dispute may not remain open indefinitely, and a party's failure to produce evidence is not grounds for extension beyond the single investigation extension above.

**After the decision.** Remedies are applied per [§9.3](#93-remedies). A submission that survives the dispute finalizes and is published in the next cohort for which it clears the alignment window of [§4.3](#43-submission-to-publication-alignment); one that does not is reclassified or withdrawn as the remedy directs. The late-concern window of [§6.6](#scope-and-standing-for-late-concerns) runs from that finalization, so time spent in dispute does not consume it.

### 9.3 Remedies

Available remedies include:

- Requiring the submitter to correct and re-submit specific measurement points.
- Reclassifying the submission to a different division or publication status category.
- Withdrawing specific results or the entire submission.
- Issuing a formal finding of non-compliance.

### 9.4 Appeal

Either party may appeal the review chair's decision to the Head of MLPerf within **14 days**. The Head of MLPerf's decision is final.

### 9.5 Status

> [!TIP]
> **[WG Approval Required]** — The panel composition, deadlines, non-participation consequences, and 8-week backstop in [§9.2](#92-escalation-path) are a current proposal and require ratification before they can be enforced. Still unspecified and deferred to the Rules Task Force: the **quorum and voting rule** among neutral members where a panel has more than two (the current text assumes a recommendation carries with a simple majority, with the chair deciding on a tie), **confidentiality provisions** covering the written statements and the investigator's report, and whether a **standing roster** of pre-cleared neutral members should be maintained so that panels can be convened inside the 2-business-day window.

---

## 10. Audit Process

For audit process guidelines see the [MLPerf Endpoints Audit Guidelines](MLPerf_Endpoints_Audit_Guidelines.md).

To ensure compliance and accuracy, audits are conducted on a regular cadence, combining random selection with nomination by the review committee.

### 10.1 Audit Quota

Audit Quota
* **Annual Cadence:** 8 audits per year.
* **Quarterly Breakdown:** 2 audits per quarter, divided as:
  * 1 randomly selected audit.
  * 1 nominated audit selected by vote.

### 10.2 Audit Votes

The review committee holds an **audit vote** each quarter to select that quarter's **nominated audit** ([§10.1](#101-audit-quota)). The quarter's random audit is drawn separately under [§10.3](#103-random-audit-selection) and is not voted on.

**Nomination.** During the review process a GitHub issue is opened in which submissions may be nominated for audit. Each nomination must state a reason — new hardware or software, unusual or interesting features, performance outside expectations, or similar.

- **Who may nominate.** Members of the review committee ([§2.1](#21-structure)) only, consistent with the standing rule of [§6.6](#scope-and-standing-for-late-concerns) and with reproducibility nominations under [§10.4](#104-audit-nomination-on-reproducibility-grounds).
- **Window.** Nominations on these grounds are open for **4 weeks following publication** of the result.
- Reproducibility nominations under [§10.4](#104-audit-nomination-on-reproducibility-grounds) reach the same vote, on their own grounds and within the [§6.6](#scope-and-standing-for-late-concerns) window.

**Compiling the slate.** The review committee chairs evaluate the nominations and compile the list of candidate systems at the close of the nomination window. The chairs may add any system with a new accelerator that was not nominated.

**The vote.** The committee selects one submission for audit by **ranked-choice voting, decided by simple majority**. An option "No Audit Selected This Quarter" may be added if a majority of the review committee requests it. Where no simple majority emerges, the chairs may select one candidate at random from the pool of nominations.

**Schedule, and the late-concern window.** The review chair publishes the schedule of audit votes in advance. Audit votes fix one boundary of the late-concern window: a result settles at the later of the next audit vote following finalization or 90 days after finalization ([§6.6](#scope-and-standing-for-late-concerns)). The published schedule is therefore what lets submitters and committee members determine when a given result becomes settled.

A nomination filed within its window is considered at the next audit vote. Where the 90-day floor of §6.6 extends past that vote, an unselected reproducibility nomination remains open for any further vote falling inside the window; otherwise the result settles at the close of the window and the nomination lapses.

**Hardware retention.** From nomination, the submitter must keep the benchmarked system in its submitted configuration and available to an auditor. If the vote passes without selecting it, the obligation ends; if it is selected, retention continues until the audit is complete ([§10.5](#105-audit-compliance-and-resolution-rules)).

> [!NOTE]
> **[WG Open Item]** — The **quorum** for an audit vote is undecided. The cadence (quarterly, per [§10.1](#101-audit-quota)), the voting rule (ranked choice, simple majority), and the chairs' authority to add new-accelerator systems are settled above.

### 10.3 Random Audit Selection

Random Audit Selection
* **Timing:** The audit selection process begins after the withdrawal deadline.
* **Rules for Random Audit Exclusion:**
* A submission is not a candidate for the randomly chosen audit if the system is equivalent to a system audited in the previous round. For the purposes of this rule, equivalent systems have the same CPU, NIC, accelerator, and accelerator count, with the same configuration of those components as per the system configuration JSON. The review committee may determine that additional systems are equivalent to those audited in a previous round and exempt them from random audit. As a guidance for this exemption, if an accelerator is audited in one of the previous rounds, then the systems using the same accelerator can be excluded from random audit, if the aggregate system performance and the performance per accelerator are not more than 10% from those submitted during last audit time. For systems with power metrics, in addition to the performance, power efficiency must also be within 10% from the last audit time to be eligible for an exclusion from random audit. If any new result like a new model, an additional non-inferred scenario measurement or a new power measurement is submitted from the last audit time, then the exclusion is not applicable unless the review committee decides otherwise.
* **Selection Mechanism:**
  * A round is randomly selected with a probability of 1/6 (e.g., rolling a 6-sided die).
  * Once a round is chosen, a submission is selected from the corresponding cohort (the set of submissions in that round) using a uniform probability of selection.
  * **Proposal: Avoiding streaks of round selection (Needs WG Approval)**
    * Reroll-on-repeat: If consecutive round is selected, a six sided die will be rolled again.
      * For a selection streak of length 2, a 6-sided die would be rolled twice and the result of the second die roll will be accepted.
      * For a selection streak of length 3, a 6-sided die would be rolled thrice and the result of the third die roll will be accepted.
      * After a selection streak of length 3, there is budget for only one more random audit in the annual budget. The audit committee will make decision on how to proceed.
      * A non-selection streak has higher probability. After a non-selection streak of length 3, the audit committee will decide based on the available annual budget and the time left in the current year. The audit committee has the final authority on all the audit decisions.

* **Hardware retention:** From selection until the audit is complete ([§10.5](#105-audit-compliance-and-resolution-rules)), on the same terms as [§10.2](#102-audit-votes).

### 10.4 Audit Nomination on Reproducibility Grounds

Reproducibility concerns arising after the peer review window are not eligible as late objections ([§6.6](#66-late-objections-post-week-6)). A member who cannot reproduce a published result instead makes their case by **nominating the submission for audit**.

Nominations are subject to the standing, time-window, and supersession limits of [§6.6 Scope and Standing for Late Concerns](#scope-and-standing-for-late-concerns): only review committee members may nominate, a result is eligible only until the later of the next audit vote or 90 days after finalization, and a more recent finalized result on a similar system supersedes the concern. A nomination must:

- Identify the specific measurement points or results in question.
- Describe the reproduction attempt — the configuration used, the hardware and software stack, and the deviation observed against the published result.
- Demonstrate that the deviation exceeds the margins of [§6.6 Reproducibility Expectations](#reproducibility-expectations): 10% in general, or 5% when re-running on the exact same system. A failure to meet the accuracy quality target requires no margin showing, as accuracy is a hard gate.
- Reference the applicable rule or section.

Nominations go to the review chair, who screens them together with at least two **neutral members** ([§2.4](#24-conflict-of-interest)) — the same panel composition as [§8.5](#85-issues-discovered-after-publication). If the nomination is accepted, the submission enters the audit queue subject to the audit capacity in force for that quarter. The chair notifies the submitter and the nominating member of the decision, with reasoning where a nomination is declined.

Where an audit substantiates the concern, remedies follow [§9.3](#93-remedies) and [§8.5](#85-issues-discovered-after-publication): correction and re-submission of the affected points, reclassification to a different division or publication status, withdrawal of specific results or the entire submission, or a formal finding of non-compliance.

> [!NOTE]
> **[WG Open Item]** — Two questions remain open: whether nominated audits count against the proposed 2-per-quarter capacity or are additional to chair-selected audits, and whether a nominating member bears any share of the audit cost. A nomination route with no capacity guarantee may in practice defer indefinitely.

### 10.5 Audit Compliance and Resolution Rules

Audit Compliance and Resolution Rules

An audit is expected to be completed within a 60 day period. Audits failing to meet this timeline can be requested to be invalidated by the auditee. The final decision to accept such a request will be taken by the Working Group.

If a submitter chosen for an audit finds it unfair, they can appeal to the MLCommons Executive Director to ensure fairness.

An auditor shall be chosen by the review committee who has no conflict of interest with the submitter. The process of auditor selection will take no more than 28 days from selection of the submitter.

The burden is on the submitter to provide sufficient materials to demonstrate that the submission is compliant with the rules. Any such materials, including software, documentation, testing results and machine access will be provided to the auditor under NDA.

The submitter shall provide two days of hardware access, at a time mutually agreed with the auditor. The first day will be used to run a pre-agreed list of tests, and to verify other system parameters if needed. The second day will allow the auditor to run additional tests based on outcome of the first day.

The auditor shall write a report describing the work that was performed, a list of unresolved issues, and a recommendation on whether the submission is compliant.

The submitter will provide the auditor an NDA within seven days of the auditor’s selection. The auditor and submitter will negotiate and execute the NDA within 14 days of the auditor’s selection.

The auditor will submit their report to the submitter no more than thirty days after executing all relevant NDAs. The submitter will make any necessary redactions due to NDAs and forward the finalized report to the review committee within seven days. The auditor will confirm the accuracy of the forwarded report.

Submissions that fail the audit at a material level will be moved to the RDI division or removed, by review committee decision. If a submission failed an audit that was delayed past publication, then any published material concerning the invalidated result is subject to the MLCommons rules for Violation Determination, Remedies and Penalties for remedial action.

MLCommons shall retain a library of past audit reports and send copies to MLCommons members, auditors, and potential auditors by request. Audit reports will not be further distributed without permission from the audited submitter.

---

## 11. Appendices

### 11.1 Committee Non-Disclosure

*[TODO] — Inherits from [General Submission Rules §9.1](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#committee-non-disclosure).*

### 11.2 Submitter Non-Disclosure

*[TODO] — Inherits from [General Submission Rules §9.2](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc#submitter-non-disclosure).*

### 11.3 Submission Checklist

*[TODO] — Endpoints-specific submission checklist (supplement to General Rules §9.3).*

### 11.4 Review Chair Checklist

*[TODO] — Endpoints-specific review chair checklist (supplement to General Rules §9.5).*
