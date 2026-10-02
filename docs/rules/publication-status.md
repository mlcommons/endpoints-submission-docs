# Publication status

Available, Preview or RDI. This says whether your hardware and software can be **bought today**,
and is independent of which [division](../understand/divisions-and-scenarios.md) you submitted in.

--8<-- "precedence-notice.md"

| Category | Hardware | Software |
|---|---|---|
| **Available** | All performance-determining components pass the four-point test at submission time | Available software stack |
| **Preview** | Not yet Available; you commit to Available within **180 days** | Available, except software supporting substantially new hardware |
| **RDI** | Meets neither | Meets neither |

!!! warning "RDI has a cooling-off period"
    An RDI component may not be submitted as Available or Preview until the later of **two cohorts
    after** the RDI submission, or **221 days** after first publication as RDI. This prevents RDI
    being used to pre-publish on unavailable hardware and immediately reclassify.

## Available: the four-point test

Every component that substantially determines ML performance must meet all four at submission time:

| # | Criterion | Requirement |
|---|---|---|
| 1 | **Pricing** | Publicly advertised or available on request |
| 2 | **Shipment** | Shipped to or rented by at least one third party |
| 3 | **Public evidence** | Externally verifiable evidence it is shipping or purchasable **today** |
| 4 | **Reasonably available** | Purchasable or rentable by additional third parties by the submission date |

Availability is judged on the **balance of evidence**. No single data point settles it.
Uneven availability across geographies or channels is not penalised, provided availability exists in
at least one market.

### Proof of shipment, not announcement

!!! danger "These are not sufficient on their own"
    - Announcements at a conference, trade show or keynote
    - Product listings on a marketing or "coming soon" page
    - Soft-launch press releases describing a future availability date
    - A "Preview" or "coming soon" entry in a cloud provider's catalogue
    - A stated or anticipated launch date without proof of current availability

**Sufficient evidence:**

- A vendor product or pricing page with a live, functional call to action for qualifying customers —
  "Buy now", "Order", "Get started" or equivalent
- Public vendor claims that the product is "now shipping", "generally available" or "in production",
  in a press release, earnings call or official blog post
- Publicly verifiable fulfilment data — a cloud instance type in live pricing and availability APIs,
  orderable by any qualifying customer
- Third-party review units or press loaners confirmed by the recipient
- Public purchase orders or shipment confirmations in securities filings

For **component vendors** (accelerators, ASICs, memory, networking) the test is applied at the
component's position in the supply chain: available to ship to system integrators, OEMs and ODMs
under standard commercial terms.

!!! warning "Your own GA definition also binds you"
    If your organisation has a formal internal process defining general availability ("open
    shipping status", "phase 4 exit", "FCS") the submitted system must satisfy that internal
    definition **in addition to** every criterion above.

### Reasonably available

1. **Non-discrimination.** Competitors must not be blocked. All software, drivers, firmware and
   support made available to any customer must be available to any other qualifying customer,
   including direct competitors, on equivalent terms.
2. **Access conditions.** Conditions common to generally available products are fine: financial
   qualification, customer size, support burden, export restrictions. "Early access" approval
   requirements are not.
3. **Supply and lead times.** Subject to market conditions, and not governed by these rules.
   Reproducing an MLPerf result gives you **no priority slot or allocation right**. A reviewer or
   auditor faces the same supply constraints as any other customer.

Qualifying pre-submission rentals or purchases *are* allowed to have been made under "early access"
restrictions.

### Available software stack

| Category | Rule |
|---|---|
| ML frameworks (PyTorch, JAX) | Any commit in an official public repository. Open PRs adding architecture support are fine if publicly accessible |
| Inference servers (TensorRT-LLM, vLLM, SGLang, Triton) | Open-source: any public commit plus accessible open PRs. Commercial: official release or labelled beta in the formal release sequence |
| Accelerator compute libraries (cuDNN, NCCL, ROCm HIP) | Official release or publicly available labelled beta. **One-off private binary drops do not qualify** |
| Hardware drivers | Official release or public beta, downloadable through a standard vendor channel at submission time |
| Quantization tools | As frameworks if open-source; as compute libraries if closed-source binary |
| Custom OS / kernel patches | Exempt if unmodified commodity software. Patches that substantially affect ML performance must be upstream-merged or in a publicly accessible PR. **Private patches are not permitted** |
| Your own submission code and custom kernels | Not part of the "stack" — must be in your submitted source package |

**Labelled beta** qualifies only if: the vendor has publicly committed the optimisations will ship
in a future official release; the beta is a standard step in the release process, not a one-off
private engagement; and it is publicly downloadable at submission time. A release candidate
qualifies; an NDA engineering sample does not.

Software must be available at the time of **submission**. If a component becomes unavailable between
submission and publication, tell the review committee. They may let it proceed if an
equivalent public release exists, or require reclassification.

### Available models

Weights and tokenizer must be available through commercially accessible channels. **Weights under a
licence prohibiting commercial deployment don't qualify**, and must be submitted as Preview or RDI.

### Division-specific additions

**Standardized CoN** — the endpoint URL must stay accessible for at least **90 days** after
publication, with a point of contact for access requests.

**Serviced** — the endpoint must be at **full GA tier**. Not "Public Preview", not "Open Beta", not
"Generally Available in Preview", even if publicly accessible. Where your own GA definition
distinguishes preview-accessible from production-GA, production-GA is required. The terms of service
must permit third-party benchmarking; any MLCommons member in good standing must be able to access
the endpoint under standard terms and attempt reproduction.

**RDI division** — always carries RDI status and cannot qualify as Available or Preview. If the
system later becomes commercially available, make a new Standardized or Serviced submission.

## Preview, and its clock

A Preview system is not Available at submission, but you **commit** to making it Available within
**180 days** of first publication and to re-submitting as Available at that point.

| | |
|---|---|
| Clock starts | The date of the cohort the result **first appears in** — not the submission date |
| Duration | 180 calendar days |
| Anchor | First publication. Corrections and amendments do **not** reset it |

Results publish tagged **"Preview — Available by [date]"**.

### Declaring Preview

1. Set `"availability_status": "preview"` in the system description
2. State a `"target_availability_date"` within the window, as an ISO 8601 date
3. Identify **with specificity** which components are not yet available — *"The X100 GPU is expected
   to begin customer shipments in Q3 2026"*. Vague statements like *"targeting H2 2026"* are **not
   sufficient**

### Software waiver

The Available software stack requirement is waived for software supporting **newly developed
hardware** that substantially determines ML performance. "Newly developed" means the hardware was
not Available as of the previous cohort and was not submitted as Preview in that cohort. All other
components must still meet the Available requirements.

### Performance continuity

When Preview transitions to Available, the re-submitted result must be **equal or better**.

- **Throughput** — `system_tps` and `tps_per_user` may each degrade by up to **5%**, applied
  independently, to account for endpoints-workload variance.
- **Latency** — the 5% tolerance does **not** apply to `ttft_p90_ms` or any latency percentile. Until
  a comparison method is ratified, a transition is not blocked on latency alone; a material
  unexplained latency regression is raised as an objection during the re-submission's own review.

### Expiry and extensions

At the end of the 180 days:

| Situation | Outcome |
|---|---|
| Available re-submission published | Preview result is superseded |
| No re-submission, no extension | Preview result is **invalidated and removed at the next cohort** — not archived |
| Approved extension in force | Remains Preview for the extended period |

!!! danger "One extension only"
    A single extension of up to **60 calendar days** may be granted by the review chair. It must be
    requested **at least 30 days before expiry**, with a written explanation and a revised, specific
    date. A second extension is denied and the result is invalidated. You can re-submit under RDI.

MLCommons maintains a public **Preview Availability Tracker** listing active Preview results, their
first publication dates, target dates and days remaining, updated each cohort.

### Transitioning to Available

1. Notify the review committee via a GitHub issue on the submission thread
2. Make a new Available submission with `"availability_status": "available"` and an
   `"availability_url"` pointing at a public product or ordering page
3. It goes through standard compliance and peer review
4. **If the hardware or software configuration changed materially, you must re-run.** Relabelling an
   existing Preview result as Available without re-running is not permitted where the configuration
   changed
5. On success the Available result publishes and the Preview result is retired

### Referencing Preview results

All external references must use the qualified designation **"MLPerf Endpoints Preview."** Omitting
the qualifier when referencing an unfinalized result violates MLCommons usage guidelines. The
standard footnote applies.

## Custom SKUs

Vendors, OEMs and ODMs **may** qualify custom SKUs as Available, including silicon variants
produced exclusively at scale for one large-volume customer, if all three hold:

1. The SKU has shipped to at least one customer
2. It is available to purchase by **similar** customers at a volume the vendor determines, with
   customizations available on request where warranted
3. The differences from any standard SKU are clearly and publicly documented by the vendor

!!! question "Hardware no comparable customer can order"
    Silicon in production at a hyperscaler but not orderable by any comparable customer fails
    criterion 2, and may have no GA commitment so can't be Preview, yet it isn't a prototype
    either. The working group is weighing a new "Production" tier against the current default.
    **Until decided, it defaults to RDI**, with the 221-day cooling-off attached. Tracked upstream as
    `[CUSTOM-SKU]`.

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
