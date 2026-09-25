# Getting support

## Where to ask

| Channel | Use for |
|---|---|
| [MLPerf Endpoints working group](https://form.asana.com/?k=OHLBqTsKzjNIoCi4LgwjNQ&d=1203603044984020) | Joining the group — the primary route for rule questions and proposals |
| `endpoints@mlcommons.org` | General questions about the benchmark and submitting |
| [`mlcommons/endpoints_policies` issues](https://github.com/mlcommons/endpoints_policies/issues) | Rule ambiguities, errors in the rules, v1.0 proposals |
| [`mlcommons/endpoints-submission-cli` issues](https://github.com/mlcommons/endpoints-submission-cli/issues) | Bugs in the submission CLI or checker |
| [`mlcommons/endpoints` issues](https://github.com/mlcommons/endpoints/issues) | Bugs in the reference benchmark client |
| Your submission's review thread | Anything about a submission already in review |

!!! note "No response times are published"
    None of the sources state a support SLA, or a turnaround for [PRISM API-creation
    access](../understand/eligibility/prism-api-key.md). Build slack into your plan rather than
    assuming same-day answers.

## What to include when asking

A question with these attached gets answered once instead of three times:

- **Which division and scenario** you are submitting under
- **Versions** — the `endpoints` commit SHA you built from, and `endpoints-submission-cli --version`
- **The exact error text**, not a paraphrase
- **The checker output** — `endpoints-submission-cli check-submission … --output checker.json`
- **The relevant `point.yaml`** with secrets removed
- **What you expected**, and what the rules clause you are reading says

For a run problem, `report.txt` and the non-histogram fields of `result_summary.json` are usually
enough. **Don't attach `events.jsonl`**, which runs to hundreds of megabytes.

!!! warning "Never paste your PRISM token"
    Not in an issue, not in a log, not in a config attachment. If you think one has leaked, rotate
    it from the API Keys dashboard immediately. It can be used to withdraw your submissions.

## Questions this documentation cannot answer

Some things are genuinely not published anywhere. If your question is one of these, go straight to
MLCommons rather than searching further:

- Whether your **organisation** needs [MLCommons membership](../understand/eligibility/membership.md) to submit
- The **PRISM and Member Central URLs**
- The **official** v1.0 model list and accuracy targets. The
  [list on this site](../reference/benchmarks.md) is put together from the working group's
  overview and the tooling
- The **approved drafter list** for the legacy benchmarks, and when the checker will carry the agentic one
- Which **load pattern** a dedicated Offline run should use
- **CoN client locations** and scheduling
- The **Preview Availability Tracker** and public results URLs

Full list with context: [Open questions](open-questions.md).

## During review

Objections are filed and resolved **on the submission's review thread**, not through support
channels. Meetings are the exception ([Submission Rules §6.9][srules-6.9]).

If you think a reviewer has a conflict of interest, raise it with the **review chair**, at any time
during review ([§2.4][srules-2.4]).

An allegation of **fraud or misrepresentation** in a published result goes by email to any MLCommons
working-group chair ([§8.4][srules-8.4]). Anything short of fraud is a late objection or an audit
nomination instead.

## Reporting a documentation problem

This site is a guided path over the authoritative sources, and it can be wrong in ways the sources
are not.

- **If this site contradicts the policy repository**, the policy repository is correct and this site
  has a bug. Please report it.
- **If a page is out of date**, check its `Last verified against:` line. Pages are verified against a
  specific commit, and the upstream rules are a live development branch.
- **If something was hard to find**, that is also a bug worth reporting. A question that keeps being
  asked means a page failed to answer it.
