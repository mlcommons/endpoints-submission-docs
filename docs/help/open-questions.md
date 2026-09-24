# Open questions and WIP rules

What could still change under you, and what this documentation could not confirm. Read this before
committing accelerator time.

--8<-- "draft-rules-warning.md"

**Last reviewed:** 2026-09-24, against `endpoints_policies@v1.0_rules_dev` (6b0b1ef),
`endpoints-submission-cli@main` (f25f71e, tag `v1.0.1.0`), `endpoints@main` (e71b928).

??? info "What changed at the 2026-09-24 review"
    The rules moved 36 commits since the 2026-09-19 review. Five merged changes matter to a
    submitter:

    - **An Offline point is now required** for non-agentic benchmarks (new §5.7). The minimum goes
      from 7 points to 8, or stays at 7 if you elect your `C_max` point, and accuracy goes from four
      points to five. This defines what the old **C7** asked about; C7 now tracks what's still
      open about Offline.
    - **Power normalization** (new §4.5). Standardized submissions need a `system_power.json` per
      system and publish `system_tps_per_kw`. Introduces **B10** and **B11**.
    - **Speculative decoding moved to approved drafter lists** per benchmark (§2.9.4). No list is
      published, which is **C10**.
    - **Three publication modes** (§6.2). An embargo on a provisional submission now also delays
      the start of review.
    - **The audit process is defined** (§10): 8 a year, one random and one voted each quarter,
      with retention obligations from nomination.

    On the tooling side, checker `v1.0.1.0` (2026-09-23) implements all of the above, and resolves
    **B4**. The steady-state detector reached `endpoints@main` as an ad-hoc script, which
    narrows **B8** and **B9**.

## A. Registration and membership

| # | Question | Status |
|---|---|---|
| **A1** | Must a submitter's **organisation** be an MLCommons member, or is the individual's CLA enough? | The rules require the CLA from the individual and describe PRISM registration at "MLCommons Member Central using your organization email id". Membership is never stated as a precondition. **Unresolved — confirm with MLCommons.** |
| **A2** | Is there a separate "register for the round" step? | The eight-week advance registration is explicitly overridden and there are no fixed rounds. Reading: a PRISM account plus a scoped API key is the whole of registration. **Confirm.** |
| **A3** | What are the **PRISM and Member Central URLs**? | Neither appears in any rules document, CLI doc, or public MLCommons page. **Needed.** |
| **A4** | Who grants API-creation access, and how long does it take? | The rules say an email notification follows account creation. No turnaround is stated. **Needed** for planning. |

## B. Tooling and policy disagreements

Found while cross-reading the rules against the tooling. Each is documented at the relevant page;
none has been resolved by guesswork.

| # | Gap | Detail | This site follows |
|---|---|---|---|
| **B1** | **`system_info` config section** | The Submission Rules tell submitters to run the benchmark "with config.yml having `system_info` section (if you want to automatically capture the system description)". **No `system_info` exists anywhere in the reference client's schema**, and the CLI docs state plainly that `system_desc.json` is submitter-authored and not an endpoints artifact. Either the policy anticipates unshipped work, or the sentence is stale. | The tooling — you author `system_desc.json` by hand |
| **B2** | **Pareto updates** | The technical rules still reference a post-submission update window, for example as the reason for the 10% High Concurrency margin, and link "Submission Rules §8.1 Pareto Updates". **That section no longer exists** — §8.1 is now *Corrections*. The CLI confirms the removal: there is no `add-run`, and `update --run-ids` rejects additions. | The Submission Rules and the CLI — points are fixed at creation |
| **B3** | **Section-number drift** | The Submission Rules link "Endpoints Rules §7" for directory structure; it is §8.1. The checker's documentation cites §14/§15/§16 for metrics, accuracy and consistency; those sections do not exist. | Clause numbers as they actually are |
| **B5** | **TTFT percentile** | The public MLCommons benchmark page still advertises **TTFT P95**. The v1.0 rules require **P90** and state P95 was the v0.7 metric. | The rules — P90 |
| **B6** | **Result labels** | The rules use Available / Preview / RDI. The public page additionally shows "Verified / Provisional / Unverified". The relationship is unstated. | The rules — needs a mapping |
| **B7** | **File and field naming** | Most of the rules now say `system_desc.json`, but the §9.1 *Max concurrency declared* row and the Submission Rules still say `system_desc_id.json`, and the result-ID definition refers to a `benchmark_model` field. The tooling uses `system_desc.json` and `model_name`. Separately, `link_config` was removed from the §8.2 field table but is still in the §8.2.1 template. | The tooling spelling; leave `link_config` empty |
| **B8** | **Which metrics gate a steady-state window** | *Narrowed.* §4.4's definition table and the detector documentation now on `endpoints@main` agree: **TPOT at P50 and P90**, with TTFT a diagnostic and drift warning only. But the paragraph after that table in §4.4 still says **TTFT and TPOT at P50/P90**. | TPOT at P50/P90, but confirm |
| **B9** | **The steady-state detector isn't part of the run** | *Partly resolved.* `scripts/steady_state_diagnostics.py` and its documentation merged to `endpoints@main` on 2026-09-16 (#447). It's an ad-hoc tool you run over a run directory or `events.jsonl`. Nothing in the client writes the `steady_state` block §8.3 requires, and the tool itself prints *not yet supported* for agentic runs. The rules still link a pinned commit on the old design branch. | Run the script yourself and fill in `steady_state` from its output |
| **B10** | **`system_power.json` field names** | Rules §4.5.2 names the fields `num_cpu`, `tdp_per_cpu`, `num_accelerator`, `tdp_per_accelerator`, `num_switches`, `tdp_per_switch`. The checker reads nested groups (`cpu`, `accelerator`, `scale_up_network`, `scale_out_network`), each with `count`, `tdp_per_unit` and `link`, plus `provisioned_power_w` and `overhead_fraction`. §4.5.2 publishes no literal schema. | The checker's shape |
| **B11** | **Who needs `system_power.json`** | §4.5's scope makes power normalization mandatory for Standardized, optional for RDI and deferred for Serviced. The same section's callout says *every submission* must include the file, and §9.1's *Power descriptor* row says Standardized. The checker requires it for every system regardless of division. | Include it whatever your division |
| **B12** | **The checker can't tell agentic from single-turn** | Nothing the checker reads says whether a benchmark is agentic. So when no point declares `offline`, `offline-point-present` only **warns**, and `point-count` applies the 7-point minimum. For a non-agentic benchmark the rules reject that submission. Ties to **C8**. | The rules — count your Offline point yourself |
| **B13** | **"One accuracy run for each of the 5 pareto regions"** | §4.3 now opens with that sentence. There are four regions plus the Offline point, and agentic benchmarks need only four accuracy results. The rest of §4.3 and §5.3 are consistent with each other: `N` = 5 non-agentic, 4 agentic. | §5.3 — five or four points |

??? success "Resolved at the 2026-09-24 review"
    | # | Was | Resolution |
    |---|---|---|
    | **B4** | Seed-set adoption couldn't be checked: the checker's mirror had no cohort keys, so `seed-set-adoption` reported SKIP | Checker `v1.0.1.0` mirrors the published `seedset.yaml`, `cohort-id: 2026-10-C1` included, and derives the four-cohort adoption window from it. Bind set `A`, target `2026-10-C1`, and upgrade the checker |
    | **Checker accuracy gap** | The checker only tested *at least one* accuracy result | `accuracy-coverage` in `v1.0.1.0` requires one in each mandatory region and at the Offline point |
    | **Agentic metric check** | §9.1's *Agentic metric consistency* row had no checker rule | `agentic-metric-consistency` in `v1.0.1.0` |

## C. Content MLCommons must still supply

None of this is available in any source we could find.

| # | Item | Why it blocks you |
|---|---|---|
| **C1** | The **v1.0 supported model list** and canonical model IDs | The rules defer to "the reference repository", published ≥ 6 weeks before a round. The client ships a ruleset named `mlperf-inference-v6.1`, which is not an Endpoints ruleset name. `model-name-valid` checks against this list |
| **C2** | **Per-benchmark accuracy targets and tolerances** | Marked `[WIP]` upstream. `accuracy-gate` is a hard reject and you cannot predict whether you pass |
| **C3** | **Dataset identities and download paths** for performance and accuracy runs | You cannot run without them. For a dedicated Offline run, the dataset's size is also your Offline concurrency |
| **C4** | **CoN client locations and scheduling procedure** | Deferred to a separate working-group publication that does not yet exist. CoN submitters cannot plan |
| **C5** | **The full submission-state list** | Only `REVIEW_PENDING`, `WITHDRAWN`, `FINALIZED` and `PUBLISHED` are documented |
| **C6** | **Preview Availability Tracker URL** and the public results/visualizer URL | Referenced by the rules; no URL given |
| **C7** | **The Offline point's loose ends** | §5.7 defines the Offline point, but several things you need to run one aren't settled. **Load pattern:** the rules say "the benchmark-defined Offline load pattern" and name no client setting; the reference client's `max_throughput` matches the definition, unconfirmed. **`region` value:** unspecified for a dedicated run; `submitters_choice` passes the checker. **Dataset smaller than `C_max`:** the Offline concurrency is the dataset size and must be ≥ `C_max`, so a large `C_max` can't satisfy both. Open upstream as `[OFFLINE]` item 2. **Steady state:** §4.4's scope excludes a dedicated Offline run, which implies `total` metrics; not stated outright. **Duration:** §6.2 applies unchanged, though the run ends when the queue drains. **Pass boundaries:** the event log doesn't mark them, so the no-mixing rule is a manual review item |
| **C8** | **Which benchmarks are agentic, and which are multi-turn** | This now decides a lot: whether you need an Offline point, how many points and accuracy runs you owe, the accuracy gate (every point vs mean-of-N), the primary chart (`tps_per_user` vs `e2e_avg_interactivity`), and whether agentic salting flags apply. None of it is mapped to a benchmark. Ties to **C1** and **B12** |
| **C9** | **Super-pass size per benchmark** | The steady-state window is measured in super-passes, defaulting to one full dataset pass "unless the benchmark definition specifies a different super-pass size". No benchmark definition is published, so you cannot compute your own floor |
| **C10** | **Approved drafter lists** | §2.9.4 allows speculative decoding only with a drafter on the benchmark's published list. None has been published, the checker ships an empty one, and a drafter approved now can only be used two cohorts later. Until a list appears, speculative decoding isn't available for any benchmark |

## D. Rules the working group has not ratified

Marked upstream as `[TENTATIVE]`, `[WIP]`, `[WG Open Item]` or `[WG Decision Required]`. These are
current policy where stated, but are the most likely to move.

### Likely to affect your run plan

| Area | What is unsettled |
|---|---|
| **Run requirements** | The entire run-requirements section is under active development. Minimum durations (600 s / 1,200 s), minimum query counts, dataset-subset rules, and the warmup model — submitter discretion plus mandatory disclosure, in place of a fixed warmup duration — are all pending ratification, though stated as locked and Task Force-approved for the v0.7 round |
| **Offline point** | §5.7 is new for v1.0 and marked pending ratification. Agentic Offline is deferred to a later version. See **C7** for the details that aren't settled |
| **Accuracy targets** | Per-benchmark tolerance values are `[WIP]` |
| **Steady-state reporting** | §4.4 carries its own pending-ratification note: whether the 4 super-pass floor rises, and whether a run where no steady state is found is declared **invalid** rather than merely reported-with-flags. The second matters most — it decides whether a run that fails detection is a failed run or just a weaker number. See also **B8** and **B9** |
| **Reproducibility margins** | The 10% and 5% throughput margins are proposals requiring ratification before they can be enforced |
| **Latency comparison method** | **None exists.** A reproducibility objection may not rest on latency alone, and a Preview-to-Available transition is not blocked on latency alone. The working group is weighing histogram-based methods, which would first require the per-metric histogram to become a required artifact |

### Likely to affect your power figure

| Area | What is unsettled |
|---|---|
| **Power normalization** `[POWER-NORM]` | All of §4.5 is pending ratification: tier definitions, overhead fractions (30% liquid, 50% air), reference components, and the name and units of `system_tps_per_kw`. v1.0 uses Tier 3, the component sum. Tier 2, nameplate power with verified capping, needs its own verification method first. Tier 1 is undefined; measured power is anticipated |
| **Serviced normalization** | Deferred to a later version. Whether RDI stays optional is flagged for confirmation |

### Likely to affect your classification

| Area | What is unsettled |
|---|---|
| **Custom SKUs** `[CUSTOM-SKU]` | How to classify hardware in production at hyperscalers but not orderable by any comparable customer. It fails Available criterion 2 and may have no GA commitment, so cannot be Preview — but it is not a prototype. Options: classify as RDI (current default, with the 221-day cooling-off), or create a new "Production" tier. **Defaults to RDI until decided** |
| **RDI comparability** `[RDI-COMP]` | Whether RDI results should be directly comparable to Available and Preview on the same charts. Currently all three are plotted together |
| **Serviced requirements** `[SERVICED-REQ]` | Which system-description fields are required versus optional for APIs; how to handle API versioning when the underlying model or stack changes without notice; whether Serviced results should carry a permanent reproducibility disclaimer |

### Likely to affect your optimisation choices

| Area | What is unsettled |
|---|---|
| **Checkpoint residency** `[CKPT-RESIDENCY]` | Whether a component present in the canonical checkpoint — an MTP or EAGLE head — must also be **resident in accelerator memory** during measurement. The rules require it in the *artifact*; residency is undecided. Memory freed by not loading it converts directly into KV-cache capacity and therefore throughput, so this is a real and currently undisclosed advantage. Options: require residency, require disclosure of the loaded component set, or leave unconstrained |
| **Speculative decoding** | §2.9.4's approved-list model is new for v1.0 and marked tentative. No list is published yet — see **C10** |
| **Token counting** `[TOK-COUNT]` | Whether stakeholders accept that published numbers may differ from serving-stack-reported numbers, given the reference-chat-template tokenization rule. Resolution needed before v1.0 publishes side-by-side charts |
| **Iteration coalescing** | Whether the server returning multiple generated tokens in a single network message is allowed. **Until resolved: disclose any token-coalescing behaviour and conservatively assume `stream_all_chunks = true` semantics** |
| **Standardized CoN techniques** | A comprehensive list of allowed techniques and optimizations for the Standardized CoN scenario is still under development |
| **Partial Unicode at chunk boundaries** | An open edge case in the tokenizer rules |

### Likely to affect review

| Area | What is unsettled |
|---|---|
| **Embargo versus cohort cadence** | Embargoed results publish on the embargo date, not a cohort date. Under provisional publication that can make a result public *earlier* than its next eligible cohort. The working group is deciding whether that's intended |
| **Late objections** | The entire late-objection policy is `[WIP — pending WG approval]`. The grounds, process and time limits are a current proposal |
| **Dispute resolution** | Panel composition, deadlines, non-participation consequences and the 8-week backstop require ratification. Still unspecified: quorum and voting among neutral members, confidentiality provisions, and whether a standing roster of pre-cleared neutrals should be maintained |
| **Audit votes** | The **quorum** is undecided. Cadence (quarterly), the voting rule (ranked choice, simple majority) and the chairs' power to add new-accelerator systems are settled |
| **Random audit selection** | The reroll-on-repeat proposal for avoiding streaks needs working-group approval. The selection text is adapted from MLPerf Inference and still talks about rounds and a withdrawal deadline, which don't map cleanly onto rolling cohorts |
| **Audit nominations** | §10.1 now fixes one voted audit per quarter, but the note under §10.4 still asks whether nominated audits count against the capacity "or are additional to chair-selected audits", and whether a nominating member bears any audit cost |
| **Review chair membership** | Whether the review chair must be an MLCommons member |

## How to use this page

- **Before planning a submission** — read sections A and C. If anything there blocks you,
  [ask](support.md) before scheduling hardware.
- **Before tuning** — read *Likely to affect your optimisation choices*. Building a submission on a
  technique whose status is open is a risk you should take knowingly.
- **Before claiming Available** — read *Likely to affect your classification*.

!!! tip "When in doubt, disclose"
    Several open items resolve the same way in practice: where a rule is unsettled, disclosing what
    you did turns a potential compliance failure into a reviewable choice.
