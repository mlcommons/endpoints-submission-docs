<!--
  PROVENANCE SNAPSHOT — do not edit.
  Upstream : endpoints_policies/endpoints_rules.md
  Repo     : mlcommons/endpoints_policies @ main@4ba8224
  Captured : 2026-10-30
  Purpose  : diff this against upstream to find what drifted since the docs were written.
-->

# MLPerf® Endpoints Rules

*MLPerf Endpoints Rules Task Force — Version 1.0 Draft — 2026-05-05*

**Companion documents:**
- Submission, review, and publication process: [endpoints_submission_rules.md](endpoints_submission_rules.md)
- General MLPerf submission rules (inherited where not overridden): [submission_rules.adoc](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc)

---

## Table of Contents

1. [Basics](#1-basics)
2. [Divisions and Deployment Scenarios](#2-divisions-and-deployment-scenarios)
   - [2.1 Client Deployment Scenarios](#21-client-deployment-scenarios)
   - [2.2 Standardized Division](#22-standardized-division)
   - [2.3 Serviced Division](#23-serviced-division)
   - [2.4 RDI Division](#24-rdi-research-development-and-internal-division)
   - [2.5 Division Summary](#25-division-summary)
   - [2.6 Reproducibility Requirements](#26-reproducibility-requirements)
   - [2.7 Transparency Requirements](#27-transparency-requirements)
   - [2.8 Tokenizer Rules](#28-tokenizer-rules)
   - [2.9 Model Equivalence Rules (Standardized Division)](#29-model-equivalence-rules-standardized-division)
3. [Benchmarks and Models](#3-benchmarks-and-models)
   - [3.1 Benchmark Definition](#31-benchmark-definition)
   - [3.2 Supported Models](#32-supported-models)
   - [3.3 Weight Transformations](#33-weight-transformations)
4. [Metrics](#4-metrics)
   - [4.1 Primary Metrics](#41-primary-metrics)
   - [4.2 Derived and Presentation Metrics](#42-derived-and-presentation-metrics)
   - [4.3 Accuracy Metric](#43-accuracy-metric)
   - [4.4 Reporting Basis (Steady-State Window)](#44-reporting-basis-steady-state-window)
   - [4.5 Performance Normalization](#45-performance-normalization)
     - [4.5.1 Power Normalization Roadmap and Rationale](#451-power-normalization-roadmap-and-rationale)
     - [4.5.2 Proposed Endpoints v1.0 Normalization Methodology](#452-proposed-endpoints-v10-normalization-methodology)
     - [4.5.3 Normalized Metric](#453-normalized-metric)
5. [Pareto Collection Methodology](#5-pareto-collection-methodology)
   - [5.1 What Is Measured](#51-what-is-measured)
   - [5.2 Pareto Curve Representation](#52-pareto-curve-representation)
   - [5.3 Minimum Submission Requirements](#53-minimum-submission-requirements)
   - [5.4 Regions of Interest](#54-regions-of-interest)
   - [5.5 Region Boundary Reference Algorithm](#55-region-boundary-reference-algorithm)
   - [5.6 Maximum Point Cap](#56-maximum-point-cap)
   - [5.7 Offline Point](#57-offline-point)
6. [Run Requirements Per Measurement Point](#6-run-requirements-per-measurement-point)
   - [6.1 Load Pattern](#61-load-pattern)
   - [6.2 Minimum Run Duration](#62-minimum-run-duration)
   - [6.3 Warmup Period](#63-warmup-period)
   - [6.4 Minimum Completed Queries](#64-minimum-completed-queries)
   - [6.5 Dataset Considerations](#65-dataset-considerations)
   - [6.6 Accuracy Requirement](#66-accuracy-requirement)
7. [Publication Status](#7-publication-status)
8. [Submission Requirements](#8-submission-requirements)
   - [8.1 Directory Structure](#81-directory-structure)
   - [8.2 System Description (system\_desc.json)](#82-system-description-system_descjson)
   - [8.3 Measurement Point YAML](#83-measurement-point-yaml)
   - [8.4 Software Disclosure](#84-software-disclosure)
   - [8.5 Result ID](#85-result-id)
9. [Compliance Validation](#9-compliance-validation)
   - [9.1 Automated Checks](#91-automated-checks)
   - [9.2 Manual Review Focus Areas](#92-manual-review-focus-areas)
- [Appendix A: Open Questions and Working Group Items](#appendix-a-open-questions-and-working-group-items)
- [Appendix B: Quick-Reference Region Boundary Table](#appendix-b-quick-reference-region-boundary-table)
- [Appendix C: Worked Power Normalization Examples](#appendix-c-worked-power-normalization-examples)
- [Appendix D: MLCommons Default Power Reference Values](#appendix-d-mlcommons-default-power-reference-values)
- [Appendix E: `system_power.json` Field Reference](#appendix-e-system_powerjson-field-reference)

---

## 1. Basics

These rules define the technical requirements for MLPerf Endpoints benchmark submissions: what to
measure, how to measure it, which division to submit under, and what evidence is required for each
publication status category.

The submission, review, and publication *process* are defined separately in the companion
[MLPerf Endpoints Submission Rules](endpoints_submission_rules.md) document.

> [!NOTE]
> **Rule Stability.** These rules are *tentative* until the **v1.0** submission round opens on
> **2026-10-12**, after which rolling submission begins. Sections explicitly marked
> **`[TENTATIVE — Subject to change after 2026-10-12]`** are the most likely to evolve, based on
> submitter feedback from the v0.7 round (which closed on 2026-06-26) and on working-group
> discussion. See [Submission Rules §4.0](endpoints_submission_rules.md#40-submission-milestones)
> for the full milestone table.

MLPerf Endpoints measures the performance of *inference endpoints* serving generative AI models.
Unlike traditional MLPerf Inference benchmarks — which measure latency or throughput at a single
operating point — MLPerf Endpoints characterizes the full performance *envelope* of a serving system
as a Pareto curve across a range of concurrency levels.

The benchmark is designed to be:

- **Fair** — standardized measurement methodology with compliance validation.
- **Inclusive** — minimum 7 measurement points with flexible placement to accommodate systems of all
  scales.
- **Relevant** — covering operating points valued by real users: single-user interactive latency
  through high-throughput batch serving.
- **Extensible** — modular rules that can accommodate new models, metrics, and divisions without redesign.

---

## 2. Divisions and Deployment Scenarios

MLPerf Endpoints replaces the traditional Closed/Open division structure from MLPerf Inference with
three divisions tailored to endpoint benchmarking: **Standardized**, **Serviced**, and **RDI**. All
divisions use the same pareto collection methodology ([§5](#5-pareto-collection-methodology)) and
run requirements ([§6](#6-run-requirements-per-measurement-point)). A submission must declare
exactly one division.

For readers familiar with MLPerf Inference's Closed/Open structure: **Standardized** plays the role
of *Closed* (strict equivalence, full disclosure); **Serviced** is a new division for commercial
endpoint services with no direct MLPerf Inference analog; **RDI** (Research / Development /
Internal) plays the role of *Open*. The Available/Preview/RDI **publication status** dimension is
*orthogonal* to the division; see [Submission Rules §7](endpoints_submission_rules.md#7-publication)
for which (division, status) combinations are valid.

### 2.1 Client Deployment Scenarios

MLPerf Endpoints defines two scenarios that determine how the client infrastructure connects to the
System Under Test (SUT).

#### 2.1.1 Client on Prem (CoP)

The submitter hosts both the client infrastructure and the endpoint server infrastructure. The
client and server may be co-located in the same data center or connected via a local network.

- The submitter provides and operates both client and server infrastructure.
- The client must use the MLPerf Endpoints reference client (`inference_endpoint` from
  `github.com/mlcommons/endpoints`) without source-code modification, compiled from a commit
  accessible to the MLCommons review committee. Submitters MAY configure runtime behavior via the
  YAML configuration file the client accepts; everything that changes behavior MUST be expressible
  via that YAML. The client logs the commit SHA used for the run; review may additionally use a
  seeded RNG check (analogous to LoadGen's RNG-output check in MLPerf Inference) to detect
  undisclosed client modifications.
  The seeded RNG check uses the submission's bound seed set, selected from the sets MLCommons make
  available every publication date
  ([Submission Rules §4.6](endpoints_submission_rules.md#46-seed-rotation)); the client's
  request-issue / sample-order RNG and the per-query salt MUST each be seeded from that set, and the
  seed set MUST be set through the YAML configuration.
- Network latency between client and server is included in all timing measurements.
- The submitter must document the network topology between client and server, including type of
  interconnect, number of hops, and measured baseline network latency.
- On-prem submissions must be self-contained: all components required to replicate the result must
  be documented and provided.

#### 2.1.2 Client over Network (CoN)

MLCommons is responsible for the client infrastructure, which interrogates the System Under Test via
an endpoint accessed over the public Internet.

- MLCommons operates the client infrastructure at a designated location.
- The submitter provides a publicly accessible endpoint URL that the MLCommons client can reach.
- All timing measurements include public Internet network latency between the MLCommons client and
  the submitter's endpoint.
- CoN submitters must provide equivalent containers and code to replicate the server in alternative
  locations, clusters, or data centers for identical hardware and software configurations.
- The endpoint must conform to the MLPerf Endpoints reference API specification.

> [!NOTE]
> The specific MLCommons client locations, network requirements, and scheduling procedures for CoN
> submissions will be published separately by the working group.

#### 2.1.3 Scenario Summary

| Property | Client on Prem (CoP) | Client over Network (CoN) |
|---|---|---|
| Client operator | Submitter | MLCommons |
| Server operator | Submitter | Submitter |
| Network | Local / data center | Public Internet |
| Network latency included | Yes | Yes |
| Reference client required | Yes | Yes (MLCommons-hosted) |

---

### 2.2 Standardized Division

The Standardized division is the primary benchmark division, requiring strict adherence to model
equivalence rules and full software disclosure. It replaces the traditional "Closed" division from
MLPerf Inference.

**Transparency:** Whitebox — model weights, configurations, optimization details, and launch /
integration scripts must be disclosed. The serving framework and low-level software stack must
satisfy the **Available** definition in the Submission Rules
([Submission Rules §7.2](endpoints_submission_rules.md#72-available)); they need not be open-sourced
verbatim if the Available criteria are met.

**Available Scenarios:** Client on Prem (CoP) and Client over Network (CoN), reported as separate sub-divisions.

#### 2.2.1 General Rules

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`** This section ports the MLPerf Inference
> optimization framing to a strictly disallowed-list ("blacklist") style. The exact disallowed
> entries below may be revised after v0.7 submitter feedback.

**Inheritance.** Standardized division submissions inherit the model-equivalence and optimization
rules of
[MLPerf Inference §Model Equivalence](https://github.com/mlcommons/inference_policies/blob/master/inference_rules.adoc#model-equivalence).
**This document is the source of truth and overrides upstream wherever the two conflict.** Where
upstream uses a non-exhaustive list of allowed examples followed by a disallowed list, Endpoints
uses a single **disallowed-only** formulation: anything not listed below and not in conflict with
the [§2.9 Model Equivalence Rules](#29-model-equivalence-rules-standardized-division) is permitted.
See [§2.9.6 Sparsity and Approximate Computation](#296-sparsity-and-approximate-computation) for the
treatment of sparse and approximate execution, and
[§2.9.9 Q&A](#299-qa-model-equivalence-clarifications) for clarifying examples.

**Operative requirements:**

- Pre-processing, post-processing, and the model executed by the SUT must be equivalent to the
  reference implementation, per [§2.9](#29-model-equivalence-rules-standardized-division).
- Submissions must be reproducible: configuration, server launch scripts, and client integration
  scripts must be submitted. For numerical recipes such as calibration, the submission must either
  (a) describe the recipe in sufficient detail for an external team to reproduce it, or (b) provide
  the scripts / software that implement it. The underlying serving framework and low-level software
  stack must satisfy the **Available** definition in the Submission Rules
  ([Submission Rules §7.2](endpoints_submission_rules.md#72-available)).
- On-prem (CoP) submissions must be self-contained.

**Disallowed optimizations** (Standardized division):

- Wholesale weight replacement or supplements.
- Discarding non-zero weight elements (pruning), except where the operation is *mathematically
  equivalent* to the dense reference (see
  [§2.9.6.2](#2962-exact-sparse-execution-and-softmax-elision)).
- Knowledge distillation to a different architecture.
- Retraining, fine-tuning, LoRA, adapter layers, RLHF, or any gradient-based weight update — applied
  to the canonical model or to any draft model used in speculative decoding (see
  [§2.9.4](#294-speculative-decoding)).
- **Speculative decoding** using a drafter or algorithm other than one approved for the benchmark —
  governed by the approval process and eligibility requirements in
  [§2.9.4](#294-speculative-decoding). (Per-point *configuration* of an approved drafter may vary;
  see §2.9.4.)
- Response caching: returning a cached response *verbatim* to a request that matches a previous
  request, bypassing the forward pass. Every request must execute the forward pass. (Note: this is
  distinct from cross-query KV-cache reuse, which still executes the forward pass on a per-query,
  salt-uniquified token stream — see [§2.9.5 KV Cache Rules](#295-kv-cache-rules) for the operative
  rule.)
- Coalescing identical queries (deduplicating duplicate queries in flight to amortize work across them).
- Modifying weights during the timed portion of an inference run (online learning).
- Benchmark detection: the framework or system must not detect a benchmark workload and behave differently.
- Input-based optimization: the submission's *implementation* (the framework, model artifacts,
  kernels, calibration outputs, and build-time configuration) must not encode any information about
  the content of the benchmark input dataset. This rule targets static / build-time dataset
  awareness (e.g., baking dataset statistics into kernel selection, embedding tables, or
  compile-time constants). It does **not** prohibit *runtime* serving-stack behavior that derives
  cache keys from the live token stream — KV-cache reuse based on shared prefixes (governed by
  [§2.9.5 KV Cache Rules](#295-kv-cache-rules)) is permitted, since it operates on the request data
  rather than on prior knowledge of the dataset.
- Client-side dispatch manipulation: the reference client must not be modified to delay, batch, or
  reorder the dispatching of queries in order to manipulate TTFT, TPS/User, or other measured
  metrics. Server-side scheduling of received requests is governed by the normal serving rules and
  is not constrained by this bullet.
- Modification of the request/response stream outside the reference API specification.
- Weight-quantization algorithms whose specification is similar in size to the non-zero weights they
  produce (inherited from upstream — defeats principled-quantization intent).
- Hard-coding the total number of queries; techniques that boost performance for fixed-length
  experiments but are inapplicable to long-running services.

> [!NOTE]
> **Why blacklist-only?** Submitters frequently ask "is X allowed?" for techniques that don't exist
> yet (new quantization formats, novel kernels, alternative attention impls). A closed whitelist
> forces a rule change every time. Endpoints maintains a single disallowed list together with the
> Model Equivalence rules ([§2.9](#29-model-equivalence-rules-standardized-division)); anything not
> banned and consistent with model equivalence is permitted. The
> [§2.9.9 Q&A](#299-qa-model-equivalence-clarifications) provides interpretive guidance.

**Divergences from upstream.** Inheritance is not wholesale. Endpoints both **narrows** and
**widens** the upstream allowances, and where the two disagree this document governs. A submitter
who reads only the upstream allowed list will get the wrong answer on the following, each of which
is a deliberate Endpoints decision rather than an oversight:

| Upstream §Model Equivalence | Endpoints position | Where |
|---|---|---|
| Allows *empirical performance and accuracy tuning based on the performance and accuracy set* (e.g. selecting batch sizes or numerics experimentally). | **Narrowed.** Tuning against the benchmark performance or accuracy dataset is input-based optimization and is disallowed. Thresholds for approximate methods must be selected on the published calibration set or by a data-independent procedure. | [§2.9.6.4](#2964-dynamic-approximate-sparsity), and the input-based-optimization bullet above |
| Allows *incorporating explicit statistical information about the calibration set* (min, max, mean, distribution). | **Retained**, and unchanged: the *calibration* set remains a legitimate source of statistics. The boundary Endpoints enforces is calibration set versus benchmark set, not statistics versus no statistics. | [§2.9.3](#293-model-weight-rules), [§2.9.6.4](#2964-dynamic-approximate-sparsity) |
| Allows *dead code elimination*, unqualified. | **Narrowed.** Permitted for the executed graph, and only where the elimination is data- and weight-invariant. Does not license removing components from the submitted checkpoint artifact. Dead code in the reference that is data-dependent must be reported to the working group. | [§2.9.6.2](#2962-exact-sparse-execution-and-softmax-elision), [§2.9.3](#293-model-weight-rules) |
| Prohibits cross-query KV-cache reuse. | **Widened.** Permitted under the per-query salt mechanism. | [§2.9.5](#295-kv-cache-rules) |
| Allows speculative decoding for an enumerated set of upstream workloads. | **Replaced.** Governed by the Endpoints approved-drafter list and its own eligibility and approval process. | [§2.9.4](#294-speculative-decoding) |
| Permits sparse execution only where mathematically equivalent to the dense reference. | **Widened.** Dynamic, run-time-derived approximate sparsity is permitted under the accuracy gate. | [§2.9.6.4](#2964-dynamic-approximate-sparsity) |

*Why the tuning rule is narrowed:* upstream is in tension with itself on this point — its allowed
list permits *empirical performance and accuracy tuning based on the performance and accuracy set*,
while its disallowed list prohibits *incorporating explicit statistical information about the
performance or accuracy sets*. Endpoints resolves that tension toward the prohibition. Upstream's
allowance is workable where a benchmark's accuracy is a pass/fail threshold on a fixed task.
Endpoints reports a curve across concurrency, and [§2.9.6.4](#2964-dynamic-approximate-sparsity)
admits approximations whose aggressiveness is set by a tunable threshold. Permitting that threshold
to be tuned against the benchmark's own accuracy set would let a submitter walk the approximation
right up to the pass line on the data it is scored against, which is a materially different rule
from choosing a batch size experimentally. The accuracy gate only means something if the threshold
was not fitted to it.

#### 2.2.2 Client over Network (CoN) — Additional Rules

When submitting to the Standardized division via the CoN scenario, the following additional rules apply:

- CoN submitters may choose to submit to CoP instead, but must follow all CoN compliance rules when
  doing so.
- Servers must not modify incoming or outgoing request/response streams outside the provided MLPerf
  Endpoints reference API specification.
- The reference client performs all request pre-processing (e.g., tokenization, packing, precision
  conversion) and all response post-processing (e.g., detokenization, ArgMax, reduction). The SUT
  executes the model and the reference's serving path only; it does not transform request/response
  payloads beyond what the reference API specifies.
- Server must not deliberately delay token dispatch to manipulate TTFT or TPS/User metrics.
- Server must not cache responses or requests across queries.

> [!NOTE]
> **[WIP]** — A comprehensive list of allowed techniques and optimizations for the Standardized CoN
> scenario is under development by the working group.

#### 2.2.3 Result Naming

Unqualified use of "MLPerf Endpoints" refers to results from the Standardized division. Example:
*"MLPerf Endpoints result of 5,000 tokens/s at concurrency 64."*

---

### 2.3 Serviced Division

The Serviced division benchmarks publicly available, generally accessible inference-as-a-service
endpoints. This is a new division unique to MLPerf Endpoints, designed to benchmark commercial Gen
AI API offerings.

**Transparency:** Greybox — the endpoint behavior must be reproducible and auditable, but full
internal implementation details need not be disclosed. The API interface, model identity, and
pricing must be public.

**Available Scenarios:** Client over Network (CoN) only.

#### 2.3.1 Rules

- The endpoint must be a publicly available, generally accessible commercial service. "Generally
  accessible" means any customer meeting standard terms of service can obtain access.
- Performance must be reproducible: the endpoint must deliver consistent results when benchmarked at
  different times within a reasonable window.
- Audit and accuracy tests are required to verify the endpoint produces correct outputs.
- The submitter must disclose: the model name and version as advertised by the service, the API
  endpoint URL, the pricing model and rates at time of submission, and any rate limits or quotas
  that apply.
- Serviced submissions may augment the base reference model by pruning, sparsification, quantizing,
  fine-tuning, modification of speculative decoding heads, and alternative attention mechanisms. Any
  such augmentations must be disclosed.
- Response caching across queries is not allowed.

**Optimization transparency:**

| Category | Requirement |
|---|---|
| Precision | Required |
| Speculative decode, fusion, changes | Disclosure required (no source code required) |
| Model quantization | Optional |

#### 2.3.2 Result Naming

Results must use the qualified name "MLPerf Endpoints Serviced." Example: *"MLPerf Endpoints
Serviced result of 3,200 tokens/s at concurrency 32."*

---

### 2.4 RDI (Research, Development, and Internal) Division

The RDI division provides a category for experimental, pre-release, or internal systems that do not
meet Standardized or Serviced requirements. It replaces the traditional "Open" division.

**Transparency:** Blackbox — no audit or compliance tests required. Internal implementation details
need not be disclosed.

**Available Scenarios:** Client on Prem (CoP) or Client over Network (CoN). Server may be
self-hosted, hybrid, or cloud-hosted. CoP and CoN are not reported as separate sub-divisions.

#### 2.4.1 Rules

- Must use the standard MLPerf Endpoints performance and accuracy datasets.
- Must report the same metrics as Standardized and Serviced divisions (System TPS, TPS/User, TTFT
  P50/P90) using the same measurement methodology.
- Must use the same base reference model. RDI submissions may augment the model by pruning,
  sparsification, quantizing, fine-tuning, modification of speculative decoding heads, and
  alternative attention mechanisms.
- No audit or compliance tests required. No code visibility requirement.
- Submitters must report achieved accuracy on the accuracy dataset.

For RDI publication status and the cooling-off period for RDI hardware transitioning to Available or
Preview, see
[Submission Rules §7.4](endpoints_submission_rules.md#74-rdi-research-development-or-internal).

#### 2.4.2 Result Naming

Results must use the qualified name "MLPerf Endpoints RDI." Example: *"MLPerf Endpoints RDI result
of 8,000 tokens/s at concurrency 128."*

---

### 2.5 Division Summary

| Property | Standardized | Serviced | RDI |
|---|---|---|---|
| Transparency | Whitebox | Greybox | Blackbox |
| Scenarios | CoP, CoN (separate) | CoN only | CoP or CoN (single) |
| Model Equivalence | Required | Augmentation allowed | Augmentation allowed |
| Code Visibility | Full | API-level | None |
| Audit / Compliance | Yes | Yes (audit + accuracy) | No |
| Retraining Allowed | No | Yes (with disclosure) | Yes |
| Public Availability | Not required | Required (GA service) | Not required |

---

### 2.6 Reproducibility Requirements

| Division | By Any 3rd Party (MLC Peer) | On 3rd Party System / Datacenter | On Publicly Available Endpoint |
|---|---|---|---|
| Standardized — CoP | Required | Required | N/A |
| Standardized — CoN | Required | Required | N/A |
| Serviced — CoN | Required | Optional | Required |
| RDI | Optional | Optional | Optional |

- **By Any 3rd Party (MLC Peer):** A review committee member or designated auditor can reproduce the
  benchmark result using the submitted materials. For Standardized, this means building from
  provided source code and configurations. For Serviced, this means running the benchmark against
  the public endpoint.
- **On 3rd Party System / Datacenter:** For Standardized, the submitter must provide containers,
  drivers, and code sufficient to replicate the setup. For Serviced, this is optional because the
  endpoint is accessed remotely regardless of client location.
- **On Publicly Available Endpoint:** Only applicable to the Serviced division, where the endpoint
  must be a generally accessible commercial service.

---

### 2.7 Transparency Requirements

**Source Code**

| Division | Code to Reproduce On-Prem | Code to Reproduce Remote Server |
|---|---|---|
| Standardized — CoP | Required | Required |
| Standardized — CoN | Required | Required |
| Serviced — CoN | N/A | Optional |
| RDI | Optional | Optional |

**Hardware**

| Division | All Rack / Node Hardware | Primary Accelerator Details | Mapping Configurations (TP, EP, PP, Batching) |
|---|---|---|---|
| Standardized — CoP | Required | Required | Required |
| Standardized — CoN | Required | Required | Required |
| Serviced — CoN | Optional | Required | Optional |
| RDI | Required | Required | Optional |

**Hardware details:** accelerator model, count, memory capacity, host CPU/memory, and the
interconnect type and topology (e.g., NVLink, InfiniBand, Ethernet, routing layer). For Serviced
submissions, full rack hardware disclosure is optional, but the primary accelerator must be
identified.

**Software / deployment configuration:** parallelism mapping (e.g., tensor parallelism `TP`, expert
parallelism `EP`, pipeline parallelism `PP`, sequence parallelism, data parallelism), batch sizes,
scheduling parameters, KV cache configuration, and any other parameters that materially affect
throughput or latency. These must be fully disclosed for Standardized division submissions.

---

### 2.8 Tokenizer Rules

Tokenizers can produce different token counts depending on how text is fed to them — the same output
text tokenized as a single string versus tokenized as a sequence of streamed chunks can yield
different counts, even with the same tokenizer. To ensure consistent and representative measurement
across divisions:

- The **reference tokenizer** — defined as the tokenizer published with the benchmarked model in its
  canonical Hugging Face repository — produces the canonical token count for the system under
  measurement. Complete-response token-count metrics such as `system_tps` are computed from the
  reference tokenizer applied to the fully reconstructed assistant response through the benchmark
  model's official reference chat template, not from any tokenizer used internally by the SUT.
- **Complete-response token counts are obtained by applying the reference tokenizer once per
  completed assistant response.** Visible-output and reasoning fragments are reassembled in arrival
  order, while tool-call fragments are reassembled by tool-call list index as defined below. The
  reconstructed visible output, reasoning, and structured tool calls are supplied together as one
  assistant message to the official reference chat template and tokenized once. Counts are *not* the
  sum of per-chunk or per-streamed-token counts observed during generation.
  - *Fairness:* every submitter is scored against the same tokenizer applied the same way,
    independent of how their system batches, chunks, or streams during generation.
  - *Representativeness:* this measures the benchmark model's canonical representation of the
    completed assistant response, rather than implementation artifacts of streamed token boundaries
    that can produce different token counts across submitters.

**Input sequence length (ISL).** For each request, the reference client first applies all
benchmark-required input preprocessing, including request construction, salting where required, and
the benchmark model's official reference chat template. The fully preprocessed input is then
tokenized with the reference tokenizer using the generation-prompt and special-token behavior
defined by the reference implementation. If the reference implementation truncates the tokenized
input, ISL is the length after that truncation. This sequence replicates the input tokenization
expected on the server side; special tokens inserted by the reference chat template are included in
ISL. Where model-equivalence rules apply, a submitter may use a different implementation only if it
produces the same effective prompt and token IDs. The generated benchmark report includes ISL
statistics computed from these per-request values.

**Response categories and what is counted.** A model response can carry three kinds of content —
user-visible **output**, **tool-call** content, and **reasoning** (thinking) traces. Serving
frameworks convert the raw generation into structured OpenAI chat-completion objects received by the
reference client. For visible output and reasoning, the client concatenates all text fragments in
arrival order. Tool calls are reassembled into structured objects by list index as described below.
The client supplies all three reconstructed fields together as one assistant message to the
benchmark model's official reference chat template and tokenizes the rendered message once.

Category assignment is mutually exclusive: the reference client MUST assign each received text
fragment to exactly one response field, and a fragment MUST NOT contribute more than once to the
reconstructed assistant message. Deliberately duplicating substantially identical content across
fields, or adding padding content to any field, for the purpose of increasing reported token counts
is prohibited and invalidates the affected measurement point.

For parallel tool calls, the client maintains a list keyed by the tool-call index supplied by the
response protocol. Function-argument fragments are appended in arrival order within their
corresponding index; the protocol-provided call ID, type, and function name are retained. After the
response completes, the structured tool calls are ordered by ascending list index. If
`function.arguments` is a JSON string encoding an object, the client parses it to that object for
the reference chat template; otherwise, it preserves the received value. If the transport provides
separate streams for parallel tool calls, each stream is reassembled independently before the
completed calls are ordered. The resulting structured tool-call list is supplied, together with
assistant visible output and reasoning, to the official reference chat template.

The assistant-payload token count excludes empty chat-template framing. The reference client renders
and tokenizes both (a) a minimal reference conversation containing an empty user message followed by
the reconstructed assistant response and (b) the same conversation with an empty assistant message.
The official output-token count is `max(0, count(a) - count(b))`. Both renders use
`add_generation_prompt = false`. Tokens introduced specifically to represent the reconstructed
assistant payload, including model-defined reasoning or tool-call framing, are counted; framing
already present for an empty assistant message is not.

| Content category | Counted? | How it is measured |
|---|---|---|
| Visible output | Yes | Text fragments concatenated in arrival order and supplied as assistant `content` |
| Tool-call content | Yes | Fragments reassembled into structured calls, ordered by ascending tool-call index, and supplied as assistant `tool_calls` |
| Reasoning / thinking | Yes | Text fragments concatenated in arrival order and supplied as the assistant reasoning field expected by the reference chat template |
| Chat-template framing / special tokens | Conditional | Payload-specific reasoning and tool-call framing inserted by the official reference chat template is counted; framing present for an empty assistant message is excluded by the baseline subtraction above |

- Because frameworks differ in their internal serialization, the reconstructed assistant message may
  not be byte-identical to the server's raw generation. Rendering the received structured response
  with the official reference chat template provides one model-specific, reproducible representation
  for scoring every submitter.
- Complete-response token-count metrics such as System TPS are derived from this single
  assistant-payload count. TPOT instead uses the suffix after the first output-bearing streamed
  chunk; that suffix is tokenized once with the reference tokenizer, and non-positive or non-finite
  samples and non-streaming responses are excluded. TTFT remains a latency measurement and is not
  derived from token counts: it is measured from query issuance until the client receives the first
  non-empty text fragment (`len(s) > 0`) in any response category (visible-output, tool-call, or
  reasoning).
- Submitters may use any tokenizer internally for output generation or accounting; that output-side
  choice does not affect scoring. The official output-token count is always produced by the
  **client-side reference tokenizer applied once to the reconstructed assistant message through the
  official reference chat template**. No equivalence demonstration or mapping factor is required for
  an internal output tokenizer. This output-scoring rule does not waive the input tokenization and
  preprocessing equivalence requirements in [§2.9.2](#292-pre-processing-equivalence).

> [!NOTE]
> **[WIP]** — One edge case remains under working-group development: partial Unicode at chunk boundaries.

---

### 2.9 Model Equivalence Rules (Standardized Division)

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`** Endpoints model-equivalence and
> optimization rules **inherit from**
> [MLPerf Inference Rules §Model Equivalence](https://github.com/mlcommons/inference_policies/blob/master/inference_rules.adoc#model-equivalence).
> The subsections below restate the inheritance and call out the Endpoints-specific deltas (most
> notably KV-cache reuse in [§2.9.5](#295-kv-cache-rules), dynamic approximate sparsity in
> [§2.9.6.4](#2964-dynamic-approximate-sparsity), and drafter PTQ in
> [§2.9.4](#294-speculative-decoding)). Where this section conflicts with upstream, this section is
> the source of truth for Endpoints submissions.

These rules define what it means for a Standardized division submission to be "model equivalent" to
the reference implementation. The subsections below define which implementation choices are
permitted; the accuracy quality target (§4.3) then determines whether a permitted approximation is
acceptable in a given submission. Passing the accuracy gate is necessary but not sufficient — see
[§2.9.8](#298-accuracy-gate).

#### 2.9.1 Reference Implementation

> [!NOTE]
> **[WIP — align with inference_rules.adoc §reference-implementation]**

Each benchmark has a **reference implementation** published in the MLPerf Endpoints reference
repository. The reference implementation defines:

- The canonical model weights and the reference tokenizer (the tokenizer published with the model on
  Hugging Face).
- The **canonical attention pattern**, including any architecture-native sparsity configuration —
  block size, token-selection rule, local/global window structure, and any learned or heuristic
  selection module — where the canonical architecture specifies sparse attention. See
  [§2.9.6.3](#2963-canonical-architectural-sparsity).
- The required input and output format.
- The **dataset** used for performance and accuracy runs (Hugging Face dataset ID or download URL,
  plus the canonical split and any preprocessing recipe).
- The **reference chat template** (Hugging Face chat-template string or the equivalent
  message-formatting spec). Submissions MUST use the reference chat template; alternative templates
  that produce different tokenized output are not permitted.
- The **reference server / sampling parameters**: temperature, top-k, top-p, repetition penalty,
  greedy-vs-stochastic decoding flag, max output tokens, stop sequences. These MUST be set per the
  benchmark definition; submissions MUST NOT modify them.
- The **approved drafter list** for the benchmark, if any — one or more approved draft models/heads,
  each with its ID, precision, algorithm, and default per-point configuration. A native head shipped
  inside one of the benchmark's approved checkpoints is approved with that checkpoint and need not
  be listed separately ([§2.9.4](#294-speculative-decoding)). Submitters select any approved
  drafter; the list is published and versioned per submission round alongside the model list (see
  [§3.2](#32-supported-models)). See [§2.9.4](#294-speculative-decoding) for eligibility, approval,
  and verification requirements.
- The accuracy evaluation methodology and quality target.
- The endpoint API interface.
- **Fixed configuration:** Configuration parameters explicitly designated as fixed by the reference
  implementation MUST NOT be modified by submitters.

An **alternative reference implementation** may be designated by the working group for a specific
architecture or hardware class, subject to passing the same accuracy quality target as the primary
reference implementation.

#### 2.9.2 Pre-Processing Equivalence

> [!NOTE]
> **[WIP — align with inference_rules.adoc closed division pre-processing rules]**

The server-side processing of each incoming request — both input pre-processing and output
post-processing — must be functionally equivalent to the reference implementation:

- **Tokenization:** Must produce the same token IDs as the reference tokenizer for the same input
  text. Submitters using an alternative tokenizer implementation must demonstrate token-for-token
  equivalence on the accuracy dataset.
- **Chat template / prompt formatting:** The system prompt, user turn formatting, special tokens
  (BOS, EOS, role markers), and chat-template flags (for example, `enable_thinking`,
  `clear_thinking`, and `preserve_thinking`) must match the benchmark specification. Modifications
  that change the effective input to the model are not permitted.
- **Input truncation:** If the reference implementation truncates inputs that exceed the model's
  context window, the submitter's truncation method must produce the same result.

#### 2.9.3 Model Weight Rules

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`**

All Standardized division submissions must begin from the **canonical model weights** specified in
the benchmark definition (identified by Hugging Face model ID or a published checksum).

Per [§2.2.1](#221-general-rules), weight transformations are governed by the inherited MLPerf
Inference rules. The following transformations of the canonical weights are **disallowed**:

- **Block-sparse weight pruning, unstructured pruning, or any operation that discards non-zero
  weight elements *without* a mathematically equivalent replacement.** Pruning that produces
  mathematically equivalent results to a dense reference (e.g., a dense matmul replaced by a sparse
  matmul that yields the same outputs) inherits the upstream "Replacing dense operations with
  mathematically equivalent sparse operations" allowance and is *not* a disallowed pruning. See
  [§2.9.6.2](#2962-exact-sparse-execution-and-softmax-elision).
- **Fine-tuning, LoRA, adapter layers, RLHF, or any gradient-based update of weights.** Applies
  equally to the canonical model and to any draft model used in speculative decoding (see
  [§2.9.4](#294-speculative-decoding)).
- **Retraining from scratch, continued pre-training, or knowledge distillation to a different architecture.**
- **Modifying weights during the timed portion of an inference run** (online learning).
- **Weight-quantization algorithms whose specification is similar in size to the non-zero weights
  they produce** (inherited from upstream — defeats principled-quantization intent).

**Post-training quantization (PTQ).** PTQ is the canonical *permitted* weight transformation,
inherited from upstream. PTQ-style methods (AWQ, GPTQ, bitsandbytes) and arbitrary numerical formats
(INT8/INT4/FP8 and similar) are permitted provided they (a) use only the published calibration set,
(b) are publicly described to a level at which they could be reproduced, (c) pass the accuracy gate
([§2.9.8](#298-accuracy-gate)), and (d) are disclosed in the submission YAML. The same conditions
govern PTQ applied to a speculative-decoding drafter ([§2.9.4](#294-speculative-decoding)).

Submitters may either derive a quantized checkpoint from the reference checkpoint using only the
MLPerf-provided calibration data set, or use a publicly available checkpoint pre-approved by the
MLPerf Endpoints Rules Task Force or Working Group.

**Components of the canonical checkpoint.** Permitted weight transformations MUST preserve the
component set of the canonical checkpoint. A submission's derived checkpoint — quantized or
otherwise transformed — MUST contain every component the reference checkpoint ships, including
auxiliary prediction heads and speculative-decoding modules such as MTP or EAGLE-style heads.
**Stripping such a component from the derived checkpoint is not permitted**, even where the
component does not participate in producing output and even where the submission does not intend to
use it. The derived checkpoint must remain a faithful transformation of the canonical one, so that
reviewers can verify provenance against the published model ID or checksum and, where applicable,
re-run the submission with the component enabled.

Declining to *use* a component at run time is a separate matter, governed by
[§2.9.4](#294-speculative-decoding): a submission MAY disable speculative decoding at some or all
measurement points, provided the drafter remains present in the submitted checkpoint and the
per-point configuration is declared in the submission YAML.

Statically removing weights that *do* participate in the forward pass for some inputs is pruning,
and is disallowed above and in [§2.9.6.5](#2965-disallowed).

**Residency is not required.** The rule above governs the checkpoint *artifact*. It does not require
a component to be loaded into accelerator memory during measurement. A submitter is **not obligated
to load or copy every weight in the checkpoint to the accelerator** where the component is not
needed for the configuration being measured — a submission that has legitimately disabled
speculative decoding at a point ([§2.9.4](#294-speculative-decoding)) need not reserve memory for
the drafter at that point.

The two obligations are therefore distinct, and both hold:

| | Requirement |
|---|---|
| **Checkpoint artifact** | MUST contain every component of the canonical checkpoint. Stripping is not permitted, whether or not the component is used ([§2.9.6.5](#2965-disallowed)). |
| **Accelerator residency** | Not required. What is loaded at run time is a submitter configuration choice, constrained only by the equivalence rules governing what is *computed*. |

Memory footprint is a legitimate dimension of a serving configuration, and the artifact requirement
is what preserves provenance and the ability to re-run the submission with the component enabled.
Nothing here licenses any change to the computation: a component that participates in the forward
pass for the configuration being measured must be present and used.

**Dead code elimination does not reach the checkpoint.** Upstream §Model Equivalence lists *dead
code elimination* among its permitted optimizations, without qualification. An unused auxiliary
component — an MTP or EAGLE-style head at a point where speculative decoding is disabled — is dead
code in the ordinary compiler sense, and the upstream allowance might be read to license deleting it
from the derived checkpoint. **It does not.** Under [§2.2.1](#221-general-rules) this document
overrides upstream wherever the two conflict, and the allowance is confined to the executed graph
([§2.9.6.2](#2962-exact-sparse-execution-and-softmax-elision)). The submitted artifact is not code
and is not subject to elimination: it is the evidence of provenance against which the submission is
verified. Eliminating such a component from the *loaded* graph is the residency question answered
above, and is permitted.

#### 2.9.4 Speculative Decoding

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`**

**v1.0 change.** Speculative decoding is now governed by a **curated approved-drafter-list model,
per benchmark**, rather than a single fixed drafter designated by the benchmark definition.
Speculative decoding is permitted for any benchmark for which the benchmark task force has approved
one or more drafters, following the process below, or whose approved checkpoint ships a native head
(see *Native heads* below). Benchmarks with no approved drafter continue to disallow speculative
decoding entirely.

List of approved drafter list is available [here](https://github.com/mlcommons/endpoints-submission-cli/blob/main/src/submission_checker/data/approved_drafters.yaml).

**Approved drafter list.** Each benchmark's set of eligible drafters (MTP head, EAGLE-style head, or
analogous module) is a **curated, published list**, not a single fixed drafter:

- The benchmark task force seeds the list with ≥ 1 approved head/drafter per benchmark model where feasible.
- Submitters (and any WG member) may propose additional drafters via a standing intake process.
  Proposal review is **WG review by default**; on escalation, the benchmark task force takes over
  the review and returns its recommendation to the WG for ratification.
- A newly approved drafter may first be used in a submission where the review eligibility date for a
  submission is **at least two cohorts dates after** the cohort in which the drafter was approved.
  Approval is recorded against the cohort in which the updated list is published.
- The approved list is published in the reference repository, versioned per submission round,
  alongside the model list (see [§3.2](#32-supported-models)).

**Native heads.** A speculative-decoding head (for example an MTP, DSpark, or EAGLE-style head) that
the model publisher ships inside a checkpoint approved for the benchmark is **approved with that
checkpoint**, in the checkpoint's approval cohort, and needs no separate list entry. The reference
repository's list of approved checkpoints notes which of them ship a native head — for Agentic
Inference, see
[Approved Checkpoints and Speculative-Decoding Heads](https://github.com/mlcommons/endpoints/blob/main/examples/10_Agentic_Inference/README.md#approved-checkpoints-and-speculative-decoding-heads).

The following are **not allowed** with a native head:

- **A head that is not the canonical checkpoint's own.** A head added, replaced, or retrained by
  anyone other than the canonical model's publisher — including in a third-party quantized
  checkpoint — or added after the approved revision, is not native and requires an entry on the
  approved list.
- **Modifying the head**, other than *PTQ on drafter weights* below.
- **Using it as a different kind of drafter** than its publisher documents (e.g., running an MTP
  head as an EAGLE-style head). Per-point configuration may vary as for any approved drafter.
- **A head the WG has excluded** for failing *Drafter eligibility* below.

**Drafter eligibility.** The following disqualify a drafter from the approved list:

- **Not open-weight.** The weights are not downloadable by anyone who agrees to the publisher's
  license terms — private hosting, or access requiring manual/discretionary approval, disqualifies.
  An automatic license click-through with the same terms and instant access for anyone (e.g., Meta's
  Llama license gate) does not disqualify.
- **Input-based optimization ([§2.2.1](#221-general-rules)).** Build-time incorporation of the
  benchmark performance dataset's content — deliberate fine-tuning or distillation on it.
  (Incidental pretraining-corpus overlap does not disqualify; a corpus-disclosure or release-date
  test is not required of the proposer.)
- **Trained for benchmark performance.** A drafter trained, fine-tuned, or distilled specifically to
  perform well on this benchmark — including against benchmark-like traffic that reproduces its task
  mix, prompt style, or length distribution — even where the benchmark dataset itself was never
  used. A drafter published for general use, and adopted for the benchmark because it happens to
  suit it, does not disqualify.
- **Undisclosed or QAT-style quantization.** Quantization-aware training, or post-training
  quantization without disclosure of the calibration set and methodology, per
  [§2.9.3](#293-model-weight-rules).

Submitters select any drafter from the benchmark's approved list, or a native head of one of its
approved checkpoints. Using any other drafter is not permitted.

A list entry identifies a drafter in one of two ways:

- **Weight-identified** — a distinct draft model or head, identified by its model ID and weight checksum.
- **Configuration-identified** — a drafter that introduces no separate weights, such as a
  self-speculative or early-exit pass through a subset of the target's own layers. It is identified
  by the target checksum together with the exit-layer and any other configuration defining the draft
  pass. See [§2.9.9 Q7](#299-qa-model-equivalence-clarifications).

The following are **also disallowed** at run time:

- **Approximate speculative-decoding methods that alter the output distribution.** The verification
  step MUST NOT introduce acceptance criteria that would cause the model to accept tokens the target
  would not have generated. Outputs MUST be token-for-token identical to what the target model would
  generate without speculation.
- **Approximating, skipping, or replacing the verification step**, including replacing the target
  with a secondary drafter for verification. The target model in the verification step MUST be the
  canonical model with the permitted transformations of [§2.9.3](#293-model-weight-rules) applied.
- **Modifying an approved drafter.** Fine-tuning, LoRA, adapter layers, RLHF, continued
  pre-training, or any other gradient-based update to a drafter by the submitter. The drafter is
  used as published on the approved list; a modified drafter is no longer that drafter, and is
  therefore not on the list. Post-training quantization is the one permitted transformation — see
  *PTQ on drafter weights* below.

**Disclosure and run-time requirements:**

- The drafter identity (name, version, source URL), precision, algorithm, and per-point
  configuration MUST be declared in the submission YAML.
- All measurement points on a submission's pareto curve for a given benchmark MUST use the same
  drafter (same head, same algorithm). Different **configurations** of the same drafter (e.g.,
  varying `speculative-num-steps` or `speculative-eagle-topk`) are permitted across pareto points,
  including disabling speculation entirely at some points. The drafter itself is fixed across the
  curve. The configuration values used at each point MUST be declared in the submission YAML, and
  any dynamic variation within a single point's run MUST be reported as a distribution.

**PTQ on drafter weights.** The drafter weights MAY be post-training quantized under the same
conditions as the canonical model ([§2.9.3](#293-model-weight-rules)): calibration-only, using only
the published calibration set, no gradient updates, disclosed in the submission YAML, and subject to
the accuracy gate. The drafter MUST NOT be modified in any other training-side sense — see *Drafter
eligibility* above.

**Leaving the drafter unused.** A submission is not required to load or use a drafter shipped with
the canonical checkpoint; see [§2.9.3](#293-model-weight-rules). Where a benchmark has no approved
drafter — no list entry and no native head of an approved checkpoint
([§2.9.1](#291-reference-implementation)) — speculative decoding is not available for that benchmark
at all.

**Model equivalence for drafters.** No drafter-specific equivalence standard applies. A
speculative-decoding submission is model equivalent on the same two general tests as any other
optimization:

- The **submission** must meet the accuracy quality target ([§2.9.8](#298-accuracy-gate)) with its
  speculative-decoding configuration in force. Because exact verification requires token-for-token
  identity with the unspeculated target, a compliant configuration does not move accuracy; a failure
  here indicates the verification requirement above was not met.
- The drafter must be free of **input-based optimization**, per the eligibility criteria above and [§2.2.1](#221-general-rules).

#### 2.9.5 KV Cache Rules

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`** This section **intentionally diverges from
> MLPerf Inference §KV-Cache**, which prohibits cross-query KV reuse. Endpoints targets
> agentic-style workloads where a shared system prompt across queries is the norm; prohibiting
> cross-query reuse would force submitters to artificially cripple production-style serving stacks.
> The salt mechanism in [§2.9.5.1](#2951-salting-mechanism) preserves measurement validity by
> ensuring caches cannot leak context beyond the system-prompt prefix.

Per [§2.2.1](#221-general-rules), KV-cache management is governed by the inherited MLPerf Inference
rules with the Endpoints-specific cross-query-reuse delta described below. The following KV-cache
techniques are **disallowed**:

- **Response caching that bypasses the forward pass.** Returning a cached response verbatim to a
  request that matches a previous request is not permitted. Every request must execute the forward
  pass. (Cross-query KV-cache reuse — covered by the Endpoints delta below — is *not* response
  caching: it still executes the forward pass on a salt-uniquified per-query token stream.)
- **KV-cache compression methods that are not in the reference implementation and that have not been
  disclosed in the submission.** Compression methods that are part of the reference or a designated
  alternative implementation are permitted by default; submission-specific compression methods are
  subject to Methodology objections during peer review even after disclosure.

**Endpoints-specific delta — cross-query KV reuse:** Sharing KV cache state across independent
requests — including prefix / prompt caching of the shared system prompt — is permitted as a serving
optimization, with no requirement of bit-for-bit output identity vs. an un-cached run and no
requirement of cross-user partitioning. The performance dataset injects a per-query salt between the
shared system prompt and the per-query user context (see [§2.9.5.1](#2951-salting-mechanism)). The
salt guarantees that the only prefix two queries can share is the system prompt itself; any KV state
derived from the user context cannot be reused across queries with different contexts.

**Disclosure requirements:**

- If the KV cache is stored at reduced precision (e.g., INT8, INT4, FP8 KV), the precision and
  quantization method MUST be disclosed in the submission YAML.
- Any KV-cache compression method that is not part of the reference implementation MUST be disclosed
  in the submission YAML.
- Paged / virtual KV cache implementations (e.g., vLLM's PagedAttention) are inherited as permitted
  under the upstream "Different in-memory representations" allowance and do not require separate
  disclosure beyond what is already captured in the serving-framework / software-stack disclosure.

##### 2.9.5.1 Salting Mechanism

The performance benchmark workload prepends a unique, deterministic-but-pseudorandom **salt** to
each per-query user prompt at request-construction time. The salt:

- **MUST** carry at least 64 bits of entropy per query.
- **MUST** be generated from a seeded pseudo-random sequence (e.g., `random.Random(seed)`) where the
  seed is declared in the run configuration. This makes the salt sequence reproducible across runs
  with the same seed while still preventing cross-query KV reuse beyond the system prompt.
- **MUST** be inserted *between* the system prompt and the user-context portion of the prompt, so
  the system prompt remains a shared prefix across queries (and is therefore cacheable as the
  legitimate optimization this section permits) while the user-context portion becomes
  per-query-unique.
- **MUST** be generated at request-construction time, **not** stored in the dataset on disk, so that
  repeated runs of the same dataset always produce a per-query-unique salt sequence regardless of
  how many times the dataset is replayed. (Storing salt in the dataset would lose uniqueness across
  replays — see [endpoints PR #305](https://github.com/mlcommons/endpoints/pull/305) for the
  reference rationale.)
- **SHOULD** use the reference implementation in `mlcommons/endpoints`
  (`Dataset.with_salt(random.Random(seed))`, introduced in
  [endpoints PR #305](https://github.com/mlcommons/endpoints/pull/305)).

The operative requirement is that the token stream actually seen by the SUT contains a unique
per-query salt between the system prompt and the user context — not the *means* by which the client
constructs that stream.

**Clients that pre-tokenize.** A client that pre-tokenizes prompts (e.g., SGLang-style adapters that
send `input_tokens` rather than text) MUST ensure the *token stream* it sends to the SUT contains
the unique per-query salt between the system-prompt tokens and the user-context tokens. Two
conforming approaches: (a) apply the salt to the text and re-tokenize the result before sending, or
(b) reserve a salt-marker token ID (or short sequence) and emit it inline. Applying the salt only to
a `prompt` text field while sending the original `input_tokens` will *not* prevent KV reuse — the
SUT never sees the text — and is non-compliant. The reference implementation in
`mlcommons/endpoints` follows path (a); see the warning logged by `Dataset._apply_salt` in
[endpoints PR #305](https://github.com/mlcommons/endpoints/pull/305) for the contract.

**Accuracy runs use the un-salted reference dataset** to ensure model output matches the canonical
implementation exactly. Submissions are not required to disable cross-query KV reuse in their
serving stack for accuracy runs; the accuracy dataset simply omits the salt prefix, and the serving
stack reuses KV as it would in production. This split (salted performance dataset, un-salted
accuracy dataset) is the operational mechanism that allows blanket cross-query KV reuse without
compromising the accuracy gate's role as a model-output check.

> [!NOTE]
> Agentic benchmarks use benchmark-specific random salting to control KV-cache reuse. The mechanism
> is controlled by benchmark-specific flags and MUST be enabled for submissions.

> [!NOTE]
> **Backward compatibility note.** This rule intentionally diverges from MLPerf Inference's KV-cache
> FAQ, which states KV state "does not apply across queries". Endpoints submissions are not portable
> to standard MLPerf Inference without disabling cross-query KV reuse; conversely, MLPerf Inference
> submissions that already prohibit cross-query reuse are trivially compliant with this section.
> Submitters should treat the two rule sets as **not** mutually compatible for code paths that rely
> on this delta.

#### 2.9.6 Sparsity and Approximate Computation

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`** This section consolidates the Endpoints
> treatment of sparse execution, softmax elision, and runtime approximation. It **intentionally
> diverges from MLPerf Inference**, which admits only mathematically equivalent sparse operations:
> [§2.9.6.4](#2964-dynamic-approximate-sparsity) permits bounded runtime approximation under the
> accuracy gate.

##### 2.9.6.1 Scope and Operative Test

This section governs techniques that skip, elide, or approximate part of the computation at run time
— sparse execution paths, softmax elision, attention-score thresholding, and activation
thresholding. It does not govern transformations of the stored weights, which remain subject to
[§2.9.3](#293-model-weight-rules).

The operative test throughout is whether the technique **changes the tokens the model emits for any
input**:

- Techniques that provably do not are permitted without an accuracy-gate condition
  ([§2.9.6.2](#2962-exact-sparse-execution-and-softmax-elision),
  [§2.9.6.3](#2963-canonical-architectural-sparsity)).
- Techniques that may are permitted only under [§2.9.6.4](#2964-dynamic-approximate-sparsity): they
  must pass the accuracy gate and be disclosed.
- Techniques that change the model's structure rather than approximating its computation are
  disallowed ([§2.9.6.5](#2965-disallowed)).

Whether a technique is implemented in software or accelerated by hardware is not a criterion. Sparse
tensor cores, block-sparse kernels, and gather/scatter dispatch are implementation choices governed
by what they compute, not by which units execute them; they are permitted on the same basis as
arbitrary frameworks and kernels ([§2.2.1](#221-general-rules)) and the upstream "different
in-memory representations" allowance inherited in [§2.9.5](#295-kv-cache-rules). Neither the
sparsity ratio achieved at run time nor the hardware's supported sparsity granularity is itself a
compliance criterion.

##### 2.9.6.2 Exact Sparse Execution and Softmax Elision

The following are mathematically equivalent to the reference computation. They are permitted with no
accuracy-gate condition and no disclosure beyond the software-stack listing of
[§8.4](#84-software-disclosure). Dead code elimination carries two further constraints, set out
after the list:

- **Mathematically equivalent sparse operations.** Replacing a dense operation with a sparse
  operation that produces the same outputs — for example a dense matmul executed as a sparse matmul,
  or skipping blocks whose values are exactly zero. Inherited from upstream §Model Equivalence.
- **Fused, streaming, and online softmax.** FlashAttention-style running max/sum, fused softmax
  kernels, log-sum-exp rearrangement, and max-subtraction for numerical stability.
- **Vocabulary softmax elision under greedy decoding.** Softmax is monotonic, so
  `argmax(softmax(logits)) == argmax(logits)`. Where the benchmark's reference sampling
  configuration is greedy (temperature = 0, per [§2.9.7](#297-post-processing-equivalence)), taking
  argmax over raw logits and skipping normalization entirely is exactly equivalent.
- **Softmax elision in speculative-decoding verification** where the target's sampling configuration
  is greedy, since acceptance reduces to comparing argmax.
- **Dead code elimination.** Removing graph nodes, kernels, or branches that cannot affect the
  output of the configuration being served — constant folding, identity-op removal, elimination of
  unreachable branches. Inherited from upstream §Model Equivalence, where it is listed without
  qualification. In Endpoints it is an *execution* allowance and is bounded by
  [§2.9.3](#293-model-weight-rules) and by the invariance requirement below: it does not extend to
  the checkpoint artifact.

Bit-exact identity is not required — ordinary floating-point reassociation is expected.

**Dead code elimination MUST be data-invariant.** The eliminated computation must be incapable of
affecting the output for **any admissible input** and for **any checkpoint** satisfying
[§2.9.3](#293-model-weight-rules) — not merely for the benchmark's dataset or for the reference
checkpoint's particular weight values. Two eliminations that look identical in a profile are treated
differently:

| Elimination | Status |
|---|---|
| A branch that is unreachable for every input the API admits. | **Permitted.** The removal is a property of the program. |
| A branch the benchmark's inputs happen never to take. | **Disallowed.** This is input-based optimization under [§2.2.1](#221-general-rules) — the implementation encodes knowledge of the dataset's content. |
| A computation compiled out because a tensor is zero or constant *in this checkpoint*. | **Disallowed.** The artifact is baked into the graph, and the elimination would not survive a re-quantized or re-released checkpoint. |
| A block skipped at run time because its values are exactly zero on that pass. | **Permitted** (first bullet above). This is a dynamic property of the data being processed, not a static assumption compiled into the implementation. |

The distinction throughout is **static versus dynamic**, the same line drawn in
[§2.9.6.3](#2963-canonical-architectural-sparsity) and [§2.9.6.5](#2965-disallowed): deciding once,
at build time, on the basis of the benchmark's data or weights is disallowed; deciding per pass,
from the values actually present, is permitted.

**Obligation to report data-dependent dead code in the reference.** A submitter who identifies
computation in the reference implementation that is dead **only** for the benchmark's data, or only
for the reference checkpoint's particular weights, MUST report it to the working group rather than
silently eliminating it. Such code is a property of the reference, not of any one submission: its
presence may indicate a reference defect, or an unintended advantage for whichever implementations
detect it first. It is resolved by changing the reference for every submitter
([§2.9.1](#291-reference-implementation)), not by each submitter's compiler. Reporting is required
whether or not the submitter intends to exploit it.

##### 2.9.6.3 Canonical Architectural Sparsity

Where the canonical architecture is itself sparse, implementing that sparsity is **required** for
equivalence rather than being a disallowed transformation. The reference point is the canonical
model's own computation, not a dense idealization of it.

- **Native sparse attention.** Architectures that specify sparse attention in the reference
  architecture define the pattern submissions must implement, including its sparsity configuration
  ([§2.9.1](#291-reference-implementation)). Substituting a different structural sparse pattern for
  the canonical one is disallowed under [§2.9.6.5](#2965-disallowed) even where the canonical model
  is itself sparse.
- **Canonical sparse routing.** Where the architecture activates a subset of weights per token by
  design — for example a mixture-of-experts model routing each token to a fraction of its experts —
  touching only the routed weights on a given forward pass *is* the canonical computation. That the
  majority of expert weights go untouched on any individual token is not grounds for an objection.

##### 2.9.6.4 Dynamic Approximate Sparsity

Sparsity derived at run time from live attention scores or activations is permitted in the
Standardized division even where it does not preserve outputs exactly. This is an Endpoints-specific
delta from upstream. It covers:

- **Attention-score thresholding.** Within the online-softmax loop, a key/value block whose local
  maximum score falls more than `ln(λ)` below the running maximum is treated as contributing
  negligible post-softmax mass, and that block's exponential, value-block load from HBM, and
  attention-weight × value product are skipped. The reference technique is softmax thresholding as
  described in BLASST ([arXiv:2512.12087](https://arxiv.org/abs/2512.12087)).
- **Activation thresholding.** Skipping computation because an activation block falls below a
  magnitude threshold. Skipping computation because an activation block is exactly zero is instead
  governed by [§2.9.6.2](#2962-exact-sparse-execution-and-softmax-elision).

Methods in this category are subject to all of the following:

- They MUST pass the benchmark's accuracy quality target on the un-salted accuracy dataset. The
  accuracy gate ([§2.9.8](#298-accuracy-gate)) is the arbiter of whether the approximation is
  acceptable.
- They MUST NOT be calibrated on the benchmark performance or accuracy dataset. Threshold selection
  must use the published calibration set or a data-independent procedure; calibrating against
  benchmark inputs is input-based optimization and is disallowed under [§2.2.1](#221-general-rules).
- They MUST be disclosed per [§2.9.6.6](#2966-disclosure).
- They apply to the computation only. This allowance does not license discarding weight elements;
  weight transformations remain governed by [§2.9.3](#293-model-weight-rules).

**Interaction with speculative decoding.** Where the target model uses an approximate method under
this section, the token-for-token identity requirement of [§2.9.4](#294-speculative-decoding) is
evaluated against the submission's own target-model configuration: the drafter and verification step
must introduce no divergence beyond the disclosed approximation.

##### 2.9.6.5 Disallowed

- **Structural attention-pattern changes.** Adding sink tokens not present in the canonical
  architecture, swapping in a different attention mask, imposing a fixed or precomputed sparsity
  pattern on a canonically dense attention layer, substituting a different structural sparse pattern
  for the canonical one, or otherwise changing the pattern the canonical model uses. Dynamic
  score-derived sparsity is governed by [§2.9.6.4](#2964-dynamic-approximate-sparsity) and is not a
  structural change.
- **Normalization substitution.** Replacing softmax with a different normalization not present in
  the canonical architecture (e.g. linear or ReLU attention) — an architecture substitution rather
  than an approximation of the canonical attention.
- **Weight-side sparsity to reach a hardware sparse path.** Pruning, zeroing, or restructuring
  weights — for example imposing a 2:4 or block-structured pattern — in order to use sparse
  hardware. Dense weights MAY be dispatched through a sparse kernel; the weights themselves must
  remain dense. See [§2.9.3](#293-model-weight-rules).
- **Static removal of weights from the checkpoint.** Dropping experts, layers, or any other weights
  that participate in the forward pass for some inputs — including experts that are rarely routed to
  — whether or not the resulting submission passes the accuracy gate. The distinction from
  [§2.9.6.3](#2963-canonical-architectural-sparsity) is static versus dynamic: canonical routing
  decides per token at run time; removing an expert from the checkpoint decides once, for every
  token. Components that do *not* participate in the forward pass, such as speculative-decoding
  heads, may likewise not be stripped from the derived checkpoint — see
  [§2.9.3](#293-model-weight-rules).
- **Softmax elision where the reference sampling configuration is stochastic** (temperature > 0,
  top-p, top-k), since sampling depends on the normalized probabilities.
- **Softmax elision where the response returns token logprobs or probabilities**, unless those
  values are computed exactly as the reference would.

##### 2.9.6.6 Disclosure

Techniques permitted under [§2.9.6.4](#2964-dynamic-approximate-sparsity) MUST be declared in the
submission YAML:

| Field | Requirement |
|---|---|
| Method and source | The technique and its paper or implementation reference. |
| Threshold parameter | The threshold or target-sparsity value (e.g. `λ`). |
| Calibration procedure | How the threshold was selected, and the data used to select it. |
| Per-point value | The value in force at each measurement point. |
| Schedule or distribution | Required where the threshold varies with context length or varies dynamically within a run. |

Techniques permitted under [§2.9.6.2](#2962-exact-sparse-execution-and-softmax-elision) and
[§2.9.6.3](#2963-canonical-architectural-sparsity) require no disclosure beyond the software-stack
listing of [§8.4](#84-software-disclosure).

#### 2.9.7 Post-Processing Equivalence

> [!NOTE]
> **[WIP — align with inference_rules.adoc closed division post-processing rules]**

- **Detokenization.** The response text must be produced by applying the reference detokenizer to
  the generated token IDs.
- **Stop token handling.** The generation must halt on the same stop tokens and EOS conditions
  defined in the benchmark specification.
- **Sampling.** For benchmarks using greedy decoding (temperature = 0), the submission must also use
  greedy decoding. For benchmarks specifying a sampling configuration, the submission must use the
  same sampling parameters as specified in the benchmark definition.
- **Output stream.** On streaming runs, the SUT must dispatch output to the client incrementally as
  it is generated. A multi-token **stream interval** is permitted: the SUT may group up to N
  consecutive generated tokens into one chunk (e.g., `stream_interval` in TensorRT-LLM, SGLang, and
  vLLM), dispatching each chunk once N tokens have accumulated or generation ends. Holding tokens
  beyond the configured interval, or buffering the complete response into a single message, is not
  permitted. This is a SUT requirement and is independent of the client-side `stream_all_chunks`
  setting (see [§6.5](#65-dataset-considerations)).

#### 2.9.8 Accuracy Gate

> [!NOTE]
> **[WIP — accuracy tolerance values to be specified per benchmark, aligned with
> inference_rules.adoc accuracy targets]**

A Standardized division submission meets model equivalence only if it meets the **accuracy quality
target** defined for the benchmark, evaluated using the reference evaluation methodology on the
accuracy dataset. Passing the accuracy gate is **necessary** for model equivalence, and it is the
arbiter of whether an approximation these rules permit — such as those in
[§2.9.6.4](#2964-dynamic-approximate-sparsity) — is acceptable in a given submission.

Passing the accuracy gate is **not a general override**. A submission that meets the quality target
while violating an operative rule of [§2.2.1](#221-general-rules) or §2.9.x — for example by pruning
weights ([§2.9.3](#293-model-weight-rules)), removing experts from the checkpoint
([§2.9.6.5](#2965-disallowed)), or fine-tuning a drafter ([§2.9.4](#294-speculative-decoding)) — is
not model equivalent.

The accuracy quality target and tolerance relative to the reference score are specified per
benchmark in the benchmark definition.

#### 2.9.9 Q&A: Model Equivalence Clarifications

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`** Q&A entries are interpretive guidance. If a
> Q&A entry conflicts with the operative rules in §2.2.1 or §2.9.x, the rules take precedence and
> the Q&A entry will be revised.

**Q1: Is response or query caching allowed?**
A: No. Returning a cached response verbatim to a request that matches a previous request is
prohibited. Every request must go through the forward pass. KV-cache reuse (within or across
queries) is a *serving optimization* governed by [§2.9.5](#295-kv-cache-rules), **not** response
caching — the distinction is that KV-cache reuse still executes the forward pass on per-query tokens
(which include a unique salt; see [§2.9.5.1](#2951-salting-mechanism)), whereas response caching
skips compute entirely.
The salt is itself part of the submission's **bound seed set**
([Submission Rules §4.6](endpoints_submission_rules.md#46-seed-rotation)), so it changes when
MLCommons refreshes the seed set every two cohorts.

**Q2: Is iteration coalescing — the server returning multiple generated tokens in a single network
message — allowed?**
A: Yes, as a multi-token stream interval under [§2.9.7](#297-post-processing-equivalence). Serving
frameworks expose this to reduce per-token dispatch overhead at large batch sizes. Changing chunk
boundaries is not a modification of the response stream under [§2.2.1](#221-general-rules), since
the concatenated output is unchanged. No metric adjustment is applied: TTFT is taken at the first
chunk, so a larger interval delays it, and TPOT is computed over the post-first-chunk output
tokenized once with the reference tokenizer (see [§2.8 Tokenizer Rules](#28-tokenizer-rules)).

**Q3: Can I use a different serving framework than the reference (vLLM vs. TensorRT-LLM vs. SGLang)?**
A: Yes. Arbitrary frameworks and runtimes are inherited from upstream, provided the framework
conforms to the rest of the rules (model equivalence, no benchmark detection, no input-based
optimization, etc.). The framework must satisfy the **Available** definition
([Submission Rules §7.2](endpoints_submission_rules.md#72-available)).

**Q4: How does cross-request KV cache sharing interact with the salt mechanism?**
A: See [§2.9.5 KV Cache Rules](#295-kv-cache-rules) and
[§2.9.5.1 Salting Mechanism](#2951-salting-mechanism). Cross-request KV sharing is **blanket
allowed** in Endpoints (this is the primary delta vs. upstream MLPerf Inference). The performance
dataset injects a per-query salt between the shared system prompt and the per-query user context, so
the only prefix two queries can share is the system prompt itself. Accuracy runs use the un-salted
dataset.

**Q5: Where did the previous Q&A entries on quantization, sparsity, and softmax elision go?**
A: They were promoted into the operative rules and are no longer restated here: PTQ and unused
checkpoint components are in [§2.9.3](#293-model-weight-rules); drafter PTQ is in
[§2.9.4](#294-speculative-decoding); pre-tokenizing clients and the salt are in
[§2.9.5.1](#2951-salting-mechanism); sparse execution, attention patterns, softmax elision, hardware
sparsity, and expert removal are all in [§2.9.6](#296-sparsity-and-approximate-computation).

**Q6: Is tree-structured verification attention permitted for speculative decoding?**
A: Yes, as an implementation detail of the exact-verification requirement in
[§2.9.4](#294-speculative-decoding) — not an exception to it. Any tree/DFlash-style verification
attention is fine provided the target-output-distribution guarantee still holds (token-for-token
identical to the unspeculated target).

**Q7: Does a drafter implemented as a truncated forward pass through a subset of the target model's own layers (self-speculative / early-exit, no separate weights) satisfy the open-weight requirement in [§2.9.4](#294-speculative-decoding)?**
A: Yes. There are no separate weights to disclose — the target model's own public checksum already
establishes this. It must still appear on the benchmark's approved drafter list, entered as a
**configuration-identified** drafter: the target checksum together with the exit-layer and any other
configuration that defines the draft pass, rather than a separate weight checksum. The approval
process and eligibility criteria of [§2.9.4](#294-speculative-decoding) apply to it unchanged.

---

## 3. Benchmarks and Models

### 3.1 Benchmark Definition

A benchmark in MLPerf Endpoints is defined by a specific model, task, and quality target. Each
benchmark has a reference implementation that defines the correct endpoint interface, input/output
format, and accuracy evaluation method.

### 3.2 Supported Models

The set of supported benchmark models is defined per submission round and maintained in the MLPerf
Endpoints reference repository. The full per-model specification — canonical weights, dataset, chat
template, server parameters, accuracy target, and (if applicable) drafter configuration — is given
by the [reference implementation](#291-reference-implementation). Each supported model is identified
by:

- A Hugging Face model ID (or equivalent checksummed source).
- The task category (text generation, summarization, reasoning, etc.) and the input/output modality
  (text, token IDs, streaming, non-streaming).

> [!NOTE]
> The model list for each submission round is published in the MLPerf Endpoints reference repository
> at least 6 weeks before the submission round opens. New models may be proposed to the working
> group per the benchmark roadmap process defined in the MLPerf General Submission Rules §4.3.

The list of benchmarks, models, and reference implementation for MLPerf Endpoints v1.0 is:

| Benchmark Type | Model Name | Reference Implementation |
|---|---|---|
| Legacy LLM | DeepSeek R1 | [Reference](https://github.com/mlcommons/endpoints/tree/main/examples/07_DeepSeekR1_Example) |
| Legacy LLM | GPT-OSS 120B | [Reference](https://github.com/mlcommons/endpoints/tree/main/examples/04_GPTOSS120B_Example) |
| Legacy LLM | Llama-3.1-8B-Instruct | [Reference](https://github.com/mlcommons/endpoints/tree/main/examples/05_Llama_Examples#llama-31-8b-instruct) |
| Agentic | Kimi K3 | [Reference](https://github.com/mlcommons/endpoints/tree/main/examples/10_Agentic_Inference#kimi-k3) |
| Agentic | DeepSeek-V4.1-Flash | [Reference](https://github.com/mlcommons/endpoints/tree/main/examples/10_Agentic_Inference#deepseek-v41-flash) |
| Agentic | Qwen3.6-35B-A3B | [Reference](https://github.com/mlcommons/endpoints/tree/main/examples/10_Agentic_Inference#qwen36-35b-a3b) |


**Approved drafter lists.** For benchmarks that support speculative decoding, the approved drafter
list is published in the reference repository alongside the model list, versioned per submission
round. The approval process, eligibility criteria, and the lead time required before a newly
approved drafter may be used are defined in [§2.9.4](#294-speculative-decoding). Native heads of the
listed checkpoints are approved with them and need not appear on the list
([§2.9.4](#294-speculative-decoding)).

### 3.3 Weight Transformations

Submitters may apply quantization, format conversion, or other weight transformations to the
reference weights, subject to the accuracy quality target. All transformations must be documented in
the submission. For the Standardized division, the full set of permitted and prohibited
transformations is defined in [§2.9.3 Model Weight Rules](#293-model-weight-rules).

---

## 4. Metrics

### 4.1 Primary Metrics

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`** TTFT framing — see note below the table on
> percentile selection.

Each measurement point on the pareto curve captures the following metrics at a specific concurrency
level:

| Metric | Symbol | Definition |
|---|---|---|
| System Tokens per Second | `system_tps` | Total output tokens produced per second across all concurrent users. `system_tps = total_output_tokens / elapsed_duration_seconds`. |
| TPS per User | `tps_per_user` | `tps_per_user = 1000 / tpot_p90_ms`, where `tpot_p90_ms` is the P90 of valid per-response TPOT samples. Higher is better. |
| E2E Average Interactivity | `e2e_avg_interactivity` | For agentic benchmarks, the output-token rate across completed turns, reported as a single scalar per measurement point. The scalar aggregates all completed turns across all trajectories in the run: `e2e_avg_interactivity = sum(output_tokens_per_turn) / sum(e2e_turn_time_seconds)`, where `e2e_turn_time_seconds` is the server-side time from receipt of the request through completion of the response for each turn and excludes tool-call execution time. |
| Time to First Token (P90) | `ttft_p90_ms` | 90th-percentile time, in milliseconds, from query issuance until the client receives the first non-empty text fragment (`len(s) > 0`) in any response category (visible-output, tool-call, or reasoning). |
| Concurrency | `concurrency` | The target number of in-flight concurrent queries for this measurement point. |

> [!NOTE]
> **Genuine first token.** TTFT is triggered by the first non-empty fragment (`len(s) > 0`).
> Emitting whitespace, control characters, punctuation, or other meaningless leading content solely
> to stop the TTFT clock — rather than as a genuine part of the model response — is not allowed.

> [!NOTE]
> **TTFT does not apply to the Offline point.** Every query is queued at the start of an Offline
> run, so TTFT is unbounded in principle and carries no information. `ttft_p90_ms` is not required
> for the Offline point — see [§5.7](#57-offline-point).

> [!NOTE]
> **TTFT versioning.** The historical v0.7 rules used **P95** for the publication plot and as the
> primary TTFT metric. These v1.0 rules use **P90**; only `ttft_p90_ms` is required to be reported
> per measurement point. Additional TTFT percentiles (e.g., P50, P99) may be reported in a
> submission YAML, but they are not plotted in the v1.0 publication chart.

### 4.2 Derived and Presentation Metrics

> [!CAUTION]
> **`[TENTATIVE — Subject to change after 2026-10-12]`**

The following metrics are derived from primary measurements and used in publication charts. All
charts use the percentile metric defined in [§4.1](#41-primary-metrics):

| Metric | Description |
|---|---|
| **Pareto curve (System TPS vs. TPS/User)** | The primary publication chart for single-turn benchmarks. **Y-axis:** `system_tps`. **X-axis:** `tps_per_user`. Each point corresponds to a different concurrency level. Represents the fundamental tradeoff between aggregate system capacity and per-user experience. |
| **Agentic Pareto curve (System TPS vs. E2E Average Interactivity)** | The primary publication chart for agentic benchmarks. **Y-axis:** `system_tps`. **X-axis:** `e2e_avg_interactivity`. Each point corresponds to a different concurrency level. Higher values are better on both axes. |
| **System TPS vs. Concurrency** | **Y-axis:** `system_tps`. **X-axis:** `concurrency`. Shows aggregate throughput scaling with load. Each point annotated with its region. |
| **TTFT (P90) vs. Concurrency** | **Y-axis:** `ttft_p90_ms`. **X-axis:** `concurrency`. Shows how first-token latency degrades with load. P90 is the default and the only percentile plotted for v1.0; additional percentiles are deferred to a later version (see [§4.1](#41-primary-metrics)). |
| **Interactivity vs. Concurrency** | **Y-axis:** `tps_per_user`. **X-axis:** `concurrency`. Shows how per-user output rate degrades with load. |

### 4.3 Accuracy Metric

Each benchmark defines a quality target expressed as a minimum acceptable score on the benchmark's
accuracy metric (e.g., ROUGE score, exact match, perplexity). The accuracy metric and quality target
are specified in the benchmark definition.

An **accuracy validation run** evaluates the benchmark's accuracy dataset; for the agentic
benchmarks, this is the SWE-bench evaluation. **Inline accuracy** is scored on the performance
run itself. It is a separate per-point requirement and is not an accuracy validation run, even
when both results are stored in the same `accuracy_results.json` file.

**Exactly one** accuracy validation run is required, and **exactly one may be submitted**, for each
of the mandatory points defined in [§5.3](#53-minimum-submission-requirements) — one in the Ultra
Low Concurrency region, one in each of the Low, Medium, and High Concurrency regions, and, for
non-agentic benchmarks, one at the Offline point. Accuracy and performance runs MUST use the same
server endpoint configuration, model weights, and software stack.

| Benchmark type | Required accuracy validation runs (`N`) | Composition |
|---|---|---|
| Non-agentic | **5** | 4 concurrency regions + Offline |
| Agentic | **4** | One SWE-bench evaluation per concurrency region; the Offline point does not apply ([§5.7](#57-offline-point)) |

`N` is a fixed count, not a floor. A submission carrying results from more than one accuracy
validation run for a region, or from any accuracy validation run at a submitter's-choice point, is
non-conforming and is rejected at automated compliance ([§9.1](#91-automated-checks)). Inline
accuracy scores are excluded from this count; an inline-only result neither fills a mandatory
accuracy point nor counts as an extra accuracy validation run.

*Rationale:* the multi-turn criterion below is a **mean** over the `N` results. If the number of
accuracy validation runs were at the submitter's discretion, that mean could be shifted by adding
runs at favourable operating points, and the threshold would no longer mean the same thing across
submissions. Fixing `N` — and fixing *which* points it covers — keeps the accuracy criterion
comparable. The three submitter's-choice performance points characterize the curve; they carry no
accuracy validation run.

For both single-turn and multi-turn benchmarks, accuracy validation runs are required at exactly
those `N` points. Their results are gated as follows:

- **Single-turn (per-point):** Each of the `N` required accuracy validation results MUST meet the quality
  threshold. Each accuracy run MUST use matching concurrency on the same instance, immediately after
  the corresponding performance run.
- **Multi-turn (mean-of-N):** The arithmetic mean of the `N` required accuracy validation results
  MUST meet the quality threshold; individual results need not. Accuracy concurrency may differ
  because multi-turn accuracy runs are time- and resource-intensive.

For agentic benchmarks, the mean-of-`N` gate applies to the four SWE-bench validation results.
Inline scores MUST NOT be included in that mean. Every submitted agentic performance point,
including all submitter's-choice points, MUST report an inline accuracy score from that point's
performance run and individually meet the model-specific inline threshold defined by the
[reference implementation](#291-reference-implementation). A missing, null, or below-threshold
inline score is non-conforming. Scores at other points cannot compensate for a failing point.

Without `c_min_override`, the Ultra Low accuracy validation result MUST come from the mandatory
point at the derived $C_{min}$. With an override, the submitter may designate any submitted point
in $[1, C_{min}]$ as the mandatory Ultra Low point and submit its accuracy validation result.
Other points in that region are submitter's-choice points and carry no accuracy validation run.
No result may fill two concurrency regions; use the same designated results for coverage and the
multi-turn mean. The elected Offline exception (§5.7.2) remains applicable.

### 4.4 Reporting Basis (Steady-State Window)

Until v0.7, a point's metrics ([§4.1](#41-primary-metrics)) were averaged over the whole
post-`TEST_STARTED` run, which still included the load-dependent **ramp-up** (inflates the TTFT
tail) and the **drain tail** (deflates throughput) that warmup period ([§6.3](#63-warmup-period))
did not remove. For 1.0 and beyond, the **official result is instead computed over the detected
steady-state window**, with the whole-run (`total`) metrics kept as supplementary. The window is
defined on **issue time** — excluding the drain from the throughput denominator with no end-crop —
and the residual ramp is cropped from the data on top of the declared warmup. Detection is a
post-processing step over the durable event log (`events.jsonl`, [§4.1](#41-primary-metrics)), off
the measured path; methodology and default parameters:
[`scripts/steady_state_diagnostics.md`](https://github.com/mlcommons/endpoints/blob/3a51022c2f52dea27fc0338b91df781c3871f538/scripts/steady_state_diagnostics.md).

**Definitions.** The terms used throughout this section:

| Term | Definition |
|---|---|
| **Super-pass** | A contiguous issue-order block of queries sized to one full pass over the dataset, unless the [benchmark definition](#31-benchmark-definition) specifies a different super-pass size. Window length and the trend-test floor are measured in super-passes. |
| **Ramp-up** | The load-dependent transient at the start of a run — after the declared warmup ([§6.3](#63-warmup-period)) — while in-flight concurrency and queue depth are still climbing to target; it inflates the TTFT tail. The residual ramp is cropped from the front of the data before detection. |
| **Drain (drain tail)** | The transient at the end of a run during which no new queries are issued and in-flight queries complete; it deflates throughput. Excluded by defining the window on issue time (no end-crop required). |
| **Gating metric** | The metrics whose stability decides whether a steady state holds: TPOT at P50 and P90. |
| **Plateau** | A gating-metric state showing no significant trend across the super-passes (per the trend-test in [`steady_state_diagnostics.md`](https://github.com/mlcommons/endpoints/blob/3a51022c2f52dea27fc0338b91df781c3871f538/scripts/steady_state_diagnostics.md)) — i.e., stable. |
| **Drifting Up / Drifting Down** | A gating-metric state showing a significant increasing / decreasing trend across the super-passes; reported as drift (range/slope), never as a point estimate. |
| **Trend-test / `MIN_TREND_N`** | The per-metric trend test applied across super-passes; `MIN_TREND_N = 4` is its minimum-sample floor (≥ 4 super-passes). Exact test and parameters: [`steady_state_diagnostics.md`](https://github.com/mlcommons/endpoints/blob/3a51022c2f52dea27fc0338b91df781c3871f538/scripts/steady_state_diagnostics.md). |
| **Change-point** | A confirmed step between two materially different, internally stable plateaus within one run; triggers the `anomaly` (staircase) verdict, where the first plateau is the reported steady state and the later shift is disclosed as likely degradation. |
| **Steady-state window** | The contiguous issue-time interval — after warmup and residual-ramp crop, before the drain — over which the gating metrics are stable (Plateau). The official result is computed over this window when the steady-state condition holds. |
| **`total` (whole-run) metrics** | Metrics averaged over the entire post-`TEST_STARTED` run (the pre-1.0 basis). Reported as supplementary alongside the steady-state result, and the official fallback where no steady state holds. |
| **Coverage `status`** | Sample-count and duration classification of a point — `windowable`, `insufficient_duration`, `insufficient_passes`, `partial_dataset` (see the status table below). |
| **Detected shape / verdict** | The detector's per-run classification of gating-metric behavior — `STEADY STATE`, `drifting_up`, `drifting_down`, `anomaly`, `not found` (see the shape table below). |

Steady-state is the official result **only where the condition holds**: the steady window spans **≥
4 super-passes** (the trend-test floor `MIN_TREND_N = 4`, so a run needs more than 4 super-passes
total), every gating metric — TTFT and TPOT at P50/P90 — is a **Plateau**, not **Drifting Up**,
**and** the window's **issue-time span meets the [§6.2](#62-minimum-run-duration) minimum run
duration** for the point's concurrency region. The effective floor is therefore
`max(4 super-passes, §6.2 minimum duration)` — at high concurrency the duration floor binds, since 4
super-passes can complete in well under the minimum. A *super-pass* is a contiguous issue-order
block sized to one full-dataset mix — by default one full dataset pass, unless the
[benchmark definition](#31-benchmark-definition) specifies a different super-pass size. Otherwise
the point falls back by coverage `status`:

| `status` | Condition | Official result |
|---|---|---|
| `windowable` | ≥ 4 super-pass steady window in Plateau **and** window issue-time span ≥ [§6.2](#62-minimum-run-duration) minimum | steady-state metrics; `total` supplementary |
| `insufficient_duration` | ≥ 4 super-passes in Plateau but window issue-time span < [§6.2](#62-minimum-run-duration) minimum | `total` (steady-state reported low-confidence, not official) |
| `insufficient_passes` | ≥ 1 super-pass but window < 4 super-passes | `total` (steady-state reported low-confidence, not official) |
| `partial_dataset` | < 1 super-pass | `total` only (no steady-state claim) |

Beyond the coverage `status` above (which gates on sample count), a point's official result depends
on the **shape** the detector finds over the super-passes. The detector
([`steady_state_diagnostics`](https://github.com/mlcommons/endpoints/blob/3a51022c2f52dea27fc0338b91df781c3871f538/scripts/steady_state_diagnostics.md))
emits one verdict per run:

| Detected shape | Verdict | What is reported | Accepted as steady-state? |
|---|---|---|---|
| All gated metrics stable across the window | `STEADY STATE` | Steady-state metrics over the window; `total` supplementary | ✅ Yes — the official result |
| A gated metric keeps **climbing** over the super-passes after the window | `drifting_up` | That metric reported as **drift** (range/slope), never a point estimate; window flagged *local-plateau only* | ⚠️ Reported-with-flags — global steady state questionable |
| A gated metric trends **down** over the tail | `drifting_down` | Reported as drift, not a point estimate | ⚠️ Reported-with-flags |
| First plateau steps to a later, materially different plateau (change-point confirmed) | `anomaly` (staircase) | **First** plateau reported as the steady state; later shift flagged as `anomaly` (likely degradation) | ✅ Yes (first plateau); anomaly disclosed |
| No contiguous run of super-passes is steady enough (drifts throughout, or too short) | `not found` | No steady-state claim; falls back to whole-run `total` | ❌ No — `total` only |

**Scope.** Only `ConcurrencyScheduler` points ([§6.1](#61-load-pattern)) are in scope;
`MaxThroughput`/`Poisson` and single-pass agentic workloads are handled only by the ad-hoc
diagnostic tool. The minimum run duration ([§6.2](#62-minimum-run-duration)) is measured over the
steady window's **issue-time span**, not wall-clock; a window shorter than the §6.2 minimum for its
concurrency region is `insufficient_duration` and falls back to `total`.

**Pending ratification.** Whether the super-pass floor is raised above 4, and whether a `not found`
run is declared *invalid* versus *reported-with-flags* (the ⚠️/❌ rows assume the latter).

---

### 4.5 Performance Normalization

> [!CAUTION]
> **`[TENTATIVE — Pending working-group ratification]`** This section introduces power-based
> normalization for Endpoints v1.0. Tier definitions, overhead fractions, component references, and
> the name and units of the reported normalized metric are subject to change.

MLPerf Endpoints normalizes total system throughput by **provisioned power**, so that systems of
different scale can be compared on a common basis. Normalization is what makes results *comparable*:
without it a larger system trivially out-performs a smaller one, and a buyer cannot tell which
delivers more for a given deployment budget.

**Scope.** Power normalization applies to **all Standardized division submissions, in both the
Client on Prem (CoP) and Client over Network (CoN) scenarios**
([§2.1](#21-client-deployment-scenarios)). Normalization options for the **Serviced** division —
managed endpoints, CSP-hosted services, and similar offerings, where provisioned power is not a
property the submitter controls or discloses — will be introduced in a later version. RDI
submissions MAY report normalized throughput but are not required to.

#### 4.5.1 Power Normalization Roadmap and Rationale

**Goal.** MLPerf Endpoints will transition to mandatory provisioned-power-based normalization of
performance (total system throughput) in Endpoints v1.0 and beyond. Toward this goal the benchmark
adopts a phased approach of increasing provisioned-power fidelity, and will eventually add true
measured power as an additional normalization option.

**Why provisioned power.** Provisioned power is selected as the normalizing factor because:

- It is a good proxy for **total cost of ownership** (cost of acquisition + cost of operation). In
  practice a buyer computes the cost of a system and the cost of operating it; this is a rough proxy
  for that.
- It correlates with the **capital cost of power delivery** for deploying a system into a rack or
  data center — UPS, PDUs, generators, and similar.
- It correlates with **measured power in well-utilized data centers**. Operators generally optimize
  to keep utilization high, which implies measured power tracks provisioned power.
- Data center operators are typically **capacity limited by provisioned power** rather than by floor
  space.
- CSPs, neoclouds, and others have given feedback that they evaluated provisioned power and find it
  **more useful than power consumption**.
- The prior **measured-power approach saw very limited uptake**, because it required additional
  power meters and extra test time against tight submission deadlines.
- Modern systems have **configurable power capping**, which lets OEMs and buyers limit consumption —
  and lets provisioned power be sized correctly for partially populated systems.
- Provisioned power is **more feasible to obtain or estimate for systems that have not been
  submitted** to MLPerf, which matters for comprehensive testing.

**Normalization factor by system type.** Provisioned power is the factor for systems whose power a
submitter provisions and can evidence. Other deployment types are expected to normalize differently,
and are deferred to a later version:

| System type | Normalizing factor | Status |
|---|---|---|
| **On-prem** | Provisioned power | In force for v1.0 |
| **IaaS** (rented instances) | Provisioned power, optionally price | Deferred. Pricing is trivially observable by a buyer, but the terms must be pinned — for example 1-year reserved-instance rates. |
| **IaaS-C** (committed capacity) | Provisioned power, optionally price | Deferred |
| **Managed endpoint** | Cost per token | Deferred |

**Phased tiers.** Provisioned power definition, calculation, and methodology advance through three
tiers of increasing fidelity, accuracy, and quality. Endpoints begins at Tier 3 and works upward.

| Tier | Definition | Status |
|---|---|---|
| **Tier 1** | Highest fidelity. Not yet defined; the roadmap anticipates true measured power as an additional normalization option. | Future |
| **Tier 2** | The **nameplate power** of the system's power supplies, accounting for any software-managed power capping. Requires robust verification of system- and rack-level power provisioning. Any power capping must be validated by an MLCommons-defined methodology, which may include Redfish-based logging, independent third-party audit of datacenter deployments, or publicly available documentation of the system's deployment specifications published by the submitting organization. | Target. Requires a dedicated effort to define verification criteria and methodology before it can be rolled out. |
| **Tier 3** | The **sum of the rated power of the key power-consuming components**, plus an assumed margin. Where publicly available and verifiable data is absent, MLCommons may substitute a proxy value for a component based on public data and analysis. | **In force for Endpoints v1.0** ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)) |

**What provisioned power must account for.** The goal is to capture the key power-consuming elements
and the reasonable margins and buffers that vendors, OEMs, ODMs, and customers would themselves
employ. Key components include computing elements (CPU, GPU, ASIC), switching, storage, networking,
cooling, and any power-correction units.

- For **remote-hosted storage**, the power of dedicated storage racks need not be included.
- For **DC-level liquid cooling**, submitters may provide the power of the entire data center and
  scale it to the submitted system's size. For an individually hosted rack or system with a
  dedicated CDU, cooling power MUST be included.

**Why Tier 3 first.** The component-sum definition is less precise than nameplate power, but it
works well for comprehensive testing — where the claimed or rated power of the
performance-determining devices (CPU, GPU, ASIC) is the information most readily found. It
accommodates systems that are partially filled or racks that are not fully used, and it puts every
submitter on the same methodology.

#### 4.5.2 Proposed Endpoints v1.0 Normalization Methodology

The v1.0 approach normalizes by power using three elements:

1. An **MLC-approved, simplified and consistent method** for calculating the power of a system from
   the TDP or power consumption of its most significant components.
2. A mechanism and guidelines for **submitters to provide the inputs** in a verifiable manner. This
   is the preferred path.
3. A mechanism and guidelines for a **third party to estimate the inputs**, as a conservative
   default or fallback path.

##### Methodology

- MLCommons provides a defined template for total system power, summing critical component power and
  adding margins for cooling and PSU overheads.
- Submitters are **encouraged to provide accurate and publicly verifiable** details for component or
  system power. In the absence of publicly verifiable sources provided by the submitter, MLCommons
  will use conservative estimations.
- If a submitter is not satisfied with an MLCommons power estimate, they must either point to better
  verified sources or disclose component power directly and publicly, thereby creating a verified
  and public source. In the abscence of public and verifiable information, MLCommons may not accept
  the submitter's recommendations.
- The MLC default template uses publicly available sources for the TDP/TGP of each component. Where
  vendor documentation is absent, MLCommons relies on industry and academic sources.
- Estimation is done conservatively, from components with similar specifications or via an
  energy-per-unit calculation. For example, for a custom CPU SKU whose TDP is not publicly listed,
  MLCommons will use CPUs of similar architecture, core count, and memory configuration.

**Verifiable and unverified sources.**

| Category | Examples |
|---|---|
| **Verifiable** (preferred) | A spec sheet on the vendor's website; disclosures in an academic or technical conference or publication; statements made to press or media during a keynote or earnings call; other public statements officially sanctioned by the submitting organization. |
| **Unverified** | Any source not officially stated by a representative of the submitting organization — media speculation, third-party social media posts, industry analyst blogs, videos, and reports. |

**Running below rated TDP.** Any component running below its rated TDP, as stated in publicly
verifiable documentation, MUST be accompanied by evidence of the lowered TDP. That evidence must be
reproducible by a third-party audit, or the reduced mode must be publicly listed as an alternative
production or operational mode. The burden of evidence for custom and low-volume SKUs is identical
to that for any other component.

##### Power Model

```
System Power   = Major_components + Other_components + Scale_out_switch_power

Major_components = CPU_power
                 + Accelerator_power
                 + Network_scale_up_power
                 + Optional_scale_out_NIC_power

Other_components = overhead_fraction × Major_components

overhead_fraction = 0.30   liquid-cooled systems
                  = 0.50   air-cooled systems
```

- **`CPU_power`** — power required for the CPUs in the system (e.g. Intel Xeon processors, Axion
  CPUs alongside a Google TPU), calculated as `number of CPUs × TDP`. Where the TDP is not
  disclosed, public sources may be used to estimate it.
- **`Accelerator_power`** — power required for the accelerators (e.g. AMD MI355X, Google TPU),
  calculated as `number of accelerators × TDP`. Where the TDP is not disclosed, public sources may
  be used to estimate it.
- In some systems CPU and accelerator power are published as a **single combined value**. That is a
  valid alternative formulation.
- **`Network_scale_up_power`** — power for the high-bandwidth network connecting the accelerators,
  such as NVIDIA NVLink, the TPU Inter-Chip Interconnect, or UALink over Ethernet. The scale-up
  network accounts for the majority of networking power. Calculated as
  `number of switches × TDP per switch`; where switch power is not disclosed it may be estimated as
  `total switch bandwidth × energy per bit`. In systems with no switches this term is zero.
- **Scale-out fabric** — where the submission spans more than one node, the fabric that connects
  them is counted in two parts, which enter the model differently. **Both are zero for a single-node
  submission, and zero where the nodes are joined only by a fabric already counted in
  `Network_scale_up_power`.**
  - **`Optional_scale_out_NIC_power`** is a **major component**, and so takes the overhead fraction.
    A NIC is a card inside the node: its rated TDP is silicon draw, and it needs the same cooling
    and power-supply headroom as any other component in the chassis.
  - **`Scale_out_switch_power`** is added to the total **outside the overhead base**. A rack switch
    figure — whether the submitter's own or a [D.4](#d4-reference-scale-out-network) reference — is
    already **wall power**, including the switch's own power supply, so applying the overhead
    fraction to it would double-count.
  - **Whether NIC power is counted depends on how node power was obtained.** Where node or rack
    power is **built with the MLC formula**, the formula has no NIC term of its own, so the
    scale-out NICs' rated TDP **MUST be included** here for a multi-node submission. Where node or
    rack power is taken from a **published specification**, the NICs are assumed to be part of that
    figure and MUST NOT be added again. A submitter who believes a published figure excluded the
    adapters must say so and evidence it. The reference value for a scale-out NIC is in
    [Appendix D.4](#d4-reference-scale-out-network).
  - **Rack switch power** is counted in full, and depends on cabling. **The submitter MUST declare
    whether the fabric uses passive or active (optical) cabling**, because an active-optical
    deployment can draw more than twice the power of the same switch with passive cables.
  - **Switch count follows from the bandwidth the fabric must carry**: the sum of per-node NIC
    bandwidth across the submitted nodes. A ten-node cluster of DGX B300s, each with eight 800 Gb/s
    NICs, needs `10 × 8 × 800 Gb/s = 64 Tb/s`. The submitter either declares the rack switches
    actually used, or selects one or more reference switches from
    [Appendix D.4](#d4-reference-scale-out-network) whose combined bandwidth meets the requirement.
- **`Other_components`** — storage, power-supply overhead, and cooling. Individually these may not
  be substantial; in aggregate they are significant and must be accounted for. They are estimated as
  a fixed fraction of the major-component power, set by cooling method. Scale-out **NICs** are
  inside that base as major components; scale-out **switches** are not, and must not be counted
  twice.

**Composing a multi-node submission from published node power.** Where a node's power is taken from
a published *system* figure, that figure already includes that node's own `Other_components` — its
cooling, power-supply overhead and storage. The overhead fraction MUST NOT be applied to it a second
time. In that case:

```
System Power = Σ published node power + Scale_out_switch_power
```

The overhead fraction is applied to **neither** term. The published node figure already carries the
node's own overhead — including its NICs, which is why they are not counted separately on this path
— and published switch power is wall power, an ATIS typical figure already including the switch's
power-supply draw. Where node power is instead built from components, the ordinary formula applies
to the whole system at once.

**Units and rounding.** Component power is declared in **watts**. `provisioned_power_kw` is the
total system power in kilowatts — total watts ÷ 1000 — rounded to **two decimal places**, and
`system_tps_per_kw` is reported to **one decimal place**. Rounding is applied once, to the final
figures; intermediate sums are not rounded.

> [!NOTE]
> Worked examples of the complete calculation — air-cooled and liquid-cooled, full and partial
> systems, rack-level node scaling, and the declared-provisioned-power override — are in
> [Appendix C](#appendix-c-worked-power-normalization-examples).

##### Component Template (`system_power.json`)

The template below is codified into a `system_power.json` descriptor file accompanying the
submission. Its field-by-field definition, validation rules, and worked descriptors are in
[Appendix E](#appendix-e-system_powerjson-field-reference).

> [!IMPORTANT]
> **`system_power.json` is mandatory.** Every submission MUST include a `system_power.json`
> descriptor for **each system**, conforming to the template below and located per
> [§8.1](#81-directory-structure). A submission without it is incomplete and is rejected at
> automated compliance ([§9.1](#91-automated-checks)). A submitter who does not supply a value for a
> given field leaves it to be auto-populated by the MLCommons checker, which triggers the
> estimated-power tag described below — but the file itself is required either way.

The MLCommons checker auto-populates power values where a submitter does not provide them or where
public information is lacking.

**Citation requirement.** Every value in `system_power.json` MUST carry a reference to a public,
verifiable source meeting the criteria above — **unless** it is an MLCommons default reference value
from [Appendix D](#appendix-d-mlcommons-default-power-reference-values), which is its own reference
and needs no further citation. **Self-declaration is not sufficient.** A figure asserted by a
submitter without a public source is treated as no value at all: the MLCommons default applies, and
the result is tagged "MLC Estimated Power".

A submitter's own verifiable public reference **overrides** the corresponding MLCommons default. The
defaults exist so that a submission is always computable, not to cap what a submitter may evidence.

| Field group | Fields | Fallback when public information is absent |
|---|---|---|
| **CPU** | `num_cpu`, `tdp_per_cpu`, link to public specification | MLCommons uses the architecture (x86 / ARM), core count, process, and memory channels declared in the system description to select the closest proxy. Default TDP values by architecture and core count are in [Appendix D.2](#d2-processors). A submitter may propose a proxy, but MLCommons may substitute a different one if it deems the proposal insufficient. |
| **Accelerator** | `num_accelerator`, `tdp_per_accelerator`, link to public specification; where run below rated spec, public evidence of the alternative SKU/TDP rating plus verifiable instructions and evidence of the reduced power (e.g. `rocm-smi` / `nvidia-smi` output) | Default TDP values for common accelerators are in [Appendix D.3](#d3-accelerators); beyond those, MLCommons relies on industry analysis and insights to estimate accelerator power for GPUs, ASICs, and similar. Under comprehensive testing the working group can guide the selection of appropriate values, and the target of the testing may volunteer better information provided it meets the public-and-verifiable requirement. Non-public information supplied by the submitter may be taken into consideration at the discretion of MLCommons or the working group. |
| **Scale-up network** (intra-node and rack-level) | `num_switches`, `tdp_per_switch` | MLCommons uses the link protocol (NVLink, Ethernet, PCIe), bandwidth per switch, and pJ/bit, drawing on publicly available information or industry analysis. Default reference points for NVLink- and Ethernet-based fabrics are in [Appendix D.1](#d1-scale-up-network). |
| **Scale-out network** (optional) | `num_nics`, `tdp_per_nic`, `num_switches`, `tdp_per_switch`, and the declared cabling (passive / active optical) | Used only for multi-node submissions. NICs are a major component and take the overhead fraction; switch power is wall power and is added outside it ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). Defaults in [D.4](#d4-reference-scale-out-network). |
| **Other components and Cooling** | auto-calculated | `overhead_fraction × (CPU + Accelerator + Scale-up + Scale-out NICs)`, with cooling estimated as a fraction of total power: **30%** for liquid-cooled, **50%** for air-cooled systems. Scale-out **switches** are excluded from this base. |
| **Total system power** | auto-calculated; used for normalization | `Major_components + Other_components`. |

**A published power figure is specific to the vendor and SKU that published it.** A rack or system
power rating applies only to that vendor's implementation of that configuration. Two vendors
shipping the same accelerator in the same rack topology may provision differently — power supplies,
cooling, redundancy, and integration all differ — so one vendor's published rating MUST NOT be used
by another. Each submitter cites a figure published for their own system.

**Declaring provisioned power directly.** If the estimated total system power is higher than a
submitter believes their system is rated at, they may instead provide a provisioned power number
directly. That number is subject to the same verification and publication requirements as every
other component. Where a published specification states a range for rack-level power, the **upper
bound** is used — for example, a system rated at 132–140 kW is taken as 140 kW.

These estimates are deliberately conservative. Submitters are encouraged to be as transparent as
possible in order to obtain a more accurate power figure.

**Obligation to report a materially wrong estimate.** A submitter who finds that the MLCommons
formula or a default reference value diverges **materially** from their system's actual provisioned
power MUST report the divergence to MLCommons, **in either direction**, so that the formula or the
default can be corrected. The estimates exist to make every submission computable, not to stand in
for a figure the submitter knows to be wrong.

Accepting an estimate known to **understate** actual provisioned power — and so to flatter the
normalized result — is a misrepresentation under
[Submission Rules §8.4](endpoints_submission_rules.md#84-issues-discovered-after-publication).
Reporting an estimate that **overstates** is equally required: an uncorrected overestimate penalizes
the submitter and leaves the formula wrong for everyone using it next round.

**Partially provisioned systems.** Where a system is only partially populated — a rack with sleds
unfilled, or a node with accelerator slots empty — provisioned power is established in one of three
ways:

1. **Verified, publicly available documentation** stating the power of the system as provisioned; or
2. **Rack-level node scaling** from published rack power, where the partial provisioning is a whole
   number of nodes ([§4.5.2.1](#4521-rack-level-node-scaling)); or
3. **The MLC formula above**, applied with the component counts limited to what is actually
   provisioned. `num_cpu` and `num_accelerator` reflect the populated configuration, not the maximum
   the chassis or rack could hold.

This describes how the system is *provisioned*, not how heavily it is *used* during a run: `num_cpu`
and `num_accelerator` reflect what is installed, and the resulting provisioned power is the same for
every measurement point. How much of that provisioned system a given point *engages* is a separate
question, answered per point by the node scaling of [§4.5.3](#453-normalized-metric).

##### 4.5.2.1 Rack-Level Node Scaling

A submitter who has published the power of a rack-scale system may scale that published figure down
to a partial rack, rather than rebuilding the number from components. For a rack of `N` nodes with
published total power `P_rack`, submitting `Y` nodes where `Y < N`:

```
provisioned_power(Y nodes) = P_rack × (Y / N)
```

`P_rack` is subject to the same verification and publication requirements as every other power value
in this section, and where the published specification states a range, the upper bound is used.

*Rationale:* at rack level, power scales linearly with the number of nodes — the per-node
contribution of compute, scale-up switching, cooling, and power-delivery overhead is essentially
constant across otherwise identical nodes. This path exists so that submitters who have already been
transparent about rack power are not forced back onto component estimation when they submit a
smaller configuration.

> [!NOTE]
> **This formula is applied twice, for two different purposes.** Here it establishes the
> **provisioned power of the submitted system** — how much hardware the submission consists of,
> fixed for the whole submission. In [§4.5.3](#453-normalized-metric) the same `× (Y / N)` scaling
> is applied again, **per measurement point**, to the nodes a given point engages. The linearity
> argument above is what licenses both. Keep the two distinct: `N` and the provisioned `Y` describe
> the system; the per-point `Y_s` describes one point on its curve.

**`Y` is a whole number of nodes, rounded up.** Any fractional node counts as a whole one. A
submitter using 4.5 nodes — because a node is shared or virtualized — declares `Y = 5`. Provisioned
power is a property of hardware that must be provisioned in whole units; half a node still requires
a whole node to be powered.

**The `N` nodes must be identical to each other and to the `Y` being submitted.** Scaling divides a
published figure by a node count, which is only valid where every node in the set draws the same
power. In particular:

- A **partially populated node is not identical** to a fully populated one
  ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). A set containing both cannot be
  scaled as one set.
- A node under a TDP cap is not identical to an uncapped node of the same model.

**Heterogeneous racks are partitioned into homogeneous sets.** Where a rack holds more than one kind
of node — a mix of H200 and B200 DGX nodes, say — the rack is divided into sets of identical nodes,
each set scaled from its own published figure or computed by the formula, and the results summed:

```
provisioned_power = Σ over homogeneous sets:  P_set × (Y_set / N_set)
```

A published *whole-rack* figure cannot be scaled directly in that case, because it does not
decompose into per-node shares that are equal across the rack.

**This path applies only at node granularity.** It MUST NOT be used for partial provisioning
*within* a node. Node power is a function of the accelerator count plus a substantial fixed
component — host CPU, memory, NICs, chassis, and power-supply overhead — that does not scale down
with accelerator count, so linear scaling materially understates the power of a partly populated
node.

| Configuration | Path |
|---|---|
| `Y` of `N` whole nodes, all identical | Rack-level node scaling above (`Y` rounded up to a whole node), or published power for that configuration |
| Partially populated node (some accelerator slots empty) | Published power for that specific configuration, or the MLC formula with component counts limited to what is populated. Not identical to a fully populated node, so it forms its own set |
| Node or rack running under a TDP cap | Published power for that specific configuration, or the MLC formula using the capped values, with the evidence required for running below rated TDP. Not identical to an uncapped node, so it forms its own set |
| Rack of mixed node types | Partition into homogeneous sets, scale each separately, and sum |

Vendor documentation describing the power provisioning of specific rack configurations is the
preferred source for `P_rack` and `N`. For example, NVIDIA publishes per-configuration power-domain
guidance for GB200 / GB300 NVL72 racks in its
[Mission Control systems administration guide](https://docs.nvidia.com/mission-control/docs/systems-administration-guide/2.3.1/prs/faq.html#example-1-configuring-a-pd-for-a-gb200-gb300-nvl72-rack).

**Estimated-power labelling.** Where power values are not provided by the submitter, or where the
result arises from comprehensive testing, MLCommons populates the missing values via the fallback
paths above and the published result is tagged **"MLC Estimated Power"**.

#### 4.5.3 Normalized Metric

| Metric | Symbol | Definition |
|---|---|---|
| Total System Throughput per Kilowatt | `system_tps_per_kw` | `system_tps_per_kw = system_tps / point_power_kw`, evaluated **per measurement point**. |
| Point power | `point_power_kw` | The normalization denominator for a single measurement point: the provisioned power of [§4.5.2](#452-proposed-endpoints-v10-normalization-methodology) scaled to the nodes used at that point, per the rule below. |

**Normalization is per measurement point.** A point that engages only part of a multi-node system is
normalized by the power of the part it engages, not by the whole system.

**Submitters MUST maximize the provisioned hardware engaged at each point.** A point may not be
served from a narrow slice of a larger provisioned system while the remainder sits idle. Where the
serving configuration admits replication, the submitter MUST replicate it **data-parallel** until no
further replica fits:

```
A_replica          = accelerators required by one replica   (TP × PP × EP at that point)
DP                 = floor( A_provisioned / A_replica )
accelerators_used  = DP × A_replica
```

A point whose configuration requires **24** accelerators per replica, on a system provisioning
**64**, MUST be run at `DP = floor(64 / 24) = 2`, engaging **48** accelerators. Running that point
at `DP = 1` on 24 accelerators, and normalizing by 24 accelerators' worth of power, is not a valid
submission of that point.

**The remainder is stranded, not charged.** In that example the leftover 16 accelerators cannot form
a third replica. They are excluded from the denominator by the node scaling below, and the point
divides by the power of the nodes holding the 48. The two rules are complements: maximal engagement
stops a submitter *choosing* to under-deploy, and per-point normalization stops them being charged
for capacity that genuinely cannot be used.

*Rationale:* a submission's pareto curve characterizes the system the submitter provisioned. Without
this rule, per-point normalization would let a submitter report every point from whatever small
slice happens to be most efficient, and publish it as a curve for the full system. Maximal
engagement is what keeps `system_tps` and `point_power_kw` descriptions of the *same* machine at
every point.

**Where a further replica genuinely does not fit.** `DP` is bounded by the formula above, so a
remainder smaller than one replica needs no justification. Where a submitter runs **fewer** replicas
than `floor(A_provisioned / A_replica)`, the shortfall and its reason MUST be declared in the point
YAML ([§8.3](#83-measurement-point-yaml)) and is subject to Methodology objection during peer
review. Accepted reasons include remaining accelerators that cannot satisfy the replica's
interconnect topology — a replica requiring a single NVLink domain cannot be formed from
accelerators spread across nodes — or a non-accelerator resource such as host memory or NIC count
binding before accelerator count. That the serving framework does not implement data parallelism is
not on its own sufficient.

**Scaling is per node.** The unit of scaling is a whole node, never a fraction of one — the same
granularity rule as [§4.5.2.1](#4521-rack-level-node-scaling), and for the same reason: a node's
power does not scale down with the accelerators left idle inside it. For each node set `s`:

```
Y_s = ceil( accelerators_used_s / accelerators_per_node_s )
```

`Y_s` is the number of nodes of set `s` engaged at that point, `N_s` the number provisioned. **Any
fractional node rounds up to a whole node.** A point engaging 42 accelerators on nodes holding 8
each uses `ceil(42 / 8) = 6` nodes, and is normalized as though all 48 accelerators in those 6 nodes
were provisioned to it.

**The denominator scales by node fraction, uniformly.** Every term of
[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)'s power model scales together,
including scale-out switch power:

```
point_power_kw = Σ over node sets s:  P_s × (Y_s / N_s)   +   S × (Σ Y_s / Σ N_s)
```

where `P_s` is node set `s`'s contribution to provisioned power and `S` is the scale-out switch
power. For a homogeneous submission — a single node set — this collapses to:

```
point_power_kw = provisioned_power_kw × (Y / N)
```

**Single-node submissions are unaffected.** With `Y = N = 1` the ratio is 1 at every point, so
`point_power_kw = provisioned_power_kw` across the whole pareto curve and the normalized curve is
the throughput curve scaled by a single constant, exactly as before.

| Submission | Denominator |
|---|---|
| Single node | Constant across the pareto. The full node power, every point. |
| Multi-node, rack-scale, or heterogeneous | Varies per point with `Y_s`. A complete `system_power.json` ([Appendix E](#appendix-e-system_powerjson-field-reference)) is **required**, since the per-set provisioned power `P_s` and node count `N_s` come from it. |

**Declaring the node count.** A submitter electing per-point normalization declares the nodes used
at each point in that point's measurement YAML ([§8.3](#83-measurement-point-yaml)), **by node
type**: a single count for a homogeneous system, one count per node type for a heterogeneous one.
The declaration is **optional**. Where it is absent for a point, that point is normalized by the
**full** provisioned power — `Y_s = N_s` for every set. The conservative figure is the default;
scaling it down requires an affirmative, checkable declaration.

**The declaration must match the serving deployment.** The declared node count is the set of nodes
holding model weights and able to receive requests for that point. It MUST be consistent with the
parallelism configuration declared in `system_desc.json`
([§8.2](#82-system-description-system_descjson)) — `tensor_parallel`, `pipeline_parallel`,
`expert_parallel`, and `data_parallel` — and reproducible on a re-run of that point. Declaring fewer
nodes than the deployment actually spans is a misrepresentation under
[Submission Rules §8.4](endpoints_submission_rules.md#84-issues-discovered-after-publication).

> [!IMPORTANT]
> **This is a deployment property, not a utilization measurement.** A node loaded with weights and
> able to serve counts as used, whether or not it happened to receive a request during the
> steady-state window. Idle-but-deployed capacity draws power and is counted. A submitter is not
> credited for uneven load balancing across a deployment that spans every node, and — under the
> maximal-engagement rule above — may not narrow the deployment to earn a smaller denominator. The
> only reduction available is the one the hardware forces: a remainder too small to hold another
> replica.

Provisioned power itself remains a property of the *system*: two otherwise identical systems that
differ in provisioned power — because of power capping, for example — are **different systems**, and
`P_s` and `N_s` are the same for every point on a submission's pareto curve. Only `Y_s` varies.


---

## 5. Pareto Collection Methodology

### 5.1 What Is Measured

Each measurement point is a benchmark run at a specific target concurrency using the
benchmark-defined fixed-concurrency load pattern. Replacement queries and dependent turns are issued
according to benchmark-defined timing.

### 5.2 Pareto Curve Representation

The pareto curve is represented exclusively as a **step-function** plot. Each submitted measurement
point defines a discrete step at its concurrency level; between submitted points, the curve holds
constant at the last measured value. No interpolation, curve fitting, or smoothing is applied to the
official curve.

Visualization tools may optionally overlay interpolated or smoothed curves for readability, but
these must be clearly labeled as **"interpolated (not official)"** and must not replace the
step-function representation in official publications.

The Offline point ([§5.7](#57-offline-point)) appears on the curve as the throughput ceiling and is
labelled separately; it does not define a step in the fixed-concurrency step function.

### 5.3 Minimum Submission Requirements

#### Minimum Point Count

Each submission for a non-agentic benchmark must include a minimum of **8 measurement points**,
structured as **1 + 3 + 3 + 1**. Agentic benchmarks require **7 points** (`1 + 3 + 3`), since the
Offline point does not apply to them ([§5.7](#57-offline-point)):

| Points | Placement |
|---|---|
| 1 mandatory point | One point in the [Ultra Low Concurrency region](#low-latency-region). |
| 3 mandatory points | One point in each of the three [Concurrency regions](#concurrency-regions) (Low Concurrency, Medium Concurrency, High Concurrency). |
| 3 submitter's-choice points | Three additional distinct points, placed as specified [below](#submitters-choice-points). |
| 1 mandatory **Offline** point | The unconstrained-throughput point defined in [§5.7](#57-offline-point). Required for every non-agentic submission; not applicable to agentic benchmarks. Where the submitter elects the $C_{max}$ point as the Offline result ([§5.7.2](#572-relationship-to-maximum-supported-concurrency)), no separate run is required and the minimum is 7. |

Accuracy validation runs are required at exactly `N` points, and at no others: the four mandatory
concurrency points — one low-latency point in the Ultra Low Concurrency region and one point in each
of the Low Concurrency, Medium Concurrency, and High Concurrency regions — plus the Offline point
for non-agentic benchmarks. `N` is therefore **5** for non-agentic benchmarks and **4** for agentic
benchmarks. Submitter's-choice points carry no accuracy validation run. Every agentic performance
point, including all submitter's-choice points, still requires its own inline accuracy score;
these scores are excluded from `N` ([§4.3](#43-accuracy-metric)).

#### No Spacing Requirements

There is no requirement to space points evenly within or across regions. Submitters choose the exact
concurrency levels that best characterize their system. This freedom enables submitters to cluster
points around inflection points, highlight sweet-spot operating points, or demonstrate consistent
performance across a region.

#### Submitter's-Choice Points

The 3 submitter's-choice points may be placed in any of the Low, Medium, or High Concurrency
regions, including regions that already have a required point. Submissions that declare
`c_min_override` may also place them in the Ultra Low Concurrency region. For example, a submitter
could place all 3 additional points in the High Concurrency region to demonstrate scaling behavior,
or distribute them to show overall consistency. All seven concurrency points MUST be distinct.

### 5.4 Regions of Interest

> [!CAUTION]
> Regions of Interest (ROIs) are named for either latency or concurrency, and in both cases they are
> constrained by concurrency. Please read the methodology carefully before proceeding.

The concurrency space is divided into four distinct regions.

#### Ultra Low Concurrency Region <a id="low-latency-region"></a>

| Property | Value |
|---|---|
| Required points | 1 |
| Concurrency range | 1–$C_{min}$ (inclusive), with $1 \le C_{min} \le 32$. |

By default, $C_{min}$ is the lowest submitted fixed concurrency. The point at $C_{min}$ is the
mandatory Ultra Low point; all other fixed-concurrency points are in the Low, Medium, or High
Concurrency regions.

Alternatively, submitters may set `c_min_override` in their `system_desc.json` to define $C_{min}$
independently of the lowest measured concurrency. A point at the declared $C_{min}$ is required,
but any submitted point in $[1, C_{min}]$ may be designated as the mandatory Ultra Low point.
Additional points in that region are permitted as submitter's-choice points.

In both cases, $C_{min} \le 32$ guarantees a measurement in the 1–32 range. Region placement and
accuracy selection follow these boundaries; the duration and dataset requirements in
[§6](#6-run-requirements-per-measurement-point) use fixed concurrency thresholds independently.

*Rationale:* Low-concurrency operation is critical for interactive applications (chatbots, coding
assistants, real-time translation). Additionally, the override option distinguishes the region
boundary from the lowest measurement while retaining at least one point in the 1–32 range for
cross-submission comparability.

> [!NOTE]
> Submitters are encouraged — but not required — to include a measurement at concurrency 1 (the
> single-user baseline). Concurrency 1 represents the best-case per-user experience and is commonly
> cited in performance comparisons. The mandatory Ultra Low point must follow the placement and
> accuracy-selection rules above.

#### Maximum Supported Concurrency

The concurrency regions are defined using the **Ultra Low Concurrency Region bound**
$C_{min}$ and a **maximum supported concurrency** value $C_{max}$ (this is the highest
concurrency level at which the submitter chooses to benchmark their system).
$C_{max}$ is the upper bound of the **High Concurrency Region**.

Rules:

- By default, $C_{min}$ is the lowest submitted fixed concurrency and MUST be in `[1,32]`.
  $C_{max}$ is defined by the submitter as the upper bound of the High Concurrency region.
- Submitters may alternatively choose to set $C_{min}$ using top-level `c_min_override`
  in `system_desc.json`. It MUST be a JSON integer in `[1,32]` equal to the
  concurrency of a submitted point on that curve. All other values, including `null`, are rejected.
- Either every point of a submission curve MUST omit the field `c_min_override`, or every point MUST
  declare the same value. Different pareto curves may choose different values.
- $C_{max}$ >> $C_{min}$.
- There is no compliance test to force a particular value of $C_{max}$.
- Submitters are incentivized to choose well: $C_{max}$ defines the extent of their published pareto
  curve, while $C_{min}$ determines how its concurrency regions are divided.
- The value of $C_{max}$ defines the upper bound of the High Concurrency region.

Override declarations are fixed once peer review starts; see
[Submission Rules §8.1](endpoints_submission_rules.md#81-corrections).

#### Concurrency Regions <a id="concurrency-regions"></a>

Beyond the `Ultra Low Concurrency` region (concurrency > $C_{min}$), the concurrency space
up to $C_{max}$ is divided into **three equal regions in logarithmic space (base 2)**.

**Region Boundary Computation**

Given a declared Maximum Supported Concurrency $C_{max}$, the log-space interval `I` is:

```
I = log2(C_max - C_min) / 3
```

The three concurrency regions are:

| Region | Start | End |
|---|---|---|
| Low Concurrency | $C_{min}+1$ | $round(C_{min} + 2^{I})$ |
| Medium Concurrency | `low_conc_end + 1` | $round(C_{min} + 2^{2I})$ |
| High Concurrency | `med_conc_end + 1` | $C_{max}$ |

All non-integer boundaries are rounded to the nearest integer using **round-half-to-even (banker's
rounding)**, consistent with Python's built-in `round()` function used in the reference
implementation.

> **Why logarithmic spacing?** Logarithmic spacing reflects how system behavior changes: the
> difference between concurrency 1 and 10 is far more significant than between 1000 and 1010.
> Log-space division ensures each region represents a similarly meaningful range of behavioral
> change, regardless of absolute concurrency scale.

**High Concurrency Margin**

The High Concurrency region has a **10% margin** beyond $C_{max}$, extending the valid upper bound
to $ceil(1.10 * C_{max})$.

This margin allows submitters to add points above their initial $C_{max}$ during the post-submission
update window (see [Submission Rules §8.1](endpoints_submission_rules.md#81-pareto-updates)) without
requiring a complete redefinition of region boundaries. The margin does not affect the required
point distribution.

**Worked Examples**

**Override example:** With a lowest measurement at 16, another at 32, and `"c_min_override": 32`, both
are in the Ultra Low Concurrency region and either may supply its one required accuracy result. With
$C_{max}=1024$, the
remaining regions are Low 33–42, Medium 43–131, and High 132–1024. If the override is not set, then
$C_{min}=16$ and the boundaries follow Example C.

<details>
<summary><strong>Example A — Large-Scale System ($C_{min} = 32$; $C_{max} = 8,192$)</strong></summary>

```
I = log2(8192 - 32) / 3 = log2(8160) / 3 = 12.994 / 3 = 4.331

Region boundaries:
  Ultra Low Concurrency: concurrency 1 – 32
  Low Concurrency:    concurrency   33 –   52  (round(32 + 2^4.331) = round(32 + 20.1) = 52)
  Med Concurrency:    concurrency   53 –  437  (round(32 + 2^8.663) = round(32 + 405.2) = 437)
  High Concurrency:   concurrency  438 – 8192

Minimum 7-point example: {32, 40, 200, 500, 1000, 2000, 4096}
```
</details>

<details>
<summary><strong>Example B — Smaller System ($C_{min} = 1$; $C_{max} = 256$)</strong></summary>

```
I = log2(256 - 1) / 3 = log2(255) / 3 = 7.994 / 3 = 2.665

Region boundaries:
  Ultra Low Concurrency: concurrency 1 – 1
  Low Concurrency:   concurrency 2 –  7  (round(1 + 2^2.665) = round(1 + 6.34) = 7)
  Med Concurrency:   concurrency 8 –  41  (round(1 + 2^5.33) = round(1 + 40.21) = 41)
  High Concurrency:  concurrency 42 – 256

Minimum 7-point example: {1, 4, 16, 32, 64, 128, 256}
```
</details>

<details>
<summary><strong>Example C — Mid-Range System ($C_{min} = 16$; $C_{max} = 1,024$)</strong></summary>

```
I = log2(1024 - 16) / 3 = log2(1008) / 3 = 9.977 / 3 = 3.326

Region boundaries:
  Ultra Low Concurrency: concurrency 1 – 16
  Low Concurrency:   concurrency  17 –   26
  Med Concurrency:   concurrency  27 –  117
  High Concurrency:  concurrency 118 – 1024

Minimum 7-point example: {16, 24, 64, 96, 128, 256, 1000}
```
</details>

**Boundary Edge Cases**

- **$C_{max}$ ≤ 33:** All three concurrency regions collapse to approximately one level each.
  Submitters with $C_{max} ≤ 33$ must notify the working group and provide written justification.
  The working group will review and may request additional information before accepting the
  submission.
- **$C_{max}$ > 100,000:** The algorithm scales correctly. The Low Concurrency region will be narrow
  while the High Concurrency region spans most of the range, reflecting the log-scale nature of
  concurrency scaling.
- **Region boundary collisions:** If rounding causes two boundaries to be equal, the affected region
  has zero width and a single valid concurrency level at the boundary value. One point at that level
  satisfies the region's requirement.

### 5.5 Region Boundary Reference Algorithm

The following pseudocode defines the authoritative computation. Submitters must use the reference
implementation in the MLCommons Endpoints repository to compute their boundaries and validate their
submitted points.

```python
def compute_regions(C_max: int, C_min: int) -> dict:
    assert 1 <= C_min <= 32, "Ultra low concurrency region boundary must be between 1 and 32 (inclusive)"
    assert C_max > 32, "Maximum Supported Concurrency must be > 32"

    # Ultra Low Concurrency region, with C_min selected according to §5.4.
    low_latency = {"start": 1, "end": C_min}

    # Compute log-space interval
    I = math.log2(C_max - C_min) / 3

    # Concurrency region boundaries (banker's rounding)
    low_conc_end = round(C_min + 2**I)
    med_conc_end = round(C_min + 2**(2 * I))

    low_concurrency  = {"start": C_min + 1,              "end": low_conc_end}
    med_concurrency  = {"start": low_conc_end+1,  "end": med_conc_end}
    high_concurrency = {"start": med_conc_end+1,  "end": C_max}

    # Extended High Concurrency margin (10%)
    margin_end = math.ceil(1.10 * C_max)

    return {
        "low_latency":      low_latency,
        "low_concurrency":   low_concurrency,
        "med_concurrency":   med_concurrency,
        "high_concurrency":  high_concurrency,
        "margin":           {"start": C_max+1, "end": margin_end},
    }
```

### 5.6 Maximum Point Cap

At no point shall the total number of measurement points on a single submission's pareto exceed **32
points** (including post-submission additions).

*Rationale:* The 32-point cap balances comprehensive characterization against review burden and
result presentation clarity.

The Offline point ([§5.7](#57-offline-point)) counts toward this cap.

### 5.7 Offline Point

> [!CAUTION]
> **`[TENTATIVE — Pending working-group ratification]`** The Offline point is newly introduced for
> Endpoints v1.0.

Every submission for a **non-agentic benchmark** MUST include one **Offline** measurement point. It
is the analogue of the **Offline scenario** in
[MLPerf Inference](https://github.com/mlcommons/inference_policies/blob/master/inference_rules.adoc):
all queries are available to the system under test at once, and the only question asked is how much
total throughput the system can sustain when nothing constrains it.

> [!NOTE]
> **Not applicable to agentic benchmarks.** Offline is out of scope for agentic workloads in
> Endpoints v1.0. An agentic submission neither requires nor may include an Offline point. Extending
> Offline to agentic workloads — including how "all queries at once" and the reported concurrency
> would be defined for multi-turn trajectories with tool-call dependencies between turns — is
> deferred to the working group for ratification in a later version.

#### 5.7.1 Definition

- **All queries are available at once.** The client makes the entire sample set available to the SUT
  at the start of the run rather than pacing issuance to hold a target concurrency. The SUT drains
  the queue at whatever rate it can.
- **Concurrency is the cardinality of the dataset.** Because the whole sample set is made available
  at once, the Offline point's reported `concurrency` is the **cardinality of the performance
  dataset** — the number of queries in one full pass over it. It is a property of the benchmark
  dataset rather than a value the submitter selects.
- **Throughput is the only metric of interest.** The Offline point reports `system_tps`, together
  with the `concurrency` defined above. It characterizes the system's peak aggregate capacity.
- **Latency metrics do not apply.** Because every query is queued at the start, a query at the back
  of the queue may wait arbitrarily long before its first token. **`ttft_p90_ms` is therefore
  unbounded in principle, is not required for the Offline point, and MUST NOT be used as a
  compliance criterion or an objection basis against it.**
- **Reordering is permitted, but bounded by the dataset.** Having the whole sample set available
  lets the SUT reorder and batch queries as it sees fit — grouping by sequence length, for example.
  That freedom is the point of the scenario. **Reordering MUST NOT cross a dataset-pass boundary.**
  Where a run issues more than one pass over the performance dataset —
  [§6.4](#64-minimum-completed-queries) requires the total sample count to be a positive integer
  multiple of the dataset size — each pass is a separate reordering domain, and a query from one
  pass MUST NOT be batched with, or reordered against, a query from another.

> [!IMPORTANT]
> **Why reordering stops at the dataset boundary.** Sorting across the whole multi-pass query set
> would let a submitter gather the repeated instances of the same underlying sample into one batch —
> identical or near-identical prompts executed together, with shared prefixes and uniform sequence
> lengths. That is an artifact of replaying a finite dataset, not a property of the serving system,
> and it sits immediately adjacent to the prohibition on coalescing identical queries in
> [§2.2.1](#221-general-rules). Confining reordering to a single pass keeps every batch's
> composition representative of one dataset.

#### 5.7.2 Relationship to Maximum Supported Concurrency

The Offline requirement is satisfied in one of two ways.

**Option 1 — a dedicated Offline run.** The Offline point is its own run under the Offline load
pattern ([§6.1](#61-load-pattern)). Such a run is not a fixed-concurrency point: it cannot serve as
the $C_{max}$ point, and it does not satisfy any region-coverage requirement of
[§5.3](#53-minimum-submission-requirements). Both of the following MUST hold between it and the
$C_{max}$ point:

| Quantity | Constraint |
|---|---|
| System throughput | `system_tps(Offline)` ≥ 0.98 × `system_tps` at the $C_{max}$ point |
| Concurrency | `concurrency(Offline)` ≥ $C_{max}$ |

**On the 2% throughput margin.** The margin exists to absorb run-to-run variation between the two
runs on the same system. It is a *tolerance, not a target*: Offline is expected to meet or exceed
the $C_{max}$ point, and the margin only prevents ordinary measurement noise from failing an
otherwise sound submission. It is tighter than the 5% same-system reproducibility margin of
[Submission Rules §6.6](endpoints_submission_rules.md#reproducibility-expectations) because both
runs come from the same submission, on the same configuration, close together in time.

*Rationale:* Offline removes every pacing constraint on the load generator, so it is by construction
an upper bound on what the fixed-concurrency points can achieve. An Offline result materially below
the $C_{max}$ point indicates either that the run did not saturate the system or that the $C_{max}$
point was measured under conditions the Offline run did not reproduce. Either way the submission
does not characterize its own ceiling, and the points are inconsistent.

An Offline point that violates either constraint is flagged at automated compliance ([§9.1](#91-automated-checks)).

**Option 2 — elect the $C_{max}$ point as the Offline result.** A submitter MAY instead **elect**
their $C_{max}$ point as the Offline result, declaring that it is already the highest aggregate
throughput their system achieves, irrespective of load pattern. Under this election:

- The elected run **remains a fixed-concurrency pareto point** and keeps its role as the $C_{max}$
  point. It is *additionally* reported as the Offline result. Its latency metrics remain defined and
  reported as for any other fixed-concurrency point — the [§5.7.1](#571-definition) exemptions for
  TTFT and the dataset-pass reordering bound apply to a dedicated Offline run, not to an elected
  point.
- `system_tps(Offline)` and `concurrency(Offline)` are those of the $C_{max}$ point by definition.
  The constraints above are met with equality, and the 2% margin does not apply — there is no second
  run and therefore no run-to-run variation to absorb.
- The election MUST be declared in the measurement point YAML ([§8.3](#83-measurement-point-yaml)),
  so that a reader can tell the Offline result was elected rather than measured under the Offline
  pattern.
- The submission then contains one fewer distinct run. The
  [§5.3](#53-minimum-submission-requirements) minimum is met with **7 runs**, the $C_{max}$ point
  counting once as its region point and once as the Offline result.

*Why this needs no verification:* electing can only understate the system. An unpaced Offline run is
by construction an upper bound on what any fixed-concurrency point can achieve, so a submitter who
elects forgoes whatever additional throughput a dedicated Offline run might have shown. The election
is a conservative declaration against the submitter's own interest, not an optimization — which is
why it is permitted on the submitter's word.

#### 5.7.3 Presentation

A **dedicated** Offline run is plotted on the pareto curve as the system's **throughput ceiling** —
the highest `system_tps` the submission reports. It is labelled **"Offline"** and is visually
distinguished from the fixed-concurrency points, because it is measured under a different load
pattern and carries no meaningful latency coordinate.

An **elected** Offline result adds no separate marker: the $C_{max}$ point is plotted as the
fixed-concurrency point it is, and additionally labelled as the Offline result.

All other run requirements of [§6](#6-run-requirements-per-measurement-point) — minimum duration,
minimum completed queries, dataset handling, and the accuracy requirement — apply to the Offline
point as they do to any other measurement point, except that the load pattern is the Offline pattern
rather than the fixed-concurrency pattern ([§6.1](#61-load-pattern)).

---

## 6. Run Requirements Per Measurement Point

> [!WARNING]
> **WORK IN PROGRESS** — This entire section is under active development. Final values will be
> determined through working group discussion and empirical validation.
> Current constraints, thresholds, and duration values are locked for the v0.7 submission in June
> 2026 and have been approved by the TaskForce.

### 6.1 Load Pattern

All measurement points except the Offline point must use the benchmark-defined fixed-concurrency
load pattern. The `target_concurrency` setting specifies the exact concurrency level for each point.
Other load patterns (`Poisson` and similar) are not valid for fixed-concurrency pareto submission
points.

The **Offline point** ([§5.7](#57-offline-point)) instead uses the benchmark-defined Offline load
pattern, in which the entire sample set is made available to the SUT at once rather than paced to a
target concurrency. It is the only point for which the fixed-concurrency pattern is not used.

### 6.2 Minimum Run Duration

*(Example values — subject to ratification.)*

Each measurement point must sustain the target concurrency for a minimum duration of steady-state
measurement, excluding warmup. These values correspond to the `min_duration_ms` setting in
`RuntimeSettings`.

The duration thresholds below and the dataset requirements in §§6.4–6.5 depend on the point's
concurrency, independently of its §5.4 region or `c_min_override`. For example, concurrency 24 uses
the 1–32 requirements even when `c_min_override=16` places it in the Low Concurrency region.

For Legacy LLM (non-agentic) benchmarks:

| Concurrency | Minimum Duration (steady state) | Rationale |
|---|---|---|
| 1–32 (inclusive) | 600 seconds | Reduced duration accounts for slower query completion at low concurrency. |
| Greater than 32 | 1200 seconds | Standard duration for statistical confidence at scale. |

For all Agentic benchmarks:

| Concurrency | Minimum Duration (steady state) | Rationale |
|---|---|---|
| 1–32 (inclusive) | 600 seconds | Reduced duration accounts for slower query completion at low concurrency. |
| Greater than 32 | 2400 seconds | Standard duration for statistical confidence at scale. |

### 6.3 Warmup Period

*(Requirements below are subject to working group ratification.)*

A warmup period may precede every measurement period. Warmup events — all requests issued before
`TEST_STARTED` — are excluded from metric computation. The purpose of warmup is to bring the system
to steady state (populated connection pools, warm caches, calibrated scheduler) before any data
contributing to reported metrics is collected. The warmup period is optional but must not exceed 24
hours, per measurement point.

#### 6.3.1 Prohibited Warmup Data

Warmup requests must not use any sample from the benchmark performance dataset. This prohibition
covers direct use, subsets, truncations, or any query whose content was derived from performance
dataset samples.
If the inference client uses benchmark performance dataset, then *salting must be enabled*.

The accuracy dataset and any other data source not drawn from the performance dataset are permitted
for warmup.

> [!WARNING]
> For v0.7, the inference client may use performance dataset during warmup. In such case - salting
> must be enabled. The salting flag is not enabled by default — submitters must manually enable it
> in the client config and also disable KV cache reuse.

#### 6.3.2 Discard Policy

All requests issued before `TEST_STARTED` are warmup requests and must not appear in any reported
metric. Warmup request logs must be retained and available for reviewer inspection.

#### 6.3.3 Documentation Requirements

Beyond the constraints above, warmup is at the submitter's discretion. Because warmup state
materially affects the measurement (KV cache population, JIT compilation, scheduler calibration),
the full warmup procedure must be documented in sufficient detail for an independent team to
reproduce it. Each submission must declare, in the measurement point metadata (see
[§8.3](#83-measurement-point-yaml)):

- Total warmup duration (seconds from the first warmup request to `TEST_STARTED`).
- Total warmup requests issued and completed.
- Warmup data source and content description (e.g., dataset name and split, synthetic generation
  method and parameters, or fixed prompt text).
- Concurrency level used during warmup.
- Any platform-specific initialization steps performed (e.g., CUDA graph capture, engine loading,
  JIT compilation triggers), and confirmation that initialization was complete before
  `TEST_STARTED`.

> [!NOTE]
> Reviewers may request warmup logs as part of a reproducibility objection. Incomplete or ambiguous
> warmup documentation is grounds for a Methodology objection under
> [Submission Rules §6.8](endpoints_submission_rules.md#68-types-of-objections).

### 6.4 Minimum Completed Queries

*(Example values — subject to ratification.)*

Each measurement point must complete a minimum number of queries (`min_sample_count` in
`RuntimeSettings`). The minimum ensures sufficient statistical confidence in the reported percentile
metrics.

| Concurrency | Minimum Completed Queries | Rationale |
|---|---|---|
| 1–32 (inclusive) | One pass over the dataset, or the approved subset under §6.5 | An approved subset reduces run time at low concurrency. |
| Greater than 32 | One pass over the dataset | Consistent and comparable accuracy across all runs. |

> [!NOTE]
> These minimum query counts require statistical validation against required sample sizes for target
> confidence intervals. Values are subject to adjustment pending working group ratification.

At each measurement point, the total number of samples issued MUST be a positive integer multiple of
the dataset size.

### 6.5 Dataset Considerations

*(Example constraints — subject to ratification.)*

- Performance runs use `WithReplacementSampleOrder` (random sampling with replacement from the
  performance dataset).
- Accuracy runs use `WithoutReplacementSampleOrder` (each sample exactly once).
- For runs at concurrency 1–32 (inclusive), a representative subset of the dataset may be used
  (configured via `n_samples_from_dataset`) to reduce run time, subject to pre-approval by the
  working group. The subset must be documented and identical across all submitters.
- Fixed-concurrency performance runs must use streaming responses (`model_params.streaming` resolves
  to `on`) so that TTFT and TPOT can be measured. A dedicated Offline run
  ([§5.7](#57-offline-point)) is exempt, since it reports only `system_tps`. The SUT's streaming
  behavior, including any multi-token stream interval, is governed by
  [§2.9.7](#297-post-processing-equivalence).
- `stream_all_chunks` is a client-side setting; either value is permitted, and the value used is
  recorded in `runtime_settings` ([§8.3](#83-measurement-point-yaml)).

### 6.6 Accuracy Requirement

*(Example constraint — subject to ratification.)*

Accuracy validation runs are required at the points defined in
[§5.3](#53-minimum-submission-requirements). The accuracy runs verify that the system meets the
benchmark's quality target and MUST follow the applicable single-turn or multi-turn requirements in
[§4.3](#43-accuracy-metric).

Agentic benchmarks additionally require inline accuracy from every submitted performance run,
including submitter's-choice points. Each score MUST meet the per-point inline requirement in
[§4.3](#43-accuracy-metric) and be included in the point's `accuracy_results.json`
([§8.1](#81-directory-structure)).

---

## 7. Publication Status

Publication status categories — **Available**, **Preview**, and **RDI** — including the four-point
availability test, public evidence rules, software stack requirements, the 180-day Preview
commitment, the 221-day RDI cooling-off period, and the [CUSTOM-SKU] open question are defined in
[MLPerf Endpoints Submission Rules §7](endpoints_submission_rules.md#7-publication).

---

## 8. Submission Requirements

### 8.1 Directory Structure

An Endpoints submission must follow this directory structure:

```
<submitting_organization>/
  └── [submission_id]/                      # Provided by MLC. Each submission can only have 1 submission_id. 
      │
      ├── src/                              # SHARED across the whole submission
      │   └── <implementation>/          # e.g. trtllm/, vllm/, sglang/
      │       ├── README.md                 # how to build/launch the SUT and reproduce a point
      │       └── <endpoint interface code, infra/cluster setup, client harness>
      │
      ├── docs/                             # SHARED across the whole submission
      │   ├── calibration.adoc              # if weight transformations applied (§3.3)
      │   ├── software_disclosure.md        # §8.4
      │   └── <additional documentation>
      │
      └── results/
          └── <system>/                     # e.g. H200-SXM-141GBx8_TRT/
              ├── system_power.json            # §4.5.2, Appendix E — REQUIRED, one per system.
              │                                #   Provisioned power + per-set node counts N_s.
              │                                #   Per-point scaling: point.yaml `nodes_used` (§4.5.3).
              └── <model_name>/        # e.g. deepseek-r1/, gpt-oss-120b/. MLC maintains a list of canonical model names for each benchmark.
                  └── r<N>/                 # one PARETO POINT per concurrency level (r1, r32, r256, …)
                      ├── point.yaml              # §8.3 — includes shared_src / shared_docs pointers
                      ├── config.yaml             # Loadgen configuration file.
                      ├── result_summary.json     # aggregate metrics (QPS, TPS, TTFT, TPOT, ISL, %iles)
                      ├── accuracy_results.json   # §6.6
                      ├── system_desc.json       # framework/parallelism/precision for this point
                      └── server_configs/         # OPTIONAL, point-specific: backend configs tuned
                                                  #   for THIS concurrency (batch size, max_seq_len,
                                                  #   KV cache %, TP/EP/PP). Non-standard — layout is
                                                  #   submitter-defined. May include its own README.md.
```

For agentic benchmarks, every point MUST include `accuracy_results.json` with the inline score
from its performance run (the client's `dataset_type: performance` entry). At each of the four
mandatory accuracy points, the same file MUST also include the SWE-bench validation result.
When performance and accuracy validation are executed separately, retain both results in that
point's file. An inline-only file at another point is required and does not count toward the
fixed number of accuracy validation runs ([§4.3](#43-accuracy-metric)).

The tree separates content that can be shared across the submission from content that is genuinely
per-measurement-point:

- **Shared content** (`src/`, `docs/`) is written once per submission. Infrastructure code (cluster
  instantiation, endpoint setup, client harness) and documentation are not duplicated per Pareto
  point. A submitter that needs different code or documentation for different systems or models adds
  another `src/<implementation_id>/` or a subdirectory under `docs/` rather than duplicating the tree.
- **Point-specific content** is only what varies with concurrency level: `point.yaml`, the result
  and metadata
  JSON files, and the optional `server_configs/`. Adding, replacing, or withdrawing a Pareto point
  must not
  require any change under `src/` or `docs/`.

Each point declares which shared content it used via the `shared_src` and `shared_docs` pointers in
its `point.yaml` (see [§8.3](#83-measurement-point-yaml)). A point whose pointers are missing or do
not resolve to an existing directory is incomplete under [§9.1](#91-automated-checks).

### 8.2 System Description (`system_desc.json`)

Endpoints submissions must include the following metadata:

| Field | Description |
|---|---|
| `division` | `Standardized`, `Serviced`, or `RDI`. |
| `system_name` | Submitter selected string to describe the system under test (SUT). |
| `shortened_system_name` |  Shortened `system_name` that's at most 20 characters. |
| `system_availability_status` | `Available` , `Preview`, or `RDI` (not available for purchase soon) at submission time. |
| `availability_url` | URL of a public product or ordering page for the system. Required when `system_availability_status` is `Available` ([Submission Rules §7.3.7](endpoints_submission_rules.md#737-transition-to-available)). |
| `target_availability_date` | Expected availability date as an ISO 8601 date. Required when `system_availability_status` is `Preview` ([Submission Rules §7.3.4](endpoints_submission_rules.md#734-declaration-requirements)). |
| `system_size` | Number of accelerators per node type, e.g. "72 accelerators + 144 accelerators" for a system comprising two types of nodes with 72 accelerators in the first node type and 144 accelerators in the second node type. |
| `system_node_ensemble_count` | How many unique combinations of Hardware and Software are part of the SUT. |
| `system_node_ensemble_total` | Total number of nodes in the SUT, equal to the sum of all number_of_nodes.|
| `system_node_ensemble_id` | Identifies a unique node type within the SUT. |
| `number_of_nodes` | How many nodes of type system_node_ensemble_id are in the SUT. |
| `host_processor_model_name` | Model name of the host processor. |
| `host_processors_per_node` | # of host processors per node. |
| `host_processor_core_count` | # of CPU cores in each processor. Optional, but at least one of host_processor_core_count and host_processor_cpu_count must be present. |
| `host_processor_vcpu_count` | # of vCPUs in each processor. Optional, but at least one of host_processor_core_count and host_processor_vcpu_count must be present. |
| `accelerator_model_name` | Model name of the accelerator. |
| `accelerators_per_node` | # of accelerators per node. |
| `accelerator_host_interconnect` | Describes the interconnect link between the accelerator and the host processors. |
| `accelerator_interconnect` | Describes the interconnect link between accelerators, when multiple accelerators are present as indicated by accelerators_per_node > 1. |
| `accelerator_memory_capacity` | Memory capacity per accelerator. |
| `accelerator_memory_type` | Type of memory for the accelerator. |
| `host_memory_capacity` | Total memory capacity for all host processors. Not per-processor. |
| `host_memory_configuration` | Memory configuration for the host processors, e.g., how many DIMMs, what kind of memory (DDR5, LPDDR4, etc.), and speed. |
| `host_network_card_count` | Describes the # and type of networking cards and associated speeds. |
| `host_networking` | Describes the networking protocol, e.g., Infiniband, Ethernet. |
| `host_storage_capacity` | Total storage capacity for the node. |
| `host_storage_type` | Description of the type of storage in the node. |
| `other_hardware` | Describes any other performance relevant hardware in the node, freeform field. |
| `cooling` | Describes if the node uses any liquid cooling, only air-cooling, or only passive cooling. |
| `hw_notes` | Submitter hardware notes to supplement other information, freeform field. |
| `serving_framework` | Serving Framework used for submission, e.g., SGLang, vLLM, etc. |
| `inference_backend` | Inference backend used for submission, e.g., vendor stack components. |
| `driver` | Driver and version number for any accelerators. |
| `container_link` | Link to container for submission. |
| `max_supported_concurrency` | Declared Maximum Supported Concurrency `M`. |
| `c_min_override` | **Optional.** Declared lower region boundary, subject to [§5.4](#54-regions-of-interest). Omit to derive it from the points. |
| `endpoint_url` | URL or description of the endpoint under test. |
| `operating_system` | OS used for the node. |
| `filesystem` | Filesystem used for the node. |
| `other_software_stack` | Describes any other performance relevant software in the node, freeform field. |
| `sw_notes` | Submitter software notes to supplement other information, freeform field. |
| `node_config` | Describes the configuration of nodes or processors in the SUT (as described by system_size) for this run. Should be provided by submitter and contain sufficient detail to enable reproducing the submission (e.g., describing configuration of inference server for all nodes). |
| `config_summary` | Describes the configuration options for the SUT for this run — a concatenation of `disaggregated`, `tensor_parallel`, `pipeline_parallel`, `expert_parallel`, `data_parallel` (where these fields are > 1) and `config_summary_notes`. Should be provided by submitter and contain sufficient detail to enable reproducing the submission. |
| `disaggregated` | Indicates whether the system is disaggregated (disaggregated > 1). If disaggregated <!-- TODO: definition is truncated in the source data dictionary, verify full text with the data dictionary owner. --> |
| `expert_parallel` | Expert parallel partitioning of the model for the run. Only applies to Mixture-of-Expert models. EP=N means that the experts are split into N separate groups that reside on different processors/accelerators, and tokens are routed to the appropriate group. EP=1 means no partitioning. |
| `tensor_parallel` | Parallel partitioning of the model weight matrices for the run. TP=N means the weight matrices are split N ways and each partition contains 1/N of the weights of each layer and computes 1/N of the layer. Generally the number of attention heads in the model must be divisible by N. TP=1 means no partitioning. |
| `pipeline_parallel` | Sequential partitioning of the layers of the model into a pipeline for the run. PP=N means the layers of the model are split sequentially into a pipeline with N stages, each stage contains 1/N of the layers of the model. Each processor/accelerator contains one stage and a single inference must pass through all stages of the pipeline. PP=1 means no partitioning. |
| `data_parallel` | Data parallel replication of the model for the run. DP=N means the model is replicated N times, and requests are distributed across the N replicas. DP=1 means no replication. |
| `batch` | Maximum batch size. |
| `config_summary_notes` | Free form field from the submitter to contain information not captured by other fields that concatenate into config_summary. |
| `tps_utilization` | reported_system_tps / (max of all reported_system_tps for all runs) |

#### 8.2.1 Template Structure

`results/<system>/<model_name>/r<N>/system_desc.json` contains the fields defined in the table above.

The template omits the optional `c_min_override`; its declaration rules are in §5.4.

```json
{
  "division": "",
  "system_name": "",
  "shortened_system_name": "",
  "system_availability_status": "",
  "availability_url": "",
  "target_availability_date": "",
  "system_size": "",
  "system_node_ensemble_count": 0,
  "system_node_ensemble_total": 0,
  "endpoint_url": "",
  "serving_framework": "",
  "node_types": [
    {
      "system_node_ensemble_id": 0,
      "number_of_nodes": 0,
      "host_processor_model_name": "",
      "host_processors_per_node": 0,
      "host_processor_core_count": 0,
      "host_processor_vcpu_count": 0,
      "host_memory_capacity": "",
      "host_memory_configuration": "",
      "accelerator_info": [
        {
          "accelerator_model_name": "",
          "accelerators_per_node": 0,
          "accelerator_memory_capacity": "",
          "accelerator_memory_type": "",
          "accelerator_interconnect": "",
          "accelerator_host_interconnect": ""
        }
      ],
      "host_network_card_count": "",
      "host_networking": "",
      "host_storage_capacity": "",
      "host_storage_type": "",
      "other_hardware": "",
      "cooling": "",
      "hw_notes": "",
      "inference_backend": "",
      "driver": "",
      "operating_system": "",
      "filesystem": "",
      "container_link": "",
      "other_software_stack": "",
      "sw_notes": ""
    }
  ],
  "node_config": "",
  "disaggregated": 0,
  "expert_parallel": 0,
  "tensor_parallel": 0,
  "pipeline_parallel": 0,
  "data_parallel": 0,
  "batch": 0,
  "config_summary": "",
  "config_summary_notes": "",
  "tps_utilization": 0
}
```

### 8.3 Measurement Point YAML

Each measurement point must be accompanied by a YAML configuration file specifying:

| Field | Description |
|---|---|
| `concurrency` | The target concurrency level. |
| `region` | The region this point satisfies (`low_latency`, `low_concurrency`, `med_concurrency`, `high_concurrency`, or `submitters_choice`). |
| `runtime_settings` | The `RuntimeSettings` used for this run (load pattern, `min_duration_ms`, `min_sample_count`, `stream_all_chunks`, etc.). |
| `dataset` | Dataset name and any `n_samples_from_dataset` override (if applicable). |
| `warmup` | The warmup procedure declaration required by [§6.3.3](#633-documentation-requirements) — `duration_s`, `requests_issued`, `requests_completed`, `data_source` (description of the warmup data and its origin), `concurrency`, and `initialization_steps` (platform-specific setup completed before `TEST_STARTED`). |
| `division` | `Standardized`, `Serviced`, or `RDI`. <!-- TODO: also listed in §8.2 pending placement review --> |
| `max_supported_concurrency` | Declared Maximum Supported Concurrency `M`. |
| `model_name` | Display name of model, should be consistent across all external usages. |
| `model_precision` | Lowest precision numerical format used for the weights of the model. For example, if a model comprises FP16 and FP8, then model_precision is FP8. |
| `link_to_model` | Link to model submitted e.g., via GitHub. |
| `link_to_model_transformation` | Link to calibration/quantization/retraining write-up. |
| `model_notes` | Submitter software notes to supplement other information, freeform field. |
| `dataset_name` | Display name of dataset, should be consistent across all external usages. |
| `dataset_type` | Is the dataset used for "Accuracy", "Performance", or "Accuracy + Performance". |
| `offline` | Offline-point declaration ([§5.7](#57-offline-point)). One of: `dedicated` — this point is a dedicated Offline run; `elected` — this is the $C_{max}$ point, elected as the Offline result under [§5.7.2](#572-relationship-to-maximum-supported-concurrency); absent or `none` otherwise. Non-agentic benchmarks only. |
| `dataset_link` | Link to data used for submission e.g., via GitHub. |
| `steady_state` | The reporting block of [§4.4](#44-reporting-basis-steady-state-window) — `status` (`windowable` / `insufficient_duration` / `insufficient_passes` / `partial_dataset`), `window` (super-pass range, sample count, the effective super-pass size used, and `duration_s` — the window's issue-time span, checked against the [§6.2](#62-minimum-run-duration) minimum), per-metric `state` (`Plateau` / `Drifting Up` / `Drifting Down`), and `anomaly` (present only on a level shift); `total` metrics reported alongside as supplementary. |
| `speculative_decoding` | If used for this point: drafter model ID/checksum, precision, public release date, a link to the drafter's model card or technical report, and tokenizer-compatibility notes for the drafter/target pair. (Algorithm and per-point configuration are already covered by [§2.9.4](#294-speculative-decoding)'s disclosure requirements.) |
| `nodes_used` | **Optional.** The nodes engaged at this point, for per-point power normalization ([§4.5.3](#453-normalized-metric)). A list of `{ system_node_ensemble_id, nodes }` entries — one entry for a homogeneous system, one per node type for a heterogeneous one. `nodes` is a whole number of nodes, rounded up from the accelerators engaged ([§4.5.3](#453-normalized-metric)), and MUST NOT exceed the `N` provisioned for that node set in `system_power.json`. **Omitting this field normalizes the point by the full provisioned power of the system.** Single-node submissions need not declare it: the result is identical either way. |
| `dp_shortfall` | Present only where `DP` is lower than `floor(A_provisioned / A_replica)` ([§4.5.3](#453-normalized-metric)). Declares the `DP` actually run, the `DP` the formula gives, and the reason the further replica could not be formed. Subject to Methodology objection during peer review. |

### 8.4 Software Disclosure

For **Standardized** and **RDI** division submissions, all software components that substantially
determine ML performance must be disclosed. This includes, at minimum:

- Inference serving framework (name, version, commit hash or release tag).
- ML accelerator library (e.g., TensorRT-LLM, cuDNN — version and build).
- Driver version.
- Operating system.

For **Serviced** division submissions, disclose all software information available from public
documentation and API metadata.

### 8.5 Result ID

Two identifiers are attached to every submission, and they serve different purposes.

The **submission ID** is generated automatically by the submission pipeline as a hash. It is opaque,
carries no meaning, and exists so the lifecycle tooling can track a bundle through upload, review,
and amendment.

The **result ID** identifies a single published result and is human-readable. A result is one
published Pareto curve: one system, one benchmark model, one dataset. It is constructed as:

```
<major-version>.<minor-version>.<cohort-number>.<model_id>.<dataset_id>.<entry-number>
```

| Component | Description |
|---|---|
| `major-version` | Major version of the MLPerf Endpoints rules under which the result was submitted (e.g., `1` for v1.0). |
| `minor-version` | Minor version of the same (e.g., `0` for v1.0). |
| `cohort-number` | Cohort Number for this submission (e.g., `0` for the first cohort of a given version, `1` for the second, etc.)
| `model_id` | Benchmark model identifier from the round's supported model list ([§3.2](#32-supported-models)). Must match `model_name` in `point.yaml` ([§8.3](#83-measurement-point-yaml)). |
| `dataset_id` | Identifier of the dataset used for the performance and accuracy runs, as named in the benchmark definition ([§3.1](#31-benchmark-definition)) and recorded in each point's `dataset` field ([§8.3](#83-measurement-point-yaml)). |
| `entry-number` | Sequence number assigned at publication, unique within the preceding four components. |

Example: `1.0.0.deepseek-r1.mmlu-pro.7` — the seventh published DeepSeek-R1 result on MMLU-Pro under
the v1.0 rules.

Result IDs are assigned by MLCommons at publication; they are not chosen by the submitter and are
not part of the submitted bundle. They are stable and never reused — a result that is superseded,
withdrawn, or invalidated retains its result ID in the historical record, and the replacement result
receives a new entry number (see
[Submission Rules §8.1](endpoints_submission_rules.md#versioning-and-historical-record)).

---

## 9. Compliance Validation

### 9.1 Automated Checks

The compliance validator — run by the submitter before submission and by MLCommons upon receipt —
performs the following checks:

| Check | Validation | Failure Action |
|---|---|---|
| **Submission completeness** | All required files, YAML configurations, result artifacts, and system descriptions are present. | Reject submission. |
| **Shared path resolution** | Each point's `shared_src` and `shared_docs` resolve to an existing directory under the submission root. | Reject submission. |
| **Power descriptor** | A `system_power.json` conforming to the [Appendix E](#appendix-e-system_powerjson-field-reference) schema is present for each system, and passes the validation rules of [E.7](#e7-validation). Required for all Standardized submissions, CoP and CoN. | Reject submission. |
| **Per-point node declaration** | Where a point declares `nodes_used` ([§8.3](#83-measurement-point-yaml)): every `system_node_ensemble_id` resolves to a node set in `system_power.json`; each `nodes` value is a positive integer not exceeding that set's provisioned `N_s`; and the implied accelerator count is consistent with the parallelism configuration in `system_desc.json` ([§4.5.3](#453-normalized-metric)). | Reject submission. |
| **Per-point denominator** | `system_tps_per_kw` at each point equals `system_tps / point_power_kw` recomputed by the checker, with `Y_s = N_s` for any point not declaring `nodes_used`. | Reject submission. |
| **Maximal engagement** | Accelerators engaged at each point equal `DP × A_replica` with `DP = floor(A_provisioned / A_replica)`, derived from the point's parallelism configuration ([§4.5.3](#453-normalized-metric)). A point falling short without a `dp_shortfall` declaration is non-conforming. | Reject submission. |
| **Point count** | ≥ 8 total measurement points including a dedicated Offline run (non-agentic); ≥ 7 where the $C_{max}$ point is elected as the Offline result, or for agentic benchmarks. Choice-point placement follows §5.3. | Reject submission. |
| **Offline point present** | Exactly one point carries an `offline` declaration of `dedicated` or `elected` for non-agentic benchmarks; none is present for agentic benchmarks ([§5.7](#57-offline-point)). An `elected` declaration appears on the $C_{max}$ point. | Reject submission. |
| **Accuracy run count** | Exactly `N` accuracy validation results are present — 5 for non-agentic, 4 for agentic ([§4.3](#43-accuracy-metric)) — one per mandatory point, and none at a submitter's-choice point. Inline accuracy scores neither count toward `N` nor count as extra validation runs. More or fewer than `N` validation results is non-conforming. | Reject submission. |
| **C_min override** | Declaration type, range, matching submitted point, and consistency satisfy §5.4. | Reject submission. |
| **Ultra Low Concurrency coverage** | ≥ 1 point in the Ultra Low Concurrency region defined by §5.4. | Reject submission. |
| **Low Concurrency coverage** | ≥ 1 point in the Low Concurrency region. | Reject submission. |
| **Medium Concurrency coverage** | ≥ 1 point in the Medium Concurrency region. | Reject submission. |
| **High Concurrency coverage** | ≥ 1 point in the High Concurrency region. | Reject submission. |
| **Max concurrency declared** | $C_{max} > 32$; declared in `system_desc.json`. | Reject submission. |
| **Point cap** | ≤ 32 total measurement points. | Reject points beyond 32. |
| **Concurrency in range** | Each point's concurrency falls within a valid region (including the 10% High Concurrency margin), computed using the reference algorithm in [§5.5](#55-region-boundary-reference-algorithm). | Flag out-of-range points. |
| **Offline ordering** | For a `dedicated` Offline run: `system_tps(Offline)` ≥ 0.98 × `system_tps` at the $C_{max}$ point, and `concurrency(Offline)` ≥ $C_{max}$ ([§5.7.2](#572-relationship-to-maximum-supported-concurrency)). Not applicable to an `elected` point. | Flag non-compliant submission. |
| **Load pattern** | All points except the Offline point used the benchmark-defined fixed-concurrency load pattern; the Offline point used the Offline pattern. | Reject non-conforming points. |
| **Run duration** | Each point meets the minimum steady-state duration for its region (see [§6.2](#62-minimum-run-duration)). | Flag non-compliant points. |
| **Minimum query count** | Each point meets the minimum completed queries for its region (see [§6.4](#64-minimum-completed-queries)). | Flag non-compliant points. |
| **Streaming config** | Streaming enabled (`model_params.streaming` resolves to `on`) for all fixed-concurrency performance runs; a dedicated Offline run is exempt. | Flag non-compliant points. |
| **Warmup metadata** | Each point's YAML declares the warmup fields required by [§6.3.3](#633-documentation-requirements) (`duration_s`, `requests_issued`, `requests_completed`, `data_source`, `concurrency`, `initialization_steps`). | Flag non-compliant points. |
| **Warmup logs retained** | Warmup request logs are retained and available for reviewer inspection (see [§6.3.2](#632-discard-policy)). | Flag non-compliant points. |
| **Metric consistency** | The valid per-response TPOT distribution must be non-empty with a finite, strictly positive P90; the normalized P90 value in milliseconds is `tpot_p90_ms` and `tps_per_user = 1000 / tpot_p90_ms`. The authoritative result schema defines TPOT serialization and units. | Flag inconsistent points. |
| **Agentic metric consistency** | Reported agentic metrics are derivable from their §4 definitions. | Flag inconsistent points. |
| **Accuracy** | Required results satisfy §4.3's band assignment and single-turn or multi-turn gate. | Reject submission. |
| **Accuracy validation** | Validation results are present for all points required by §5.3 and satisfy the applicable single-turn or multi-turn gate in §4.3. For agentic benchmarks, the mean is over the four SWE-bench results only. | Reject submission. |
| **Agentic inline accuracy** | Every submitted agentic performance point, including all submitter's-choice points, reports an inline score from its performance run and individually meets the model-specific threshold (§4.3). Missing, null, or below-threshold scores are non-conforming. | Reject submission. |
| **Seed-set validity** | For an initial submission, every point must record the same seed set, and that set must have been published for `target_cohort` or one of the three immediately preceding cohorts. For an amendment, every new or replacement point must match the original submission's bound seed set; the four-cohort adoption test is not reapplied using the amendment's later cohort. See [Submission Rules §4.6](endpoints_submission_rules.md#46-seed-rotation). | Reject submission. |
| **Configuration consistency** | Same model, endpoint configuration, software stack, and seed set across all measurement points. | Flag inconsistencies. |
| **Approved drafter** | For points using speculative decoding, the disclosed drafter matches an entry on the benchmark's published approved drafter list — by weight checksum, or by target checksum plus configuration for a configuration-identified entry — or is a native head, identified by the model ID and checksum of an approved checkpoint that ships it ([§2.9.4](#294-speculative-decoding)/[§3.2](#32-supported-models)). | Reject non-conforming points. |
| **Drafter approval lead time** | The drafter used was approved at least two cohorts before the submission's `target_cohort`; a native head counts from its checkpoint's approval cohort ([§2.9.4](#294-speculative-decoding)). | Reject non-conforming points. |

### 9.2 Manual Review Focus Areas

Human reviewers should focus on aspects that automation cannot easily verify:

- Whether the pareto curve shape is physically plausible (throughput should generally increase with
  concurrency up to saturation, then plateau or decrease).
- Whether metric distributions suggest artificial manipulation (e.g., suspiciously uniform TTFT
  values across very different concurrency levels).
- Whether TTFT-triggering fragments are genuine parts of the model response rather than meaningless
  leading content emitted to stop the TTFT clock.
- Whether content is duplicated across assistant-response fields or padded to inflate the official
  output-token count.
- For the Offline point, whether query reordering stayed within dataset-pass boundaries
  ([§5.7.1](#571-definition)) — batches composed of repeated instances of the same underlying sample
  indicate sorting across passes.
- Whether warmup requests drew on any sample from the performance dataset (prohibited under
  [§6.3.1](#631-prohibited-warmup-data)); reviewers may cross-check retained warmup logs against the
  performance dataset.
- Whether the system description accurately reflects the hardware and software used.
- Cross-submission consistency for the same hardware platform.
- Division eligibility (especially Serviced division API compliance and availability status).
- Whether post-submission updates are consistent with the original submission's system configuration.
- For speculative-decoding submissions: whether the disclosed drafter/algorithm is consistent with
  the benchmark's approved list — informs a Methodology objection under
  [Submission Rules §6.8](endpoints_submission_rules.md#68-types-of-objections), not a
  SpecDec-specific check.

---

---

## Appendix A: Open Questions and Working Group Items

> [!WARNING]
> **WORK IN PROGRESS** — This appendix collects items that require working group decision before the
> rules can be finalized. Items marked **Open** do **not** represent current policy; they are
> placeholders for decisions in progress.

Entries marked **Resolved** record a decision that has been taken and is already reflected in the
operative sections; they are retained here for traceability, and the operative section is the source
of truth. An entry may carry both — a settled core with a narrower question still open.

### \[RDI-COMP\] Comparability Across Publication Status Categories

**Question:** Should RDI results be directly comparable to Available and Preview results on the same
charts?

**Context:** Currently all three categories are plotted together. RDI systems may have unfair
advantages (unavailable hardware, prototype software) that make direct comparison misleading.

**Options under consideration:**

1. Display all categories on the same chart with clear visual distinction (current approach).
2. Display RDI on separate charts.
3. Allow submitters to opt in to co-display with Available/Preview.

### \[CUSTOM-SKU\] Production Hardware Not Available for General Purchase

See [§7.4](#74-open-question-custom-sku-classification-custom-sku).

### \[SERVICED-REQ\] Serviced Division Requirements

**Question:** What additional disclosure and reproducibility requirements apply to the Serviced
division, given that the submitter does not control the underlying stack?

**Context:** Key open items include: what system description fields are required vs. optional for
APIs; how to handle API versioning (the underlying model or serving stack may change without
notice); and whether Serviced results should carry a permanent disclaimer about reproducibility.

### \[RUN-REQ\] Run Requirements Ratification

**Question:** What are the ratified values and rules for minimum run duration, minimum query count,
dataset subset rules, and the warmup data and documentation requirements
([§6.3](#63-warmup-period))?

**Context:** [§6 Run Requirements](#6-run-requirements-per-measurement-point) currently contains
illustrative example values, and the [§6.3](#63-warmup-period) warmup model — submitter discretion
plus mandatory disclosure, in place of a fixed warmup duration — is itself pending ratification. All
constraints in that section are pending working group ratification based on empirical validation
data.

### \[TOK-COUNT\] Reference-Chat-Template Tokenization and Reported Throughput

**Question:** The reference-chat-template tokenization rule
([§2.8 Tokenizer Rules](#28-tokenizer-rules)) may produce token counts that differ from what
individual serving stacks report as "tokens/second" internally. Have MLC stakeholders and submitter
organizations agreed that the published metric will use the client-side reference count and not the
serving-stack-reported count?

**Context:** Resolution is needed before v1.0 publishes side-by-side comparison charts. The current
§2.8 wording—coalesce visible output and reasoning in arrival order, reassemble parallel tool calls
by list index, render the complete structured assistant message with the official reference chat
template, subtract empty assistant framing, and tokenize once—is the proposed rule. The open
question is whether stakeholders accept that the published numbers may differ from internal
serving-stack-reported numbers.

### \[CKPT-RESIDENCY\] Checkpoint Component Residency

**Question:** Must a component present in the canonical checkpoint — for example a
speculative-decoding head such as MTP — be loaded into accelerator memory during measurement, or is
it sufficient that it be present in the submitted checkpoint artifact?

**Resolved — residency is not required.** The checkpoint *artifact* must preserve the full component
set of the canonical checkpoint; unused weights MUST NOT be removed from it. Residency is a separate
question, and the answer is that submitters are **not obligated to load or copy every weight in the
checkpoint to the accelerator** where the component is not needed for the configuration being
measured. The operative rule is in [§2.9.3](#293-model-weight-rules).

| Item | Resolution |
|---|---|
| Stripping unused components from the checkpoint | **Not permitted** ([§2.9.3](#293-model-weight-rules), [§2.9.6.5](#2965-disallowed)). |
| Loading every component into accelerator memory | **Not required.** A submitter may decline to load a component that the measured configuration does not use. |
| Memory footprint | A legitimate configuration dimension, consistent with [§2.9.4](#294-speculative-decoding) already permitting speculative decoding to be disabled at any or all measurement points. |

*Rationale:* the artifact requirement is what preserves provenance and lets a reviewer re-run the
submission with the component enabled; it does not depend on the component occupying accelerator
memory during a run. Requiring residency would force a submitter to reserve HBM for a module the
rules explicitly permit them to disable, which the working group did not consider a meaningful
comparability gain.

**Open — disclosure of the loaded component set.** This resolution permits non-residency without
requiring the loaded component set to be declared per measurement point. For speculative-decoding
heads the choice is already visible through the per-point drafter configuration required by
[§2.9.4](#294-speculative-decoding); for any other auxiliary component it is not. Whether to extend
the [§2.9.6.6](#2966-disclosure) table to cover the general case is left to the working group.

### \[POWER-NORM\] Power Normalization Open Items

**Question:** What remains to be settled in [§4.5](#45-performance-normalization)?

**Open — Tier 1 definition.** The roadmap runs from Tier 3 up to Tier 1, but only Tiers 3 and 2 are
specified ([§4.5.1](#451-power-normalization-roadmap-and-rationale)). The roadmap anticipates true
measured power as an additional normalization option; whether that constitutes Tier 1, and what
evidence and instrumentation it would require, is to be defined in a later version.

**Open — Serviced division normalization.** Power normalization is mandatory for Standardized (CoP
and CoN) and optional for RDI. Normalization options for Serviced — managed endpoints, CSP-hosted
services, and similar — are deferred to a later version. Whether RDI should remain optional or be
brought into line with Standardized is worth confirming.

**Resolved, recorded here for traceability:**

| Item | Resolution |
|---|---|
| Air-cooled overhead fraction | **50%**, as used in [§4.5.2](#452-proposed-endpoints-v10-normalization-methodology). Liquid-cooled remains 30%. |
| Normalized metric | **`system_tps_per_kw`** = total system throughput ÷ `point_power_kw`, evaluated **per measurement point** ([§4.5.3](#453-normalized-metric)). Provisioned power is fixed per system; the denominator scales by the node fraction `Y_s / N_s` engaged at each point, rounded up to whole nodes. Single-node submissions have a constant denominator. |
| Per-point node declaration | Optional `nodes_used` field in the measurement-point YAML ([§8.3](#83-measurement-point-yaml)), by node type. Omitted ⇒ the point is normalized by full provisioned power. |
| Partially provisioned systems | Verified public documentation; rack-level node scaling `P_rack × Y/N` for whole nodes ([§4.5.2.1](#4521-rack-level-node-scaling)); or the MLC formula with component counts limited to what is provisioned. Linear scaling does not apply within a node. |
| Scope | All Standardized CoP and CoN submissions; Serviced deferred; RDI optional. |
| Estimated-power labelling | Results tagged **"MLC Estimated Power"** where values were not supplied by the submitter or arise from comprehensive testing. |
| Descriptor file | `system_power.json` is required for a valid submission, defined field-by-field in [Appendix E](#appendix-e-system_powerjson-field-reference), and checked at automated compliance ([§9.1](#91-automated-checks)). Two schema choices remain open for the working group — per-value sourcing and the node-set array ([E.7](#e7-validation)). |

### \[OFFLINE\] Offline Point Open Items

**Question:** What remains to be settled for the Offline point ([§5.7](#57-offline-point))?

1. **Agentic workloads.** Offline is out of scope for agentic benchmarks in v1.0. Defining it
   requires deciding what "all queries available at once" means for multi-turn trajectories whose
   later turns depend on earlier responses and on tool-call results, and what the reported
   concurrency would be when the query count is not known in advance. To be ratified by the working
   group for a later version.
2. **Dataset cardinality below $C_{max}$.** The Offline concurrency is the cardinality of the
   performance dataset ([§5.7.1](#571-definition)), while
   [§5.7.2](#572-relationship-to-maximum-supported-concurrency) requires `concurrency(Offline)` ≥
   $C_{max}$. A submitter whose declared $C_{max}$ exceeds the dataset cardinality cannot satisfy
   both. The working group should decide whether $C_{max}$ is capped at the dataset cardinality,
   whether the dataset is replayed to reach it, or whether the concurrency constraint is waived in
   that case.
3. **Steady-state windowing.** [§4.4](#44-reporting-basis-steady-state-window) scopes steady-state
   detection to fixed-concurrency points, which excludes a dedicated Offline run — so it reports
   whole-run `total` metrics. That follows from the scope wording rather than being stated outright,
   and is worth making explicit.
4. **Minimum run duration.** [§6.2](#62-minimum-run-duration) applies to the Offline point
   unchanged, but an Offline run ends when the queue drains rather than when a clock expires.
   Whether a separate duration rule is needed is open.
5. **Enforcing the pass boundary.** [§5.7.1](#571-definition) bars reordering across dataset passes.
   The reference client should make pass boundaries explicit in the event log so the constraint is
   checkable after the fact rather than resting on attestation; today it is a manual review item
   ([§9.2](#92-manual-review-focus-areas)).

### Division and Scenario Open Items

| Item | Current Proposal | Status |
|---|---|---|
| Allowed techniques for Standardized CoN | Framework defined, details TBD | TBD |
| Output-tokenizer scoring rules | Client-side reference tokenizer is canonical for official output counts; internal serving tokenizers do not require equivalence or mapping factors for output scoring | Proposed |
| Serviced division audit procedures | Required, details TBD | TBD |
| Caching rules for Serviced division | Not allowed across queries | Proposed |
| Response stream modification rules | Not allowed outside reference API | Proposed |
| Multi-token stream interval (iteration coalescing) | Allowed ([§2.9.7](#297-post-processing-equivalence)) | Proposed |
| Future division for new models/datasets | To be determined by WG | TBD |
| Fabric vs. bus restrictions (Standardized CoN) | Not imposed (borrowed from Network Division) | Proposed |
| Batch/chunk tokenizer variability | Reconstruct the complete structured assistant response, render it with the official reference chat template, exclude empty assistant framing, and apply the reference tokenizer once | Proposed |

---

## Appendix B: Quick-Reference Region Boundary Table

<details>
<summary><strong>Quick-Reference Region Boundaries by $C_{min}$ and $C_{max}$</strong></summary>

Pre-computed region boundaries for common combinations of the lower region boundary ($C_{min}$) and
Maximum Supported Concurrency ($C_{max}$) values using the reference algorithm.

| Max Concurrency <br>($C_{max}$) | Lower Boundary <br>($C_{min}$) | Low Concurrency | Medium Concurrency | High Concurrency | 10% Margin |
|---|---|---|---|---|---|
| 64 | 2 | 3–6 | 7–18 | 19–64 | 65–71 |
| 128 | 2 | 3–7 | 8–27 | 28–128 | 129–141 |
| 256 | 2 | 3–8 | 9–42 | 43–256 | 257–282 |
| 256 | 8 | 9–14 | 15–47 | 48–256 | 257–282 |
| 512 | 8 | 9–16 | 17–71 | 72–512 | 513–564 |
| 1,024 | 8 | 9–18 | 19–109 | 110–1,024 | 1,025–1,127 |
| 512 | 16 | 17–24 | 25–79 | 80–512 | 513–564 |
| 1,024 | 16 | 17–26 | 27–117 | 118–1,024 | 1,025–1,127 |
| 2,048 | 16 | 17–29 | 30–176 | 177–2,048 | 2,049–2,253 |
| 1,024 | 32 | 33–42 | 43–131 | 132–1,024 | 1,025–1,127 |
| 2,048 | 32 | 33–45 | 46–192 | 193–2,048 | 2,049–2,253 |
| 4,096 | 32 | 33–48 | 49–287 | 288–4,096 | 4,097–4,506 |
| 8,192 | 32 | 33–52 | 53–437 | 438–8,192 | 8,193–9,012 |
| 16,384 | 32 | 33–57 | 58–676 | 677–16,384 | 16,385–18,023 |

Boundaries use [§5.5](#55-region-boundary-reference-algorithm) with banker's rounding.
$C_{min}$ selection and Ultra Low coverage follow [§5.4](#54-regions-of-interest).

</details>


---

## Appendix C: Worked Power Normalization Examples

> [!NOTE]
> **All values below are illustrative.** They are chosen to be round, so the arithmetic of
> [§4.5.2](#452-proposed-endpoints-v10-normalization-methodology) is easy to follow. They are not
> vendor specifications and must not be cited as such. A submitter's own figures come from the
> verified sources of §4.5.2.

Recalling the model:

```
Major  = CPU_power + Accelerator_power + Network_scale_up_power
Other  = overhead_fraction × Major          (0.30 liquid-cooled, 0.50 air-cooled)
Total  = Major + Other
```

### C.1 Air-cooled single node, no scale-up switch

A standalone 8-accelerator node. There is no rack-level scale-up fabric, so that term is zero.

| Component | Count | Per-unit | Subtotal |
|---|---|---|---|
| CPU | 2 | 350 W | 700 W |
| Accelerator | 8 | 1,000 W | 8,000 W |
| Scale-up switch | 0 | — | 0 W |
| **Major** | | | **8,700 W** |
| Other (air-cooled, × 0.50) | | | 4,350 W |
| **Total system power** | | | **13,050 W = 13.05 kW** |

If this node reports `system_tps` = 26,100 at a measurement point, then
`system_tps_per_kw` = 26,100 ÷ 13.05 = **2,000.0**.

### C.2 Liquid-cooled rack-scale system

An 18-node rack, each node carrying 2 CPUs and 4 accelerators, with 9 rack-level scale-up switches.

| Component | Count | Per-unit | Subtotal |
|---|---|---|---|
| CPU | 36 | 350 W | 12,600 W |
| Accelerator | 72 | 1,200 W | 86,400 W |
| Scale-up switch | 9 | 2,000 W | 18,000 W |
| **Major** | | | **117,000 W** |
| Other (liquid-cooled, × 0.30) | | | 35,100 W |
| **Total system power** | | | **152,100 W = 152.10 kW** |

### C.3 Per-point normalization across the pareto

**Single node — the denominator is constant.** For the C.1 node at 13.05 kW there is one node, so
`Y = N = 1` at every point and the denominator never moves ([§4.5.3](#453-normalized-metric)):

| Concurrency | `system_tps` | `point_power_kw` | `system_tps_per_kw` |
|---|---|---|---|
| 32 | 1,100 | 13.05 | 84.3 |
| 512 | 5,400 | 13.05 | 413.8 |
| 4,096 | 7,830 | 13.05 | 600.0 |

The normalized curve is the throughput curve scaled by a single constant. A single-node submitter
need not declare `nodes_used` at all.

**Rack-scale — the denominator tracks the nodes engaged.** Take the C.2 rack at 152.10 kW, built from
`N = 18` nodes of 4 accelerators each (72 accelerators). The submitter uses a wider tensor-parallel
replica at low concurrency for latency, and a narrower one at high concurrency for throughput. At
every point `DP` is maximized against the 72 provisioned accelerators
([§4.5.3](#453-normalized-metric)):

| Concurrency | `A_replica` | `DP = floor(72 / A_replica)` | Accelerators engaged | `Y = ceil(acc / 4)` | `point_power_kw = 152.10 × Y/18` | `system_tps` | `system_tps_per_kw` |
|---|---|---|---|---|---|---|---|
| 32 | 16 | 4 | **64** | 16 | 135.20 | 6,000 | 44.4 |
| 512 | 8 | 9 | 72 | 18 | 152.10 | 60,000 | 394.5 |
| 4,096 | 4 | 18 | 72 | 18 | 152.10 | 91,260 | 600.0 |

Only the first point engages less than the whole rack, and not by choice: a 16-accelerator replica
divides into 72 four times with **8 accelerators left over** — two nodes that cannot hold a fifth
replica. Those two nodes are excluded from that point's denominator. The other two points use a
replica size that divides 72 exactly, so `DP` consumes the rack and `Y = N`.

**Why maximal engagement is about the curve, not the ratio.** Had the submitter run that first point
at `DP = 1` on a single 16-accelerator replica, they would have engaged 4 nodes and divided by
`152.10 × 4/18 = 33.80 kW`. Throughput would fall roughly in proportion — about 1,500 `system_tps` —
giving ≈ 44.4 `system_tps_per_kw`, essentially the same ratio. What changes is the **position of the
point on the pareto curve**: 1,500 TPS instead of 6,000, from a system the submission describes as a
72-accelerator rack. Maximal engagement is what stops a submitter publishing a curve measured on a
slice and labelled with the whole machine ([§4.5.3](#453-normalized-metric)).

Had the submitter omitted `nodes_used` entirely, every point would divide by the full 152.10 kW —
giving 39.4 at the first point instead of 44.4. **The conservative figure is the default**; the
declaration is what earns the scaling.

> [!NOTE]
> Scaling applies to *every* term of the power model, including scale-out switch power
> ([§4.5.3](#453-normalized-metric)). The C.2 rack has no scale-out fabric, so the single ratio above
> covers the whole figure; [C.11](#c11-multi-node-ten-dgx-b300-nodes-over-ethernet) is the case where
> a separate switch term is present.

### C.4 Partially populated node — why intra-node scaling is barred

The C.1 chassis with only **4** of its 8 accelerator slots populated. Apply the formula with the
counts limited to what is provisioned
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology), path 3):

| Component | Count | Per-unit | Subtotal |
|---|---|---|---|
| CPU | 2 | 350 W | 700 W |
| Accelerator | 4 | 1,000 W | 4,000 W |
| **Major** | | | **4,700 W** |
| Other (air-cooled, × 0.50) | | | 2,350 W |
| **Total system power** | | | **7,050 W = 7.05 kW** |

Scaling C.1 linearly by accelerator count would instead give 13.05 × 4/8 = **6.53 kW** — about **7%
low**, because the two CPUs and the fixed chassis overhead do not halve when half the accelerators
are removed. This is why [§4.5.2.1](#4521-rack-level-node-scaling) permits linear scaling only at
**node** granularity, never within a node.

### C.5 Partial rack by node scaling

A submitter has published a rack power of **140 kW** for a rack of **N = 18** nodes, and is
submitting **Y = 6** of those nodes:

```
provisioned_power = 140 kW × (6 / 18) = 46.67 kW
```

Two rules apply to the published figure:

- Where the published specification states a **range**, the upper bound is used. A rack published at
  132–140 kW is taken as **140 kW**, not 132.
- The nodes must be otherwise identical. Scaling is by whole nodes only — a partial rack containing
  a partly populated node falls back to C.4's treatment for that node.

**The same scaling runs again, per point.** The 46.67 kW above is this submission's *provisioned*
power: `N = 6` is now what the submission consists of. Each measurement point then scales that figure
by the nodes it engages ([§4.5.3](#453-normalized-metric)) — a point deploying across 2 of the 6
nodes divides by `46.67 × (2 / 6) = 15.56 kW`. The two applications compose; they do not conflict.
See [C.3](#c3-per-point-normalization-across-the-pareto).

### C.6 Declaring provisioned power directly

Where the formula produces a figure higher than the submitter's system is actually rated at, the
submitter may declare provisioned power directly instead. Taking C.2: if the rack is published at
**140 kW** while the component sum gives **152.10 kW**, the submitter may declare 140 kW, subject to
the same verification and publication requirements as every other value.

This lowers the denominator and therefore raises `system_tps_per_kw`, which is precisely why the
declared figure must be publicly verifiable — it is the one path where a submitter's own number
improves their result.

A divergence of this size also triggers the **obligation to report** of
[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology): the submitter tells MLCommons that
the formula overestimated their system, so the formula or the defaults behind it can be corrected. A
submitter may not simply take whichever of the two figures suits them and stay silent — least of all
where the estimate is *lower* than reality, which flatters the result and is a misrepresentation.

### C.7 Estimating switch power when TDP is not published

Where a scale-up switch's power is not disclosed, it may be estimated from bandwidth and energy per
bit ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). Using the Ethernet reference
point of **3.5 kW at 10.8 TB/s per direction**, a switch of half that bandwidth is estimated
proportionally:

```
5.4 TB/s/dir × (3.5 kW ÷ 10.8 TB/s/dir) ≈ 1.75 kW per switch
```

The reference point used, and the source it came from, are declared in `system_power.json` alongside
the resulting figure.


### C.8 Real system: air-cooled NVIDIA DGX B300

An 8-accelerator air-cooled node, with every component figure taken from a public source.

| Component | Count | Per-unit | Source | Subtotal |
|---|---|---|---|---|
| CPU — Intel Xeon 6776P | 2 | 350 W | [Intel](https://www.intel.com/content/www/us/en/products/sku/243691/intel-xeon-6776p-processor-336m-cache-2-30-ghz/specifications.html) | 700 W |
| Accelerator — NVIDIA B300 | 8 | 1,100 W | [NVIDIA Blackwell Ultra datasheet](https://resources.nvidia.com/en-us-blackwell-architecture/blackwell-ultra-datasheet) | 8,800 W |
| Scale-up — NVSwitch, 14.4 TB/s aggregate | 2 | — | [D.1](#d1-scale-up-network), 5 pJ/bit | 576 W |
| **Major** | | | | **10,076 W** |
| Other (air-cooled, × 0.50) | | | | 5,038 W |
| **Formula total** | | | | **15,114 W = 15.11 kW** |

Scale-up power is derived per [C.7](#c7-estimating-switch-power-when-tdp-is-not-published) from the
node's **aggregate** NVLink bandwidth — 14.4 TB/s across both switches, not a per-switch figure:

```
14.4 × 10¹² B/s × 8 = 1.152 × 10¹⁴ bit/s ,  at 5 pJ/bit  →  576 W
```

> [!IMPORTANT]
> **Use the aggregate bandwidth of the fabric being costed, once.** A rack-level aggregate figure —
> NVL72's 130 TB/s, for example — is not a per-switch number and must not be multiplied by the switch
> count. Applying 130 TB/s per switch here yields 10.4 kW of switch power in a node whose entire
> published draw is 14.5 kW, which is self-evidently wrong.

**Published figure, and the override.** NVIDIA publishes the DGX B300 system power as **14.5 kW**
([NVIDIA DGX B300 user guide](https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html)).
The formula's 15.11 kW is **4.2% above** that — the conservative margin the method is designed to
produce. Because the published figure is lower and is itself a verifiable public source, the
submitter may declare it directly under
[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology):

```
provisioned_power_kw = 14.50
```

This is the ordinary case for a system with a published rating: the component sum acts as a sanity
check, and a published figure within a few percent of it is adopted. A component sum that diverges
*materially* from a published rating is a signal that an input is wrong — see the caution above.


### C.9 Real system: liquid-cooled Lenovo GB300 NVL72

A rack-scale liquid-cooled system. Every figure is taken from a public source, and the rack rating is
**Lenovo's** — see the caution below.

| Component | Count | Per-unit | Source | Subtotal |
|---|---|---|---|---|
| CPU — NVIDIA Grace | 36 | 300 W | [Lenovo LP2357](https://lenovopress.lenovo.com/lp2357.pdf?ref=wheresyoured.at); [D.2](#d2-processors) ARM ≤ 128 cores | 10,800 W |
| Accelerator — Blackwell Ultra | 72 | 1,400 W | [NVIDIA Blackwell Ultra datasheet](https://resources.nvidia.com/en-us-blackwell-architecture/blackwell-ultra-datasheet) | 100,800 W |
| Scale-up — 18 × NVSwitch, 130 TB/s aggregate | — | — | [D.1](#d1-scale-up-network), 5 pJ/bit | 5,200 W |
| **Major** | | | | **116,800 W** |
| Other (liquid-cooled, × 0.30) | | | | 35,040 W |
| **Formula total** | | | | **151,840 W = 151.84 kW** |

The scale-up term uses the rack's **aggregate** NVLink bandwidth once, across all 18 switches — not
130 TB/s per switch ([C.8](#c8-real-system-air-cooled-nvidia-dgx-b300)):

```
130 × 10¹² B/s × 8 = 1.04 × 10¹⁵ bit/s ,  at 5 pJ/bit  →  5,200 W   (≈ 289 W per switch)
```

**Published figure, and the override.** Lenovo publishes rack power for this system as **135 kW**
([LP2357](https://lenovopress.lenovo.com/lp2357.pdf?ref=wheresyoured.at)). The formula's 151.84 kW is
**12.5% above** that. The published figure is lower and verifiable, so the submitter declares it:

```
provisioned_power_kw = 135.00
```

> [!IMPORTANT]
> **This 135 kW belongs to Lenovo's system, not to GB300 NVL72 in general.** Another vendor
> submitting a GB300 NVL72 rack must cite a rating published for *their* system, or fall back to the
> component sum. Published provisioned power is a property of a vendor's integration — power
> supplies, cooling loop, redundancy — not of the accelerator or the rack topology
> ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)).

> [!NOTE]
> **CPU count.** This example takes the NVL72 configuration as **36** Grace CPUs (2 per compute tray
> across 18 trays) paired with 72 Blackwell GPUs. If a submitter's configuration differs, the count
> changes the result materially: 72 Grace would give 165.88 kW (+22.9% over published) and 144 would
> give 193.96 kW (+43.7%). A component sum that diverges this far from a published rating is a signal
> that an input is wrong.


### C.10 No published rating: liquid-cooled AMD MI455X Helios rack

A rack-scale liquid-cooled system with **no published rack power figure**. This is the case the
MLCommons defaults exist for, and the one that carries the estimated-power tag.

| Component | Count | Per-unit | Source | Subtotal |
|---|---|---|---|---|
| CPU — AMD Venice, 256 cores | 18 | 500 W | [D.2](#d2-processors) default, x86 > 64 cores | 9,000 W |
| Accelerator — AMD MI455X | 72 | 2,500 W | [D.3](#d3-accelerators) default | 180,000 W |
| Scale-up — switch trays | 6 | 7,000 W | [AMD MI400 System Architecture, Hot Chips 2026](https://hc2026.hotchips.org/assets/program/conference/day1/FINAL_AMD%20MI400_System_Arch_Hot_Chips_2026.pdf) | 42,000 W |
| **Major** | | | | **231,000 W** |
| Other (liquid-cooled, × 0.30) | | | | 69,300 W |
| **Total system power** | | | | **300,300 W = 300.30 kW** |

```
provisioned_power_kw = 300.30
```

There is no published rack rating to declare instead, so the component sum stands as the provisioned
power. This is the ordinary outcome where a vendor has not published a system-level figure.

> [!IMPORTANT]
> **An expected figure is not a usable one.** AMD's Helios rack is widely expected to be around
> **245 kW**, which would put the component sum **22.6% above** it. No official AMD reference for that
> figure could be found, so under [§4.5.2](#452-proposed-endpoints-v10-normalization-methodology) it
> is an *unverified source* and cannot be declared as provisioned power — the 300.30 kW component sum
> governs regardless.
>
> This is the incentive the methodology is built on, seen from the submitter's side: a vendor who has
> not published a system-level rating is normalized against a deliberately conservative estimate, and
> the only remedy is to publish. Should AMD publish 245 kW, a submitter may declare it and the
> estimated-power tag falls away for that component.

**Two of the three inputs are MLCommons defaults.** The Venice TDP is not published, so
[D.2](#d2-processors)'s x86 default for more than 64 cores applies; the MI455X figure comes from
[D.3](#d3-accelerators). Only the switch-tray power is submitter-evidenced, from a public AMD
presentation. Because defaults were used, the published result is tagged **"MLC Estimated Power"**
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). Supplying verifiable public
figures for the CPU and accelerator would remove the tag and replace the defaults.

> [!NOTE]
> The 7 kW switch-tray figure is consistent with [D.1](#d1-scale-up-network)'s Ethernet reference of
> 3.5 kW per switch — two switches per tray. Where a submitter has a directly published tray or
> switch power, as here, that figure is used and the pJ/bit path of
> [C.7](#c7-estimating-switch-power-when-tdp-is-not-published) is not needed.


### C.11 Multi-node: ten DGX B300 nodes over Ethernet

Ten of the [C.8](#c8-real-system-air-cooled-nvidia-dgx-b300) nodes, joined by an Ethernet scale-out
fabric. This is the only example with a scale-out fabric, and so the only one where
`Scale_out_switch_power` is non-zero.

**Step 1 — node power.** Each node uses its published figure of **14.5 kW** (C.8):

```
10 × 14.5 kW = 145.00 kW
```

That figure already includes each node's own cooling and power-supply overhead, so the overhead
fraction is **not** applied to it again
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)).

**Step 2 — scale-out bandwidth.** Each DGX B300 carries eight 800 Gb/s NICs:

```
10 nodes × 8 NICs × 800 Gb/s = 64 Tb/s
```

**Step 3 — NICs.** Node power here comes from a **published** figure, so the NICs are assumed to be
inside the 14.5 kW and are **not counted again**
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)).

Had the nodes instead been built with the MLC formula, the NICs would have to be added, since the
formula has no NIC term. At the [D.4](#d4-reference-scale-out-network) reference of 75 W each they are
a major component and take the overhead, so each node becomes
`(700 + 8,800 + 576 + 8 × 75) × 1.50 = 16.01 kW`, and the cluster
`10 × 16.01 + 1.80 = 161.94 kW` — against 146.80 kW on the published path. Which path was taken
changes what must be counted, not merely the arithmetic.

**Step 4 — rack switches.** 64 Tb/s is met by **two SN5610** switches at 51.2 Tb/s each
([D.4](#d4-reference-scale-out-network)), a combined 102.4 Tb/s. Power depends on the declared
cabling. Published switch power is wall power, so no overhead multiplier is applied:

| Declared cabling | Per switch | 2 switches | Cluster total |
|---|---|---|---|
| **Passive copper** | 900 W | 1,800 W | **146.80 kW** |
| **Active optical** | 2.08 kW | 4,160 W | **149.16 kW** |

```
System Power = 145.00 kW + 1.80 kW = 146.80 kW      (passive)
             = 145.00 kW + 4.16 kW = 149.16 kW      (active optical)
```

**The cabling declaration is worth 2.36 kW**, about 1.6% of the cluster — which is why
[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology) requires the submitter to declare it
rather than leaving it to be assumed. Taking the passive figure:

```
provisioned_power_kw = 146.80
```

> [!NOTE]
> A single **SN6810-LD** at 102.4 Tb/s would also satisfy the 64 Tb/s requirement, at 1.96 kW typical
> — giving `145.00 + 1.96 = 146.96 kW`. A submitter selects reference switches that meet the
> bandwidth requirement; where the switches actually deployed are known, those are declared instead.


---

## Appendix D: MLCommons Default Power Reference Values

These are the MLCommons **default reference values** used where a submitter does not supply a
publicly verifiable figure for a component. A default is its own reference and needs no further
citation in `system_power.json`.

A submitter's own **verifiable public reference overrides** the default for that component
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). Self-declaration does not:
a figure asserted without a public source is treated as absent, the default applies, and the result
is tagged "MLC Estimated Power".

**Update cadence.** MLCommons revisits these values **at least once per quarter**, and may update any
of them sooner as soon as higher-quality information becomes available — a vendor publishing a
specification, a correction reported under the obligation in
[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology), or better industry analysis. Each
revision is published with the version of the reference list it belongs to, so that a submission can
be read against the values in force when it was made.

### D.1 Scale-up network

| Interconnect | Default reference | Source |
|---|---|---|
| **NVLink-based** | **5 pJ/bit** | [IEEE ComSoc CTN — *Data Deluge: Powering and Straining AI*](https://www.comsoc.org/publications/ctn/data-deluge-powering-and-straining-ai) |
| **Ethernet-based** | **3.5 kW per switch at 10.8 TB/s per direction** | [AMD MI400 System Architecture, Hot Chips 2026](https://hc2026.hotchips.org/assets/program/conference/day1/FINAL_AMD%20MI400_System_Arch_Hot_Chips_2026.pdf) |

The two defaults are expressed in different forms because they are applied differently. The NVLink
default is an **energy-per-bit** figure, applied as `bandwidth × 5 pJ/bit`. The Ethernet default is
a
**power-at-a-stated-bandwidth** point, applied proportionally to the switch's actual bandwidth.

### D.2 Processors

| Architecture | Core count | Default TDP | Source |
|---|---|---|---|
| **x86** | ≤ 64 | **350 W** | [Intel Xeon 696x](https://www.intel.com/content/www/us/en/products/sku/243860/intel-xeon-696x-processor-336m-cache-2-40-ghz/specifications.html) |
| **x86** | > 64 | **500 W** | [Intel Xeon 6980P](https://www.intel.com/content/www/us/en/products/sku/240777/intel-xeon-6980p-processor-504m-cache-2-00-ghz/specifications.html) |
| **ARM** | ≤ 128 | **300 W** | [Arm AGI CPU product brief](https://www.arm.com/static/az/pdf/product-brief/arm-agi-cpu-product-brief.pdf) |
| **ARM** | > 128 | *not yet defined* | — |

> [!NOTE]
> **[WG Open Item]** — No default exists for ARM CPUs above 128 cores. Until one is set, such a CPU
> has no fallback and the submitter must supply a verifiable public figure.

### D.3 Accelerators

| Accelerator | Default TDP |
|---|---|
| NVIDIA B300 | 1,100 W |
| NVIDIA B200 | 1,000 W |
| NVIDIA GB300 | 1,400 W |
| NVIDIA GB200 | 1,200 W |
| AMD MI455X | 2,500 W |
| AMD MI355X | 1,400 W |
| AMD MI350X | 1,000 W |
| Google TPU v7 | 1,000 W |
| AWS Trainium Tn3 | 700 W |

Accelerators not listed have no default; MLCommons estimates them from industry analysis under
[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology), and a submitter may always supply a
verifiable public figure instead.


### D.4 Reference scale-out network

Where a submitter does not declare the rack switches actually used, these reference switches may be
selected to meet the fabric's bandwidth requirement ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)).
Power depends on cabling, and the submitter must declare which applies.

| Switch | Bandwidth | Passive cables | Active optical | Source |
|---|---|---|---|---|
| **SN4700** | 12.8 Tb/s | **630 W** (ATIS typical) | *not published* | [NVIDIA SN4000 specifications](https://networking-docs.nvidia.com/sn4000hw/specifications) |
| **SN5400** | 25.6 Tb/s | **670 W** (ATIS typical) | *not published* | [NVIDIA SN5000 specifications](https://networking-docs.nvidia.com/sn5000hw/specifications) |
| **SN5610** | 51.2 Tb/s | **900 W** (ATIS typical) | **2.08 kW** (64 optical modules) | [NVIDIA SN5000 specifications](https://networking-docs.nvidia.com/sn5000hw/specifications#SN5610-Specifications) |
| **SN6810-LD** | 102.4 Tb/s | **1.96 kW** typical (2.2 kW max) | *not differentiated* | [NVIDIA SN6000 specifications](https://networking-docs.nvidia.com/sn6000hw/1.3/specifications) |

Figures are the vendor's **typical** power. Where a switch has no published figure for the declared
cabling, it cannot serve as a reference for that case and the submitter must declare the switch
actually used.

**Scale-out NICs.**

| Component | Default | Basis |
|---|---|---|
| Scale-out NIC | **75 W** | A PCIe network adapter |

This default applies only where scale-out NIC power is counted at all — that is, where node power was
built with the MLC formula rather than taken from a published specification
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). A submitter may declare the rated
TDP of the adapter actually fitted instead.



---

## Appendix E: `system_power.json` Field Reference

This appendix is the normative definition of the `system_power.json` descriptor required by
[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology) and located per
[§8.1](#81-directory-structure). It is what the automated **Power descriptor** check in
[§9.1](#91-automated-checks) validates against.

The descriptor records **how provisioned power was established**, not how the system behaved during
a
run. It is written once per system, and the provisioned power it yields is the same for every
measurement point. What varies per point is how much of that system a point *engages*: each point's
denominator is this file's figure scaled by the nodes declared in that point's `nodes_used`
([§4.5.3](#453-normalized-metric), [§8.3](#83-measurement-point-yaml)). This file supplies the `P_s`
and `N_s` that scaling divides into; it does not itself vary by point.

### E.1 Sourced values

[§4.5.2](#452-proposed-endpoints-v10-normalization-methodology) requires **every** power value to
carry a public, verifiable reference. Every such value is therefore written as a three-key object
rather than a bare number:

```json
{ "value_w": 1100, "source_type": "vendor_spec", "source": "https://..." }
```

| Key | Type | Description |
|---|---|---|
| `value_w` | number | The value, in **watts**. Values naturally expressed in kilowatts use `value_kw` instead; no field carries both. |
| `source_type` | string | One of `vendor_spec`, `publication`, `public_statement`, `mlc_default`. The first three are the verifiable categories of [§4.5.2](#452-proposed-endpoints-v10-normalization-methodology); a submitter assertion with no public source is **not** a source type and is rejected. |
| `source` | string | A resolvable URL for the first three types. For `mlc_default`, the Appendix D subsection the value came from — `"D.1"`, `"D.2"`, `"D.3"`, `"D.4"` — which is its own reference and needs no URL. |

Any value whose `source_type` is `mlc_default`, or which is left absent and auto-populated by the
checker, causes the submission to be tagged **"MLC Estimated Power"** — **where that value
contributes to `provisioned_power_kw`**. A default that reaches only a superseded cross-check does
not trigger the tag: where `declared_provisioned_power` governs, the tag follows the sourcing of the
declared figure alone, and the component block beneath it may hold defaults without consequence
([E.6.2](#e62-component-sum-with-a-published-override)).

### E.2 Top-level fields

| Field | Type | Required | Description |
|---|---|---|---|
| `system_desc_id` | string | yes | The `system_desc.json` this descriptor belongs to ([§8.2](#82-system-description-system_descjson)). |
| `cooling` | string | yes | `liquid` or `air`. Selects the overhead fraction — 0.30 or 0.50 ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). MUST be consistent with the `cooling` field of `system_desc.json`. |
| `node_sets` | array | yes | One entry per **homogeneous set of identical nodes** ([E.3](#e3-node-sets)). A single-node or uniform-rack submission has exactly one entry. |
| `scale_out` | object | yes | The scale-out fabric ([E.4](#e4-scale-out-fabric)). For a single-node submission, or where nodes are joined only by the scale-up fabric, this is `{ "present": false }`. |
| `declared_provisioned_power` | sourced value (`value_kw`) | no | A directly declared provisioned power figure, overriding the computed total ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). Where a published specification states a range, the **upper bound** is declared. |
| `computed` | object | checker | The derived arithmetic ([E.5](#e5-computed-block)). A submitter may populate it; the checker recomputes and the checker's values govern. |
| `provisioned_power_kw` | number | checker | The system's full provisioned power: `declared_provisioned_power` where present, otherwise `computed.total_system_power_w / 1000`. Two decimal places. This is the figure each point's `point_power_kw` is scaled from, and is the denominator in full for any point not declaring `nodes_used` ([§4.5.3](#453-normalized-metric)). |
| `mlc_estimated_power` | boolean | checker | Set by the checker where any value was defaulted or auto-populated. A submitter-supplied value is ignored. |
| `notes` | string | no | Free-form submitter notes. Not a substitute for a `source`. |

### E.3 Node sets

Each entry of `node_sets` describes one set of **identical** nodes. A partially populated node, or a
node under a TDP cap, is not identical to a fully populated or uncapped one and belongs to its own
set ([§4.5.2.1](#4521-rack-level-node-scaling)).

| Field | Type | Required | Description |
|---|---|---|---|
| `node_set_id` | integer | yes | Identifier, unique within the file. |
| `system_node_ensemble_id` | integer | yes | The matching node type in `system_desc.json` ([§8.2](#82-system-description-system_descjson)). |
| `nodes_provisioned` | integer | yes | `N_s` — the number of nodes of this set the submission provisions, **rounded up to a whole node** ([§4.5.2.1](#4521-rack-level-node-scaling)). This is the denominator of the per-point node fraction; a point's `nodes_used` for this set MUST NOT exceed it ([§4.5.3](#453-normalized-metric)). |
| `power_method` | string | yes | How this set's power was established: `component_sum`, `published_system`, or `node_scaling`. |
| `published_power` | sourced value | conditional | Required for `published_system` and `node_scaling`. The published figure for **one node** (`published_system`) or for the **whole rack** (`node_scaling`). A figure published by another vendor for a comparable system MUST NOT be used ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). |
| `nodes_in_published_rack` | integer | conditional | `N` — required for `node_scaling` only; the node count the published rack figure covers. |
| `components` | object | conditional | Required for `component_sum` ([E.3.1](#e31-components)). Optional otherwise, where a submitter wishes to record the component sum as a cross-check. |

#### E.3.1 Components

| Field | Type | Description |
|---|---|---|
| `cpu.model` | string | Model name, matching `system_desc.json`. |
| `cpu.count_per_node` | integer | CPUs actually provisioned, not the maximum the chassis holds. |
| `cpu.tdp_per_unit` | sourced value | Per-CPU TDP. Defaults by architecture and core count in [D.2](#d2-processors). |
| `accelerator.model` | string | Model name, matching `system_desc.json`. |
| `accelerator.count_per_node` | integer | Accelerators actually provisioned. Also the divisor for the per-point node rounding `Y_s = ceil(accelerators_used_s / accelerators_per_node_s)` ([§4.5.3](#453-normalized-metric)). |
| `accelerator.tdp_per_unit` | sourced value | Per-accelerator TDP. Defaults in [D.3](#d3-accelerators). |
| `accelerator.below_rated_tdp` | object | Present only where the accelerator runs below its rated TDP. Carries `rated_tdp_w`, the sourced alternative rating, and `evidence` — a description of the reproducible third-party check, or the public listing of the reduced mode as a production operating point ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). |
| `combined_cpu_accelerator` | sourced value | Alternative to the two blocks above where the vendor publishes a single combined figure ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). Mutually exclusive with `cpu.tdp_per_unit` and `accelerator.tdp_per_unit`. |
| `scale_up_network.method` | string | `declared_tdp`, `bandwidth_estimate`, or `none` (systems with no scale-up switch). |
| `scale_up_network.switch_count` | integer | Number of scale-up switches. |
| `scale_up_network.tdp_per_switch` | sourced value | Required for `declared_tdp`. |
| `scale_up_network.aggregate_bandwidth_tbps` | number | Required for `bandwidth_estimate`. The **aggregate** bandwidth of the fabric being costed, counted once — not a per-switch figure multiplied by the switch count ([C.8](#c8-real-system-air-cooled-nvidia-dgx-b300)). |
| `scale_up_network.energy_per_bit_pj` | sourced value (`value_pj`) | Required for `bandwidth_estimate`. Reference points in [D.1](#d1-scale-up-network). The reference point used and its source are declared here ([C.7](#c7-estimating-switch-power-when-tdp-is-not-published)). |

### E.4 Scale-out fabric

| Field | Type | Required | Description |
|---|---|---|---|
| `present` | boolean | yes | `false` for a single-node submission, or where the nodes are joined only by a fabric already counted in `scale_up_network`. All other fields are omitted when `false`. |
| `cabling` | string | yes | `passive` or `active_optical`. **A mandatory declaration** — an active-optical deployment can draw more than twice the power of the same switch with passive cables ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). |
| `required_bandwidth_tbps` | number | yes | The sum of per-node NIC bandwidth across the submitted nodes. Determines the switch count. |
| `nics.count_per_node` | integer | yes | Scale-out adapters per node. |
| `nics.bandwidth_per_nic_gbps` | number | yes | Per-adapter line rate, from which `required_bandwidth_tbps` follows. |
| `nics.counted` | boolean | yes | `true` where node power was built with the MLC formula — the formula has no NIC term, so the adapters MUST be counted. `false` where node power came from a published specification, which is assumed to include them ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). |
| `nics.tdp_per_nic` | sourced value | conditional | Required where `counted` is `true`. Reference value in [D.4](#d4-reference-scale-out-network). |
| `nics.excluded_from_published_power` | string | no | Where `counted` is `true` **despite** a published node figure, the evidence that the published figure excluded the adapters. |
| `switches` | array | yes | One entry per switch model: `model`, `count`, `bandwidth_tbps`, and `power_per_switch` as a sourced value. The combined bandwidth MUST meet `required_bandwidth_tbps`. Where the switches actually deployed are known they are declared; otherwise reference switches from [D.4](#d4-reference-scale-out-network) are selected. |

Scale-out **NICs** are major components and take the overhead fraction. Scale-out **switch** power is
wall power and is added outside the overhead base
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)).

### E.5 Computed block

| Field | Description |
|---|---|
| `major_components_w` | `Σ over node sets: nodes_provisioned × (cpu + accelerator + scale_up)` `+ scale-out NIC power where counted`. Zero for sets on the `published_system` or `node_scaling` path. |
| `overhead_fraction` | `0.30` liquid, `0.50` air — from `cooling`. |
| `other_components_w` | `overhead_fraction × major_components_w`. |
| `published_node_power_w` | `Σ over node sets on a published path: nodes_provisioned × published node power`, with `node_scaling` sets contributing `published_power × (nodes_provisioned / nodes_in_published_rack)`. The overhead fraction is **not** applied to this term ([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)). |
| `scale_out_switch_power_w` | `Σ switches: count × power_per_switch`. Outside the overhead base. |
| `total_system_power_w` | `major_components_w + other_components_w + published_node_power_w + scale_out_switch_power_w`. |

Rounding is applied once, to `provisioned_power_kw` (two decimal places) and to `system_tps_per_kw`
(one decimal place). Intermediate sums are not rounded
([§4.5.2](#452-proposed-endpoints-v10-normalization-methodology)).

### E.6 Worked descriptors

#### E.6.1 Component sum, no published rating

The AMD MI455X Helios rack of [C.10](#c10-no-published-rating-liquid-cooled-amd-mi455x-helios-rack).
Every component value falls back to an Appendix D default, so the result carries the estimated-power
tag.

```json
{
  "system_desc_id": "helios_mi455x",
  "cooling": "liquid",
  "node_sets": [
    {
      "node_set_id": 0,
      "system_node_ensemble_id": 0,
      "nodes_provisioned": 1,
      "power_method": "component_sum",
      "components": {
        "cpu": {
          "model": "AMD Venice 256-core",
          "count_per_node": 18,
          "tdp_per_unit": { "value_w": 500, "source_type": "mlc_default", "source": "D.2" }
        },
        "accelerator": {
          "model": "AMD Instinct MI455X",
          "count_per_node": 72,
          "tdp_per_unit": { "value_w": 2500, "source_type": "mlc_default", "source": "D.3" }
        },
        "scale_up_network": {
          "method": "declared_tdp",
          "switch_count": 6,
          "tdp_per_switch": {
            "value_w": 7000,
            "source_type": "publication",
            "source": "https://hc2026.hotchips.org/assets/program/conference/day1/FINAL_AMD%20MI400_System_Arch_Hot_Chips_2026.pdf"
          }
        }
      }
    }
  ],
  "scale_out": { "present": false },
  "computed": {
    "major_components_w": 231000,
    "overhead_fraction": 0.30,
    "other_components_w": 69300,
    "published_node_power_w": 0,
    "scale_out_switch_power_w": 0,
    "total_system_power_w": 300300
  },
  "provisioned_power_kw": 300.30,
  "mlc_estimated_power": true
}
```

The rack is declared as a single node set of one, because the Helios rack is submitted as one
indivisible system; `nodes_provisioned` counts submitted units of the granularity being costed.

#### E.6.2 Component sum with a published override

The NVIDIA DGX B300 node of [C.8](#c8-real-system-air-cooled-nvidia-dgx-b300).
The component sum is 15.11 kW; the published figure of 14.5 kW is lower and verifiable, so it is
declared and governs. The component block is retained as the cross-check but is optional.

```json
{
  "system_desc_id": "dgx_b300_1node",
  "cooling": "air",
  "node_sets": [
    {
      "node_set_id": 0,
      "system_node_ensemble_id": 0,
      "nodes_provisioned": 1,
      "power_method": "component_sum",
      "components": {
        "cpu": {
          "model": "Intel Xeon 6776P",
          "count_per_node": 2,
          "tdp_per_unit": {
            "value_w": 350,
            "source_type": "vendor_spec",
            "source": "https://www.intel.com/content/www/us/en/products/sku/243691/intel-xeon-6776p-processor-336m-cache-2-30-ghz/specifications.html"
          }
        },
        "accelerator": {
          "model": "NVIDIA B300",
          "count_per_node": 8,
          "tdp_per_unit": {
            "value_w": 1100,
            "source_type": "vendor_spec",
            "source": "https://resources.nvidia.com/en-us-blackwell-architecture/blackwell-ultra-datasheet"
          }
        },
        "scale_up_network": {
          "method": "bandwidth_estimate",
          "switch_count": 2,
          "aggregate_bandwidth_tbps": 14.4,
          "energy_per_bit_pj": { "value_pj": 5, "source_type": "mlc_default", "source": "D.1" }
        }
      }
    }
  ],
  "scale_out": { "present": false },
  "declared_provisioned_power": {
    "value_kw": 14.50,
    "source_type": "vendor_spec",
    "source": "https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html"
  },
  "computed": {
    "major_components_w": 10076,
    "overhead_fraction": 0.50,
    "other_components_w": 5038,
    "published_node_power_w": 0,
    "scale_out_switch_power_w": 0,
    "total_system_power_w": 15114
  },
  "provisioned_power_kw": 14.50,
  "mlc_estimated_power": false
}
```

Note that the component block is retained as the cross-check but is optional. Submitters may leave
it as NULL instead. Eg:

```json
{
  "system_desc_id": "dgx_b300_1node",
  "cooling": "air",
  "node_sets": [
    {
      "node_set_id": 0,
      "system_node_ensemble_id": 0,
      "nodes_provisioned": 1,
      "power_method": "component_sum",
      "components": {
        "cpu": {
          "model": "Intel Xeon 6776P",
          "count_per_node": NULL,
          "tdp_per_unit": {
            "value_w": NULL,
            "source_type": "vendor_spec",
            "source": NULL
          }
        },
        "accelerator": {
          "model": "NVIDIA B300",
          "count_per_node": 8,
          "tdp_per_unit": {
            "value_w": NULL,
            "source_type": "vendor_spec",
            "source": NULL
          }
        },
        "scale_up_network": {
          "method": "declared",
          "switch_count": NULL,
          "aggregate_bandwidth_tbps": NULL,
          "energy_per_bit_pj": { "value_pj": 5, "source_type": "mlc_default", "source": "D.1" }
        }
      }
    }
  ],
  "scale_out": { "present": false },
  "declared_provisioned_power": {
    "value_kw": 14.50,
    "source_type": "vendor_spec",
    "source": "https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html"
  },
  "computed": {
    "major_components_w": 0,
    "overhead_fraction": 0.50,
    "other_components_w": 0,
    "published_node_power_w": ,
    "scale_out_switch_power_w": 0,
    "total_system_power_w": 0
  },
  "provisioned_power_kw": 14.50,
  "mlc_estimated_power": false
}
```


`aggregate_bandwidth_tbps` is the fabric's aggregate across both switches, counted once. It is **not**
multiplied by `switch_count` ([C.8](#c8-real-system-air-cooled-nvidia-dgx-b300)).

#### E.6.3 Multi-node, published node power plus scale-out

The ten-node DGX B300 cluster of [C.11](#c11-multi-node-ten-dgx-b300-nodes-over-ethernet).
Node power comes from a published figure, so the NICs are inside it and `nics.counted` is `false`.
Switch power is wall power and is added outside the overhead base.

```json
{
  "system_desc_id": "dgx_b300_10node_eth",
  "cooling": "air",
  "node_sets": [
    {
      "node_set_id": 0,
      "system_node_ensemble_id": 0,
      "nodes_provisioned": 10,
      "power_method": "published_system",
      "published_power": {
        "value_kw": 14.50,
        "source_type": "vendor_spec",
        "source": "https://docs.nvidia.com/dgx/dgxb300-user-guide/introduction-to-dgxb300.html"
      }
    }
  ],
  "scale_out": {
    "present": true,
    "cabling": "passive",
    "required_bandwidth_tbps": 64.0,
    "nics": {
      "count_per_node": 8,
      "bandwidth_per_nic_gbps": 800,
      "counted": false
    },
    "switches": [
      {
        "model": "NVIDIA Spectrum SN5610",
        "count": 2,
        "bandwidth_tbps": 51.2,
        "power_per_switch": { "value_w": 900, "source_type": "mlc_default", "source": "D.4" }
      }
    ]
  },
  "computed": {
    "major_components_w": 0,
    "overhead_fraction": 0.50,
    "other_components_w": 0,
    "published_node_power_w": 145000,
    "scale_out_switch_power_w": 1800,
    "total_system_power_w": 146800
  },
  "provisioned_power_kw": 146.80,
  "mlc_estimated_power": true
}
```

The switch power is an Appendix D reference rather than the switch actually deployed, which is what
sets `mlc_estimated_power`. Declaring `"cabling": "active_optical"` instead would select the 2.08 kW
figure from [D.4](#d4-reference-scale-out-network) and give 149.16 kW.

#### E.6.4 Heterogeneous rack by node scaling

A rack holding two kinds of node is partitioned into homogeneous sets, each scaled from its own
published rack figure. Only the `node_sets` array is shown.

```json
"node_sets": [
  {
    "node_set_id": 0,
    "system_node_ensemble_id": 0,
    "nodes_provisioned": 5,
    "nodes_in_published_rack": 8,
    "power_method": "node_scaling",
    "published_power": { "value_kw": 120.00, "source_type": "vendor_spec", "source": "https://..." }
  },
  {
    "node_set_id": 1,
    "system_node_ensemble_id": 1,
    "nodes_provisioned": 3,
    "nodes_in_published_rack": 8,
    "power_method": "node_scaling",
    "published_power": { "value_kw": 96.00, "source_type": "vendor_spec", "source": "https://..." }
  }
]
```

This contributes `120.00 × (5/8) + 96.00 × (3/8) = 75.00 + 36.00 = 111.00 kW` to
`published_node_power_w`. A published *whole-rack* figure covering both kinds of node cannot be
scaled directly, because it does not decompose into equal per-node shares
([§4.5.2.1](#4521-rack-level-node-scaling)).

### E.7 Validation

The [§9.1](#91-automated-checks) **Power descriptor** check rejects a submission where:

- the file is absent for any system, or is not valid JSON;
- a required field of [E.2](#e2-top-level-fields), [E.3](#e3-node-sets) or
  [E.4](#e4-scale-out-fabric) is missing;
- any object carrying `value_w`, `value_kw` or `value_pj` lacks a `source_type` and `source`, or
  gives a `source_type` outside the four permitted values;
- `cooling` disagrees with the `cooling` field of the corresponding `system_desc.json`;
- `nodes_provisioned` is not a positive integer, or `nodes_in_published_rack` is absent, or is not
  greater than `nodes_provisioned`, for a `node_scaling` set;
- `scale_out.present` is `true` and the combined `bandwidth_tbps × count` across `switches` is below
  `required_bandwidth_tbps`;
- `scale_out.cabling` is absent while `scale_out.present` is `true`;
- the submitter-supplied `computed` block disagrees with the checker's recomputation.

The last case is a rejection rather than a silent correction: a disagreement means either the
descriptor or the formula is being read differently by the two parties, and that must be resolved
before the result is normalized.

> [!NOTE]
> **[WG Open Item] — two structural choices in this schema need working-group confirmation before
> v1.0.**
>
> 1. **References are carried per value** ([E.1](#e1-sourced-values)), as a three-key object on every
>    power figure, rather than as a single citation list at the top of the file. Per-value sourcing
>    is
>    what makes the [§9.1](#91-automated-checks) check mechanical — a checker can assert that every
>    value has a source — at the cost of some verbosity.
> 2. **Heterogeneity is expressed as an array of node sets** ([E.3](#e3-node-sets)) rather than a flat
>    per-system field list, so that a rack of mixed node types, or a mix of fully and partially
>    populated nodes, can be described at all. A flat schema cannot express
>    [§4.5.2.1](#4521-rack-level-node-scaling)'s per-set scaling.
>
> Both are recommendations, not settled policy.
