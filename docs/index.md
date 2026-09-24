---
hide:
  - navigation
---

# MLPerf Endpoints — Submitter Documentation

How to prepare, validate and submit an MLPerf&reg; Endpoints benchmark result.

<div class="grid cards" markdown>

-   :material-school:{ .lg .middle } **New here**

    ---

    What the benchmark measures, how a submission moves through review, and which division you
    belong in.

    [:octicons-arrow-right-24: Start with the concepts](understand/index.md)

-   :material-rocket-launch:{ .lg .middle } **Ready to submit**

    ---

    The eight-step path from an empty directory to a submission in the review queue.

    [:octicons-arrow-right-24: Before you begin](workflow/before-you-begin.md)

-   :material-gavel:{ .lg .middle } **Rules**

    ---

    What you must meet, what gets you rejected, and which rules are still moving.

    [:octicons-arrow-right-24: Requirements](rules/requirements.md)

-   :material-console:{ .lg .middle } **Reference**

    ---

    Three CLIs, two hand-authored files, one bundle layout, every compliance check.

    [:octicons-arrow-right-24: Look something up](reference/index.md)

</div>

## Round status

--8<-- "round-status.md"

There is **no submission deadline**. Submissions are timestamped on receipt and enter review
immediately; results publish in the next cohort they clear.

## Coming from MLPerf Inference?

Six things work differently here:

| | MLPerf Inference | MLPerf Endpoints |
|---|---|---|
| Schedule | Fixed rounds, advance registration | Rolling; register by holding a PRISM token |
| Result | A score at one operating point | A **Pareto curve** of 7–32 points, plus an **Offline** result |
| Divisions | Closed / Open | **Standardized** / **Serviced** / **RDI** |
| Cross-query KV reuse | Prohibited | **Permitted**, with a per-query salt |
| TTFT percentile | — | **P90** for v1.0 (v0.7 used P95) |
| Power | Measured, optional | **Provisioned** power, declared per system; required for Standardized |

Full detail in [What is MLPerf Endpoints?](understand/what-is-mlperf-endpoints.md) and
[Model equivalence](rules/model-equivalence.md).

## Before you invest hardware time

!!! danger "The v1.0 rules are a working draft"
    MLPerf Endpoints v1.0 rules are on an active development branch, and several sections that
    determine how long your runs must be and what you must disclose are explicitly marked
    tentative or pending working-group ratification. Read
    [Open questions and WIP rules](help/open-questions.md) before you commit accelerator time.

## Authoritative sources

This site is a guided path over the sources below. It doesn't restate policy. Where this site and
the policy repository disagree, **the policy repository is correct**.

- [`mlcommons/endpoints_policies`](https://github.com/mlcommons/endpoints_policies) — all rules and process
- [`mlcommons/endpoints-submission-cli`](https://github.com/mlcommons/endpoints-submission-cli) — submission CLI and checker
- [`mlcommons/endpoints`](https://github.com/mlcommons/endpoints) — the reference benchmark client
