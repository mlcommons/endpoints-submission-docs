# Glossary

Terms used across this site. Most also appear as hover tooltips on their first use on any page.

## Benchmark concepts

**Pareto curve**
:   The published result: a step function of `system_tps` against `tps_per_user` across concurrency
    levels. One curve = one system, one benchmark model, one dataset.

**Measurement point**
:   One benchmark run at one concurrency level. A submission carries 7–32 of them, 8 when one is a
    dedicated Offline run.

**Offline point**
:   The point that makes the whole performance dataset available at once and measures only
    throughput. Required for non-agentic benchmarks, not allowed for agentic ones. Either a
    **dedicated** run, whose concurrency is the dataset size, or the `C_max` point **elected** as
    the Offline result.

**Step function**
:   The official curve representation. Each point is a discrete step; between points the curve holds
    at the last measured value. No interpolation.

**Region of interest (ROI)**
:   One of four concurrency bands — Ultra Low, Low, Medium, High. Named for latency or concurrency,
    but all four are bounded by concurrency.

**`C_min` / `C_max`**
:   Minimum and Maximum Supported Concurrency. `C_max` is declared; `C_min` is derived from your
    lowest point in v1.0. Together they set your region boundaries.

**SUT**
:   System Under Test.

## Metrics

**`system_tps`**
:   Total output tokens per second across all concurrent users.

**`tps_per_user`**
:   Per-user output rate: `1000 / tpot_p90_ms`.

**`system_tps_per_kw`**
:   `system_tps` divided by the system's provisioned power in kW. The v1.0 normalized metric.

**Provisioned power**
:   What a system is built to draw, estimated from the rated power of its CPUs, accelerators and
    scale-up switches plus a cooling-dependent overhead. Fixed for a system across its whole curve.
    Declared in `system_power.json`.

**MLC Estimated Power**
:   The tag on a result whose power figures were partly or wholly filled in by MLCommons.

**TTFT**
:   Time To First Token. Measured to the first **non-empty** text fragment in any response category.
    v1.0 reports P90.

**TPOT**
:   Time Per Output Token. Measured on the suffix after the first output-bearing streamed chunk.

**ISL / OSL**
:   Input / Output Sequence Length, in reference-tokenizer tokens.

**Accuracy gate**
:   The benchmark's minimum quality target. A hard gate with no variability allowance.

## Divisions and scenarios

**Standardized**
:   The primary division. Strict model equivalence, full disclosure. Plays the role of *Closed*.

**Serviced**
:   For publicly available commercial inference-as-a-service endpoints. CoN only.

**RDI**
:   Research, Development and Internal. Experimental or pre-release systems. Plays the role of *Open*.

**CoP / CoN**
:   Client on Prem — you operate the client. Client over Network — MLCommons operates it and reaches
    your public endpoint.

## Publication

**Available / Preview / RDI**
:   Publication status — whether the hardware and software can be bought today. Orthogonal to
    division.

**Cohort**
:   A publication batch, `YYYY-MM-C0` (1st Wednesday) or `YYYY-MM-C1` (3rd Wednesday), 08:00 Pacific.

**Publication mode**
:   Confidential review, confidential review with an embargo, or provisional publication. Chosen
    at submission; irrevocable.

**Provisional publication**
:   Opting to publish before peer review completes, tagged *peer review pending*. Review starts at
    provisional publication.

**Embargo**
:   A requested hold on when results become public. Under provisional publication it also delays
    the start of review.

**Finalization**
:   The point at which all objections are resolved and the *peer review pending* tag is removed.
    Most post-publication clocks run from here.

**Result ID**
:   `<major>.<minor>.<cohort>.<model_id>.<dataset_id>.<entry>`. Assigned at publication, never reused.

## Process

**PRISM**
:   The MLCommons portal that issues the API tokens the submission CLI authenticates with. See
    [PRISM API key](../understand/eligibility/prism-api-key.md).

**CLA**
:   Contributor License Agreement. Required of the **individual** making the submission. See
    [MLCommons CLA process](../understand/eligibility/cla-process.md).

**Objection**
:   A formal challenge filed as a GitHub issue during peer review, tagged `by <org>` and
    `against <org>`.

**Late objection**
:   An objection after Week 6, permitted only on availability, validity, model-equivalence and
    division-rule grounds. Reproducibility is not eligible.

**Audit nomination**
:   Proposing a published result for the quarterly audit vote, open to committee members for 4
    weeks after publication. Also the route for a post-finalization reproducibility concern.

**Neutral member**
:   A person with minimal conflict of interest, used to staff dispute panels and screening. Ordinary
    competition and CSP/OEM/ODM partnerships do not disqualify.

**Settled**
:   A result no longer open to late objection or audit nomination — the later of the next audit vote
    or 90 days after finalization.

## Technical

**Seed set**
:   The seeds MLCommons publishes that control the client's run-to-run non-determinism. A submission
    binds to exactly one.

**Salt**
:   A unique value added between the system prompt and the user context on every query. It's what
    makes cross-query KV-cache reuse allowable.

**Reference tokenizer**
:   The tokenizer published with the benchmarked model in its canonical Hugging Face repository.
    Produces the official token count.

**Reference chat template**
:   The canonical message-formatting spec for the model. Submissions must use it.

**PTQ**
:   Post-Training Quantization. The canonical permitted weight transformation.

**Drafter**
:   The speculative-decoding module (MTP, EAGLE-style head, or a self-speculative pass). Chosen
    from the benchmark's approved list, used unmodified apart from PTQ, and the same across the
    curve.

**Approved drafter list**
:   The per-benchmark, per-round list of drafters a Standardized submission may use. None has been
    published yet.

**ConcurrencyScheduler**
:   The reference client's fixed-concurrency load pattern — it keeps a set number of queries in
    flight, issuing a replacement as each one completes. The rules now say "the benchmark-defined
    fixed-concurrency load pattern" rather than naming it. `Poisson` is invalid for any Pareto
    point; `MaxThroughput` only for a dedicated Offline run.

**Warmup**
:   Requests issued before `TEST_STARTED`. Excluded from metrics, but logged, retained and declared.

**Steady-state window**
:   The stable stretch of a run, after warmup and the residual ramp-up and before the drain, over
    which the official metrics are computed. Found by a post-processing step over `events.jsonl`.
    See [Metrics and regions](metrics-and-regions.md#what-your-numbers-are-measured-over).

**Super-pass**
:   A contiguous block of queries in issue order, sized to one full pass over the dataset. The unit
    the steady-state window is measured in. A window needs at least 4.

**Plateau / Drifting Up / Drifting Down**
:   The three states a gating metric can be in across the super-passes. A window counts as steady
    only when every gating metric is a Plateau.

**Ramp-up / drain**
:   The transients at the start and end of a run — concurrency climbing to target, and the last
    queries finishing with nothing new issued. Ramp-up inflates the TTFT tail; drain deflates
    throughput. Both are excluded from the steady-state window.

**`total` metrics**
:   Whole-run metrics, averaged over everything after `TEST_STARTED`. The pre-v1.0 basis. Now
    supplementary — except where no steady state holds, when they become the official result.

**E2E average interactivity**
:   `e2e_avg_interactivity`. The agentic-benchmark counterpart to `tps_per_user`: output tokens
    summed across all completed turns, divided by the summed server-side turn time, excluding
    tool-call execution.

**TP / EP / PP / DP**
:   Tensor / Expert / Pipeline / Data parallelism.

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
