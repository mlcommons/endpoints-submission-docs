# Model equivalence (Standardized division)

If you are submitting Standardized, this page governs every optimisation decision you are about to
make. If you are submitting Serviced, augmentation is allowed with disclosure; RDI allows it without
that requirement. For both, this page is background only.

--8<-- "precedence-notice.md"

The rules themselves are in [§2.9 of the rules][rules-2.9], with the disallowed list in
[§2.2.1][rules-2.2.1]. This page tells you which part to read for which decision, and adds what the
checker does.

!!! warning "Check your assumptions here"
    Endpoints inherits the MLPerf Inference model-equivalence rules, then deliberately diverges in
    three places: cross-query KV reuse, dynamic approximate sparsity, and quantization of a
    speculative-decoding drafter. If you've submitted to MLPerf Inference before, recheck your
    assumptions on all three.

## The framing: disallowed-only

Anything not on the disallowed list, and not in conflict with §2.9, is permitted
([§2.2.1][rules-2.2.1], including the note on why). You don't need permission for a technique that
isn't banned, but you do need to pass the accuracy gate and meet the disclosure requirements.

## Parameters the benchmark fixes

Sampling parameters and anything else the reference implementation marks as fixed must not be
changed ([§2.9.1][rules-2.9.1]). This was made explicit in 2026-09. It doesn't narrow what was
already allowed; it closes the reading that a parameter not otherwise mentioned in the rules was
yours to tune.

## Always disallowed

The full list is under *Disallowed optimizations* in [§2.2.1][rules-2.2.1]. The entries that catch
people out are response caching (distinct from KV reuse, which is allowed), input-based
optimization, client-side dispatch manipulation, and speculative decoding with a drafter that isn't
on the benchmark's approved list ([below](#speculative-decoding)).

## Weights

Start from the canonical weights. Post-training quantization is the permitted transformation, under
four conditions; fine-tuning, pruning and distillation are not ([§2.9.3][rules-2.9.3]).

### The checkpoint must stay complete

!!! danger "You may not strip components from the derived checkpoint"
    A derived checkpoint must keep every component the reference checkpoint ships, MTP and
    EAGLE-style heads included, even ones you don't use ([§2.9.3][rules-2.9.3]). Not using a
    component at run time is fine; removing it from the artifact is not.

!!! question "Residency is undecided"
    Whether a component present in the checkpoint must also be **resident in accelerator memory**
    during measurement is an open working-group item, and the freed memory converts directly into
    KV-cache capacity. Tracked upstream as [`[CKPT-RESIDENCY]`][rules-ckpt-residency], and here
    under *Likely to affect your optimisation choices* in [Open
    questions](../help/open-questions.md).

## Speculative decoding

Allowed only with a drafter on your benchmark's approved list. The list, how drafters get on it,
what disqualifies one, what you can't do with an approved drafter, and the per-point disclosure are
all in [§2.9.4][rules-2.9.4]. Tree-structured verification is covered in Q6 of
[§2.9.9][rules-2.9.9].

!!! warning "No list has been published yet"
    As of 2026-09-24 there is no approved-drafter list for any benchmark. The checker ships an empty
    one and reads that as "not permitted anywhere", which is also what the rules say. Until a list
    is published, run without speculative decoding. Tracked as **C10** in [Open
    questions](../help/open-questions.md).

What the checker enforces:

- **`approved-drafter`**: the drafter you declare must match a list entry.
- **`drafter-approval-lead-time`**: a newly approved drafter is usable only from two cohorts after
  the list update, counted against your `target_cohort`.

Both reject the affected **points**, not the whole submission ([§9.1][rules-9.1]). Across the curve
it's one drafter; its configuration can vary per point, including switching speculation off. The
drafter's details go in the `speculative_decoding` block of `point.yaml` ([§8.3][rules-8.3]).

## KV cache: the big difference

!!! success "Cross-query KV reuse is permitted in Endpoints"
    Sharing KV state across requests, including prefix caching of the shared system prompt, is
    allowed. MLPerf Inference prohibits it ([§2.9.5][rules-2.9.5]).

A per-query salt between system prompt and user context is what keeps this sound; its requirements
are in [§2.9.5.1][rules-2.9.5.1]. Accuracy runs use the un-salted dataset, and you don't have to
switch KV reuse off for them. Reduced-precision KV and non-reference compression have to be
disclosed.

!!! danger "Pre-tokenizing clients must salt the token stream"
    If your client sends `input_tokens`, the salt has to be in the token stream the system receives.
    Salting only a `prompt` text field is non-compliant. The two conforming approaches are under
    *Clients that pre-tokenize* in [§2.9.5.1][rules-2.9.5.1].

!!! note "Agentic benchmarks salt differently"
    They use benchmark-specific salting switched on by benchmark-specific flags, which **must be
    enabled** ([§2.9.5.1][rules-2.9.5.1]). Which flags, for which benchmarks, is not yet published —
    see [Open questions](../help/open-questions.md).

!!! warning "Not portable to MLPerf Inference"
    Code paths that rely on cross-query KV reuse won't carry over to an MLPerf Inference submission
    without disabling it.

## Sparsity and approximate computation

The test is whether a technique **changes the tokens the model emits for any input**
([§2.9.6.1][rules-2.9.6.1]). Where that puts a technique:

| If the technique… | Status | Section |
|---|---|---|
| is mathematically equivalent (exact sparse ops, online softmax, softmax elision under greedy) | Permitted, no disclosure | [§2.9.6.2][rules-2.9.6.2] |
| implements sparsity the canonical architecture specifies (native sparse attention, MoE routing) | Required | [§2.9.6.3][rules-2.9.6.3] |
| derives sparsity at run time from scores or activations (the Endpoints-specific delta) | Permitted under the accuracy gate, no benchmark-data calibration, disclosed | [§2.9.6.4][rules-2.9.6.4], [§2.9.6.6][rules-2.9.6.6] |
| changes structure: attention pattern, normalization, weight-side sparsity, removed experts | Disallowed, whatever accuracy does | [§2.9.6.5][rules-2.9.6.5] |

## Pre- and post-processing

Tokenization, chat template, truncation, detokenization, stop handling and sampling all have to
match the reference ([§2.9.2][rules-2.9.2], [§2.9.7][rules-2.9.7]).

!!! warning "Chat-template flags count too"
    Since 2026-09 the rule names the template's **flags** explicitly, such as `enable_thinking`.
    Flipping a thinking flag changes the effective input to the model, so it is a template
    modification, not a serving-side setting.

## The accuracy gate is necessary, not sufficient

!!! danger "It is not a general override"
    Hitting the quality target doesn't make a banned technique model equivalent
    ([§2.9.8][rules-2.9.8]).

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef) and
`mlcommons/endpoints-submission-cli@main` (f25f71e), 2026-09-24.*
