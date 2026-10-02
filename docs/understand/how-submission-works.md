# How submission works

What you do, what MLCommons does, what reviewers do, and when your results become public.

## Overview

```mermaid
flowchart TD
    A[Plan the curve<br/>choose C_max, compute regions] --> B[Run 7-32 measurement points<br/>plus the Offline point]
    B --> C[Author system_desc.json and point.yaml<br/>per point, system_power.json per system]
    C --> D[Register each run<br/>runs create]
    D --> E[Assemble + validate<br/>submissions create]
    E -->|checker errors| C
    E -->|checker passes| F[Bundle uploaded<br/>peer-review PR opened]
    F --> G[Week 0<br/>automated compliance]
    G -->|fails| X([Rejected<br/>correct and resubmit as new]):::bad
    G -->|passes| H[Weeks 1-3<br/>peer review, objections filed]
    H --> I[Weeks 4-6<br/>objection resolution]
    H -->|all objections resolved| K
    I -->|objection needs a fix| C
    I -->|unresolved at Week 6| J[Dispute resolution<br/>~5 weeks, chair decides]
    I --> K[Finalized<br/>peer review pending tag removed]
    J --> K
    H -.->|no response for<br/>10 business days| Y([Withdrawn]):::bad
    K --> L[Published in the next cohort<br/>1st or 3rd Wednesday]
    classDef bad stroke:#c62828,stroke-width:2px;
```

## Rolling submission and cohorts

There is no submission deadline: submit on any day and the submission enters the pipeline on receipt
([§4.1][srules-4.1]). Results publish in **cohorts** on the 1st and 3rd Wednesday of each month
(`YYYY-MM-C0` and `YYYY-MM-C1`, [§4.2][srules-4.2]).

To make a cohort, your submission has to pass automated compliance at least one business day before
that Wednesday, and every review clock counts from the cohort you first appear in, not the day you
uploaded ([§4.3][srules-4.3]).

## The review phases

The windows, from [§6.5][srules-6.5]:

| Phase | Window | What happens |
|---|---|---|
| **Automated compliance** | Week 0 (may finish in a day) | Any failure **rejects** the submission; you resubmit as a *new* one |
| **Peer review** | Weeks 1–3 | Objections filed as GitHub issues. **None new after Week 3.** Resolve everything here for early finalization |
| **Objection resolution** | Weeks 4–6 | Carried-over objections must be resolved |
| **Dispute resolution** | From Week 6, ~5 weeks | Automatic escalation for anything still open; the chair decides |

### Your response deadline

This is easy to miss. Once someone files an objection against you, you have **3 business days** to
post an initial response, either a resolution schedule or a counter-argument with evidence
([§6.3][srules-6.3]). Miss it and penalties apply automatically. They add up and can't be reversed:

| Business days without your response | Penalty |
|---|---|
| 3 | Finalization delayed by **1 cohort** |
| 6 | Delayed by **2 cohorts** |
| 10 | Submission is **withdrawn** |

See [After you submit](../workflow/after-submission.md).

## Who reviews you

Organisations with a *finalized* Endpoints result in the lookback window form the review committee
([§2.1][srules-2.1]), and each submission also gets one randomly assigned reviewer from that pool,
never from your own organisation ([§2.6][srules-2.6]). Being a competitor is **not** a conflict of
interest ([§2.4][srules-2.4]).

## When your results become public

You pick one of three publication modes when you submit, and you **can't change it afterwards**
([§6.2][srules-6.2]):

| Mode | Public before finalization? | Peer review starts | Finalized result goes public |
|---|---|---|---|
| **Confidential review** *(default)* | No | When automated checks pass | The first cohort after finalization |
| **Confidential review, embargoed** | No | When automated checks pass | On your embargo date |
| **Provisional publication** | Yes, tagged **"peer review pending"** | At provisional publication | The tag is removed at finalization |

Use the embargoed mode to time a launch without early numbers going out, and provisional publication
when you need numbers public before review ends. An embargo under provisional publication also
delays the start of your review. The details of each mode are in [§6.2.1–6.2.3][srules-6.2.1], and
who can see what during review is in [§6.10][srules-6.10].

## After publication

Published results can't be edited; an error gets the affected points or the submission invalidated
([§8.1][srules-8.1]). Others can challenge a result until the **later** of the next audit vote or
**90 days** after finalization, and you keep the system available for audit until then ([Scope and
Standing for Late Concerns][srules-scope-and-standing-for-late-concerns]).

## Audits

MLCommons runs **8 audits a year**: each quarter one drawn at random and one chosen by committee
vote from nominations ([§10][srules-10]). The thing to plan for: **you have to keep the system as
submitted from the moment it's nominated**, not only once it's picked ([§10.2][srules-10.2]). What
an audit asks of you, and what happens if you fail, is in [§10.5][srules-10.5]. What the auditor
checks is in the [MLPerf Endpoints Audit Guidelines][audit].

**Next:** [Divisions and scenarios](divisions-and-scenarios.md)

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
