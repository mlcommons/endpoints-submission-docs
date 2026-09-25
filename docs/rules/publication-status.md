# Publication status

Available, Preview or RDI. This says whether your hardware and software can be **bought today**, and
is independent of which [division](../understand/divisions-and-scenarios.md) you submitted in.

--8<-- "precedence-notice.md"

The categories are defined in [Submission Rules §7][srules-7]. In short ([§7.1][srules-7.1]):

| Category | Hardware | Software |
|---|---|---|
| **Available** | All performance-determining components pass the four-point test at submission time | Available software stack |
| **Preview** | Not yet Available; you commit to Available within **180 days** | Available, except software supporting substantially new hardware |
| **RDI** | Meets neither | Meets neither |

!!! warning "RDI has a cooling-off period"
    An RDI component can't come back as Available or Preview until the later of two cohorts after
    the RDI submission or **221 days** after first publication as RDI ([RDI Cooling-Off
    Period][srules-rdi-cooling-off-period]). Don't use RDI as a way to publish early.

## Available: the four-point test

Pricing, shipment to a third party, public evidence that it ships **today**, and reasonable
availability to other buyers, for every component that substantially determines performance, judged
on the balance of evidence ([§7.2][srules-7.2]).

### Proof of shipment, not announcement

!!! danger "Announcements don't count"
    Keynotes, "coming soon" pages, soft-launch press releases and catalogue previews are not
    sufficient on their own. What does count is listed in [§7.2.1][srules-7.2.1].

!!! warning "Your own GA definition also binds you"
    If your organisation has a formal internal definition of general availability, the submitted
    system has to meet it as well as the rules' criteria ([§7.2.1][srules-7.2.1]).

### Reasonably available

No blocking competitors and no "early access" gating; supply and lead times aren't governed by the
rules ([§7.2.2][srules-7.2.2]). Reproducing an MLPerf result gives no one a priority allocation.

### Available software stack

Each category of software (frameworks, inference servers, compute libraries, drivers, quantization
tools, OS patches) has its own availability rule in [§7.2.3][srules-7.2.3], along with when a
labelled beta qualifies. Private binary drops and private kernel patches don't. Your own submission
code isn't part of the stack; it goes in your source package.

### Available models

Weights under a licence that prohibits commercial deployment can't be Available
([§7.2.4][srules-7.2.4]).

### Division-specific additions

Standardized CoN keeps its endpoint reachable for 90 days after publication; Serviced has to be at
full GA tier on the same endpoint paying customers use, with terms that allow benchmarking; the RDI
division is always RDI status ([§7.2.5][srules-7.2.5]).

## Preview, and its clock

You commit to Available within **180 days**, counted from the cohort the result first appears in,
and corrections don't reset it ([§7.3][srules-7.3], [§7.3.1][srules-7.3.1]).

### Declaring Preview

Set `availability_status` and `target_availability_date` in the system description, and name the
unavailable components specifically. "Targeting H2 2026" isn't enough ([§7.3.4][srules-7.3.4]).

### Software waiver

The software-stack requirement is waived only for software that supports newly developed hardware
([§7.3.2][srules-7.3.2]).

### Performance continuity

The Available re-submission has to match the Preview result, with 5% allowed on throughput and no
percentage band on latency ([§7.3.3][srules-7.3.3]).

### Expiry and extensions

With no Available re-submission and no extension, the Preview result is removed at the next cohort
([§7.3.5][srules-7.3.5]).

!!! danger "One extension only"
    One extension of up to 60 days, requested at least 30 days before expiry
    ([§7.3.6][srules-7.3.6]). A second is denied and the result is invalidated.

### Transitioning to Available

A new Available submission through normal review, re-run if the configuration changed materially
([§7.3.7][srules-7.3.7]).

### Referencing Preview results

Always call them **"MLPerf Endpoints Preview"**, with the standard footnote
([§7.3.8][srules-7.3.8]).

## Custom SKUs

A custom SKU can be Available if it has shipped, similar customers can buy it, and its differences
are publicly documented ([§7.5][srules-7.5]).

!!! question "Hardware no comparable customer can order"
    Silicon in production at a hyperscaler but not orderable by any comparable customer is an open
    working-group item. **Until decided, it defaults to RDI**, with the 221-day cooling-off
    attached. Tracked upstream as `[CUSTOM-SKU]` ([Open
    Question][srules-open-question-hardware-not-orderable-by-any-comparable-customer]).

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
