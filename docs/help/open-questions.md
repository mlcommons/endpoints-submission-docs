# Open questions and WIP rules

What could still change under you, and what this documentation could not confirm. Read this before
committing accelerator time.

--8<-- "draft-rules-warning.md"

**Last reviewed:** 2026-10-02, against `endpoints_policies@v1.0_rules_dev` (d2d9da6),
`endpoints-submission-cli@main` (a42a056, tag `v1.1.0.0`), `endpoints@main` (f1100cf), and the
working group's v1.0 rules overview of 2026-09-22.

??? info "What changed at the 2026-10-02 review"
    The rules moved 10 commits. What matters to a submitter:

    - **Streaming (§6.5, §2.9.7).** Fixed-concurrency performance runs must use streaming
      responses; a dedicated Offline run is exempt. `stream_all_chunks` is now a client-side
      setting that can take either value. A multi-token stream interval on the server is allowed.
    - **`model_name` moved to `point.yaml`.** It's gone from `system_desc.json`, and so is
      `link_config`. The rules now say `system_desc.json` everywhere.

    Checker `v1.1.0.0` (2026-10-01) knows the agentic models and gates their accuracy, rejects a
    single-turn curve with no Offline point, and follows §4.5.2's power model. That resolves
    **B10**, **B12** and **B15**. The client's agentic example settled the third agentic model as
    DeepSeek-V4.1-Flash and published its targets, which closes **C11**.

    In the CLI, `--publication-cycle` is gone and a new submission waits in `COMPLIANCE_CHECKING`
    instead of moving straight to `REVIEW_PENDING`. On the client side, the steady-state detector
    now runs inside the benchmark when you ask for it, which narrows **B9**.

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
    **B4**. The steady-state detector reached `endpoints@main` as an ad-hoc script, which narrows
    **B8** and **B9**.

## A. Registration and membership

| # | Question | Status |
|---|---|---|
| **A1** | Must a submitter's **organisation** be an [MLCommons member](../understand/eligibility/membership.md), or is the individual's CLA enough? | [Submission Rules §5.1][srules-5.1] requires the CLA from the individual and has you register with an organisation email, but never makes membership a precondition. **Unresolved — confirm with MLCommons.** |
| **A2** | Is there a separate "register for the round" step? | The eight-week advance registration is overridden ([Submission Rules §5.1][srules-5.1]) and there are no fixed rounds ([§4.1][srules-4.1]). Reading: a PRISM account plus a scoped API key is the whole of registration. **Confirm.** |
| **A3** | What are the **[PRISM](../understand/eligibility/prism-api-key.md) and Member Central URLs**? | Neither appears in any rules document, CLI doc, or public MLCommons page. **Needed.** |
| **A4** | Who grants API-creation access, and how long does it take? | [Submission Rules §5.1][srules-5.1] says an email follows account creation. No turnaround is stated. **Needed** for planning. |

## B. Tooling and policy disagreements

Found while cross-reading the rules against the tooling. Each is documented at the relevant page;
none has been resolved by guesswork.

| # | Gap | Detail | This site follows |
|---|---|---|---|
| **B1** | **`system_info` config section** | [Submission Rules §5.2][srules-5.2] tells submitters to run the benchmark "with config.yml having `system_info` section (if you want to automatically capture the system description)". **No `system_info` exists anywhere in the reference client's schema**, and the CLI docs state plainly that `system_desc.json` is submitter-authored and not an endpoints artifact. Either the policy anticipates unshipped work, or the sentence is stale. | The tooling — the reference client doesn't capture it. Write `system_desc.json` yourself or capture it with [`mlperf-sysinfo`](https://docs.mlcommons.org/mlperf-sysinfo/) |
| **B2** | **Pareto updates** | The technical rules still reference a post-submission update window, for example as the reason for the 10% High Concurrency margin, and link "Submission Rules §8.1 Pareto Updates". **That section no longer exists** — §8.1 is now [*Corrections*][srules-8.1]. The CLI confirms the removal: there is no `add-run`, and `update --run-ids` rejects additions. | The Submission Rules and the CLI — points are fixed at creation |
| **B3** | **Section-number drift** | The Submission Rules link "Endpoints Rules §7" for directory structure; it is §8.1. The checker's documentation cites §14/§15/§16 for metrics, accuracy and consistency; those sections do not exist. | Clause numbers as they actually are |
| **B5** | **TTFT percentile** | The public MLCommons benchmark page still advertises **TTFT P95**. The v1.0 rules require **P90** and state P95 was the v0.7 metric. | The rules — P90 |
| **B6** | **Result labels** | The rules use Available / Preview / RDI. The public page additionally shows "Verified / Provisional / Unverified". The relationship is unstated. | The rules — needs a mapping |
| **B8** | **Which metrics gate a steady-state window** | *Narrowed.* [§4.4][rules-4.4]'s definition table and the detector documentation now on `endpoints@main` agree: **TPOT at P50 and P90**, with TTFT a diagnostic and drift warning only. But the paragraph after that table in §4.4 still says **TTFT and TPOT at P50/P90**. | TPOT at P50/P90, but confirm |
| **B9** | **Nothing writes the `steady_state` block** | *Partly resolved.* Since endpoints#514 (2026-09-24) the client finds the steady-state window during the run when you pass `--steady-state`, and reports it in `result_summary.json` and `report.txt`. It's off by default and doesn't run for Offline or agentic runs. The checker reads the block from `point.yaml`, and nothing copies it there. The standalone detector is now `python -m inference_endpoint.metrics.steady_state_diagnostics <run_dir>`. The rules still link a pinned commit on the old design branch. | Run with `--steady-state` and copy the result into `steady_state` |
| **B11** | **Who needs `system_power.json`** | [§4.5][rules-4.5]'s scope makes power normalization mandatory for Standardized, optional for RDI and deferred for Serviced. The same section's callout says *every submission* must include the file, and §9.1's *Power descriptor* row says Standardized. The checker requires it for every system regardless of division. | Include it whatever your division |
| **B13** | **"One accuracy run for each of the 5 pareto regions"** | [§4.3][rules-4.3] now opens with that sentence. There are four regions plus the Offline point, and agentic benchmarks need only four accuracy results. The rest of §4.3 and §5.3 are consistent with each other: `N` = 5 non-agentic, 4 agentic. | §5.3 — five or four points |
| **B14** | **§5.4 Example C boundaries** | For `C_min` = 16, `C_max` = 1,024, [§5.4][rules-5.4]'s worked Example C computes `2^6.652` as 100.4 and gives Medium Concurrency 27–116, High 117–1,024. `2^6.652` is 100.57, so the reference algorithm in §5.5, Appendix B and `submission-checker regions` all give Medium 27–117, High 118–1,024. A point at 117 is Medium, not High. | The algorithm and the checker — 27–117 |

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
| **C1** | The **official v1.0 model list** and canonical model IDs | *Partly answered.* The working group's 2026-09-22 overview names the suite: Llama 3.1 8B, GPT-OSS 120B and DeepSeek-R1, plus three agentic models. It's collected in [Benchmarks and models](../reference/benchmarks.md). The rules still defer to a list in "the reference repository", published ≥ 6 weeks before a round, and that list doesn't exist yet. The names the checker accepts are on the same page |
| **C2** | **Per-benchmark accuracy targets and tolerances** | Marked `[WIP]` upstream. For the legacy benchmarks the checker applies the MLPerf Inference targets, listed in [Benchmarks and models](../reference/benchmarks.md#accuracy-targets), but nothing says those are the Endpoints targets. For the agentic benchmarks the client's agentic README publishes targets for all three models, and checker `v1.1.0.0` enforces them |
| **C3** | **Dataset identities and download paths** for performance and accuracy runs | *Mostly answered* in [Benchmarks and models](../reference/benchmarks.md#datasets), from the overview and the client. The GPT-OSS performance set is on MLCommons storage, but the client's README says the LLM task force is still finalizing it. For a dedicated Offline run, the dataset's size is also your Offline concurrency |
| **C4** | **CoN client locations and scheduling procedure** | Deferred to a separate working-group publication that does not yet exist. CoN submitters cannot plan |
| **C5** | **The full submission-state list** | Only `REVIEW_PENDING`, `WITHDRAWN`, `FINALIZED` and `PUBLISHED` are documented |
| **C6** | **Preview Availability Tracker URL** and the public results/visualizer URL | Referenced by the rules; no URL given |
| **C7** | **The Offline point's loose ends** | [§5.7][rules-5.7] defines the Offline point, but several things you need to run one aren't settled. **Load pattern:** the rules say "the benchmark-defined Offline load pattern" and name no client setting; the reference client's `max_throughput` matches the definition, unconfirmed. **`region` value:** unspecified for a dedicated run; `submitters_choice` passes the checker. **Dataset smaller than `C_max`:** the Offline concurrency is the dataset size and must be ≥ `C_max`, so a large `C_max` can't satisfy both. Open upstream as [`[OFFLINE]` item 2][rules-offline]. **Steady state:** §4.4's scope excludes a dedicated Offline run, which implies `total` metrics; not stated outright. **Duration:** §6.2 applies unchanged, though the run ends when the queue drains. **Pass boundaries:** the event log doesn't mark them, so the no-mixing rule is a manual review item |
| **C8** | **Which benchmarks are agentic, and which are multi-turn** | This now decides a lot: whether you need an Offline point, how many points and accuracy runs you owe, the accuracy gate (every point vs mean-of-N), the primary chart (`tps_per_user` vs `e2e_avg_interactivity`), and whether agentic salting flags apply. *Partly answered:* the 2026-09-22 overview splits the suite into legacy and agentic benchmarks, so you can tell which are agentic, and the checker decides it from the load pattern: a curve run with `agentic_inference` is agentic. Which benchmarks count as multi-turn for the accuracy rule still isn't stated. Ties to **C1** |
| **C9** | **Super-pass size per benchmark** | The steady-state window is measured in super-passes, one full dataset pass unless the benchmark definition says otherwise ([§4.4][rules-4.4]). No benchmark definition is published, so you cannot compute your own floor |
| **C10** | **Approved drafter lists** | [§2.9.4][rules-2.9.4] allows speculative decoding only with a drafter on the benchmark's published list. None has been published, the checker ships an empty one, and a drafter approved now can only be used two cohorts later. Until a list appears, speculative decoding isn't available for any benchmark |
| **C12** | **How long a registered run lasts** | `runs create` says a run's expiry "defaults to server policy", and the policy isn't published. It's also not stated whether a run can expire once it's part of a submission. Pin every run you intend to submit |

## D. Rules the working group has not ratified

Marked upstream as `[TENTATIVE]`, `[WIP]`, `[WG Open Item]` or `[WG Decision Required]`. These are
current policy where stated, but are the most likely to move.

### Likely to affect your run plan

| Area | What is unsettled |
|---|---|
| **Run requirements** | All of [§6][rules-6] is pending ratification: minimum durations, query counts, dataset-subset rules and the warmup model. See [`[RUN-REQ]`][rules-run-req] |
| **Offline point** | [§5.7][rules-5.7] is new for v1.0 and marked pending ratification. Agentic Offline is deferred to a later version. See **C7** for the details that aren't settled |
| **Accuracy targets** | Per-benchmark tolerance values are `[WIP]` ([§2.9.8][rules-2.9.8]) |
| **Steady-state reporting** | [§4.4][rules-4.4] carries its own pending-ratification note: whether the 4 super-pass floor rises, and whether a run where no steady state is found is declared **invalid** rather than merely reported-with-flags. The second matters most — it decides whether a run that fails detection is a failed run or just a weaker number. See also **B8** and **B9** |
| **Reproducibility margins** | The 10% and 5% throughput margins are proposals requiring ratification ([Submission Rules §6.6][srules-reproducibility-expectations]) |
| **Latency comparison method** | **None exists.** Until one is ratified, latency alone can't carry a reproducibility objection or block a Preview-to-Available transition. See [Submission Rules §6.6][srules-reproducibility-expectations] and [§7.3.3][srules-7.3.3] |

### Likely to affect your power figure

| Area | What is unsettled |
|---|---|
| **Power normalization** `[POWER-NORM]` | All of [§4.5][rules-4.5] is pending ratification, including tier definitions, overhead fractions and the name of `system_tps_per_kw`. v1.0 uses Tier 3, the component sum. See [`[POWER-NORM]`][rules-power-norm] |
| **Serviced normalization** | Deferred to a later version, and whether RDI stays optional is to be confirmed ([`[POWER-NORM]`][rules-power-norm]) |

### Likely to affect your classification

| Area | What is unsettled |
|---|---|
| **Custom SKUs** `[CUSTOM-SKU]` | How to classify hardware in production at hyperscalers but not orderable by any comparable customer. **Defaults to RDI until decided.** Options are in [Submission Rules §7.5][srules-open-question-hardware-not-orderable-by-any-comparable-customer] |
| **RDI comparability** `[RDI-COMP]` | Whether RDI results should share charts with Available and Preview, as they do now ([`[RDI-COMP]`][rules-rdi-comp]) |
| **Serviced requirements** `[SERVICED-REQ]` | Required versus optional fields for APIs, API versioning, and a possible permanent reproducibility disclaimer ([`[SERVICED-REQ]`][rules-serviced-req]) |

### Likely to affect your optimisation choices

| Area | What is unsettled |
|---|---|
| **Checkpoint residency** `[CKPT-RESIDENCY]` | Whether a component the canonical checkpoint ships, such as an MTP or EAGLE head, must be **resident in accelerator memory** during measurement or only present in the artifact. Leaving it unloaded frees memory for KV cache, an advantage that's currently undisclosed. See [`[CKPT-RESIDENCY]`][rules-ckpt-residency] |
| **Speculative decoding** | [§2.9.4][rules-2.9.4]'s approved-list model is new for v1.0 and marked tentative. No list is published yet — see **C10** |
| **Token counting** `[TOK-COUNT]` | Whether stakeholders accept that published token counts may differ from what serving stacks report ([`[TOK-COUNT]`][rules-tok-count]) |
| **Multi-token stream interval** | Now allowed: the server may group up to N generated tokens into one chunk, as `stream_interval` does in TensorRT-LLM, SGLang and vLLM ([§2.9.7][rules-2.9.7], [§2.9.9 Q2][rules-2.9.9]). Holding tokens past the interval, or sending the whole response as one message, isn't. A larger interval delays TTFT and no metric adjustment is made. Appendix A still lists it as proposed |
| **Standardized CoN techniques** | The list of allowed techniques for Standardized CoN is still being written ([§2.2.2][rules-2.2.2]) |
| **Partial Unicode at chunk boundaries** | An open edge case in the tokenizer rules ([§2.8][rules-2.8]) |

### Likely to affect review

| Area | What is unsettled |
|---|---|
| **Embargo versus cohort cadence** | Embargoed results publish on the embargo date, not a cohort date, which under provisional publication can make a result public *earlier* than its next cohort. The working group is deciding whether that's intended ([Submission Rules §4.5][srules-4.5]) |
| **Late objections** | The whole late-objection policy is `[WIP — pending WG approval]` ([Submission Rules §6.6][srules-6.6]) |
| **Dispute resolution** | The panel, deadlines and 8-week backstop need ratification, and quorum, confidentiality and a standing roster of neutrals are unspecified ([Submission Rules §9.5][srules-9.5]) |
| **Audit votes** | The **quorum** is undecided; the rest of the vote is settled ([Submission Rules §10.2][srules-10.2]) |
| **Random audit selection** | The reroll-on-repeat proposal needs working-group approval ([Submission Rules §10.3][srules-10.3]). The selection text is adapted from MLPerf Inference and still talks about rounds and a withdrawal deadline, which don't map cleanly onto rolling cohorts |
| **Audit nominations** | §10.1 now fixes one voted audit per quarter, but the note under [§10.4][srules-10.4] still asks whether nominated audits count against the capacity "or are additional to chair-selected audits", and whether a nominating member bears any audit cost |
| **Review chair membership** | Whether the review chair must be an MLCommons member ([Submission Rules §2.2][srules-2.2]) |

## How to use this page

- **Before planning a submission** — read sections A and C. If anything there blocks you,
  [ask](support.md) before scheduling hardware.
- **Before tuning** — read *Likely to affect your optimisation choices*. Building a submission on a
  technique whose status is open is a risk you should take knowingly.
- **Before claiming Available** — read *Likely to affect your classification*.

!!! tip "When in doubt, disclose"
    Several open items resolve the same way in practice: where a rule is unsettled, disclosing what
    you did turns a potential compliance failure into a reviewable choice.
