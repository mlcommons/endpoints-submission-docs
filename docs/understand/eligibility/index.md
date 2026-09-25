# Membership, PRISM and eligibility

| Concept | What it gets you | Who needs it | Where it happens |
|---|---|---|---|
| [**MLCommons CLA**](cla-process.md) | The right to contribute code and submit results | The **individual** making the submission | MLCommons CLA process |
| [**PRISM account + API token**](prism-api-key.md) | Authentication for every CLI command | The individual making the submission | MLCommons Member Central → API Keys |
| [**MLCommons membership**](membership.md) | Organisational standing; review-committee eligibility | Your organisation | MLCommons |

## What membership does affect

**Getting on the review committee.** The committee is drawn from organisations with a recent
*finalized* MLPerf Endpoints result ([Submission Rules §2.1][srules-2.1]). You earn a committee seat
by submitting, not by joining.

**Challenging other submissions.** Only committee members can raise a late objection or nominate a
submission for audit; any MLCommons member can report suspected fraud ([Scope and Standing for Late
Concerns][srules-scope-and-standing-for-late-concerns]).

**Failing to review costs you the seat.** Miss an assigned review and your organisation is off the
committee until it has a newly finalized result
([§2.6][srules-failure-to-complete-an-assigned-review]).

## Licensing

Submitted code is made under the MLCommons CLA and is Apache-2-compatible ([§5.4][srules-5.4]). Plan
for your `src/` directory (endpoint interface code, infrastructure setup, client harness) to be
published after finalization. For the Standardized division this is not optional; reproducibility by
a third party is a requirement.

The full pre-flight list is on [Before you begin](../../workflow/before-you-begin.md).

**Next:** [Before you begin](../../workflow/before-you-begin.md)

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
