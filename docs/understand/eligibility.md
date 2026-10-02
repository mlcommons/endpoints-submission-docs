# Membership, PRISM and eligibility

What you need in place before you can submit anything. Three separate things often get confused
with each other.

## The three things

| Concept | What it gets you | Who needs it | Where it happens |
|---|---|---|---|
| **MLCommons CLA** | The right to contribute code and submit results | The **individual** making the submission | MLCommons CLA process |
| **PRISM account + API token** | Authentication for every CLI command | The individual making the submission | MLCommons Member Central → API Keys |
| **MLCommons membership** | Organisational standing; review-committee eligibility | Your organisation — see the caveat below | MLCommons |

## What the rules actually require

The Submission Rules are explicit on one point and quiet on another.

!!! success "Confirmed: the CLA applies to the individual, not the whole organisation"
    The individual making the submission must have signed the relevant MLCommons CLA. It is **not**
    required that every member of their organisation has signed. All submitted code is made under
    the CLA and will be Apache-2-compatible; third-party libraries need not be Apache 2.

!!! success "Confirmed: registration *is* getting a PRISM token"
    The eight-week advance registration requirement from the general MLPerf rules is explicitly
    **overridden** and does not apply to rolling submissions. Registration is:

    1. Create an account at **MLCommons Member Central** using your **organisation email address**.
    2. You are granted API-creation access and notified by email.
    3. Sign in and create an API key with **Service Scope: MLPerf Endpoints** from the API Keys
       dashboard.

    That key, which looks like `mlc_…`, is what every `endpoints-submission-cli` command uses to
    authenticate. There's no separate "register for the round" step, because there are no rounds.

!!! question "Unconfirmed: must your organisation be an MLCommons member?"
    **The rules do not say.** They require the CLA from the individual and describe registration
    "at MLCommons Member Central using your organization email id". The name *Member Central*
    implies membership; no clause states it as a precondition for submitting.

    Treat this as unresolved. If your organisation isn't already an MLCommons member,
    [ask them](../help/support.md) before planning a submission, because the answer could add a lot
    of lead time. Tracked as **A1** in [Open questions](../help/open-questions.md).

## What membership does affect

Two things are clear regardless of how that resolves.

**Getting on the review committee.** The committee for a cohort comes from organisations with at
least one *finalized* MLPerf Endpoints result in the previous 6 months or 12 cohorts, whichever is
longer. Results still tagged "peer review pending" don't count, and neither do finalized results
that were later withdrawn. In other words, you earn a committee seat by submitting, not by joining.

**Challenging other submissions.** Only review committee members can raise a late objection or
nominate a submission for audit. Any MLCommons member can report suspected fraud or
misrepresentation, which is a separate route with no time limit.

**Failing to review costs you the seat.** An organisation that does not complete an assigned review
by the close of the peer review window is removed from the committee, and becomes eligible again
only once it has a newly finalized result.

## Licensing

All submitted code is made under the MLCommons CLA and will be Apache-2-compatible. Plan for your
`src/` directory (endpoint interface code, infrastructure setup, client harness) to be published
after finalization. For the Standardized division this is not optional; reproducibility by a third
party is a requirement, not a courtesy.

## What you still need before you can submit

- [ ] The individual submitter has signed the MLCommons CLA
- [ ] A PRISM account exists, created with an organisation email
- [ ] An API key with Service Scope **MLPerf Endpoints** has been minted
- [ ] Organisation membership question (**A1**) resolved with MLCommons if applicable
- [ ] `gh` CLI installed and authenticated — required for create, update and withdraw

The full pre-flight list is on [Before you begin](../workflow/before-you-begin.md).

!!! warning "Two URLs are missing from every source"
    Neither the PRISM portal URL nor the Member Central URL appears in the rules, the CLI
    documentation, or the public MLCommons pages. Nor is there a stated turnaround time for
    API-creation access being granted. Ask MLCommons for both, and do not assume the token is
    available same-day. Tracked as **A3** and **A4** in [Open questions](../help/open-questions.md).

**Next:** [Before you begin](../workflow/before-you-begin.md)

--8<-- "precedence-notice.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
