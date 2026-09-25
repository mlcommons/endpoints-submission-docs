# Before you begin

Work through this list top to bottom. If you can't tick everything in the first section, stop here
rather than after step 4, which is where the accelerator time gets spent.

## You can't submit without these

- [ ] The **individual** who will submit has signed the [MLCommons CLA](../understand/eligibility/cla-process.md)
- [ ] A [**PRISM API token**](../understand/eligibility/prism-api-key.md) in `mlc_…` format, scoped to *MLPerf Endpoints* — see [step 1](register.md)
- [ ] You have chosen a **division**: Standardized, Serviced or RDI — see [Divisions and scenarios](../understand/divisions-and-scenarios.md)
- [ ] You have chosen a **scenario**: CoP or CoN
- [ ] Your benchmark **model is on the round's supported list**, and you know whether it's an
      **agentic** benchmark: that decides whether you need an Offline point and how many accuracy
      runs you owe. See [Benchmarks and models](../reference/benchmarks.md)
- [ ] You have the benchmark's **performance and accuracy datasets**, or know where to get them.
      The GPT-OSS performance set is on
      [MLCommons storage](https://inference.mlcommons-storage.org/index.html#gpt-oss-benchmark)
- [ ] You can reach the endpoint under test from wherever the client will run
- [ ] Python **3.12+** for the reference client; Python 3.10+ for the submission CLI

## What this actually costs

- [ ] **Accelerator time.** Minimum 8 points for a non-agentic benchmark: one at 600 s of steady
      state, six at 1,200 s, and a dedicated Offline run, plus warmup. Then **five accuracy runs**,
      one at each mandatory region point and one at the Offline point. Electing your `C_max` point
      as the Offline result saves the separate run. Agentic benchmarks need 7 points and four
      accuracy runs. You need exclusive access to the system for all of it, and a point that fails
      has to be re-run. See [§5.3][rules-5.3], [§6.2][rules-6.2] and [§4.3][rules-4.3].
- [ ] **Power data** (Standardized). Public TDP figures for your CPUs, accelerators and scale-up
      switches, each with a link to a spec sheet. Gaps get filled with MLCommons's conservative
      estimates, so it's worth finding these early. See
      [Power normalization](../rules/requirements.md#power-normalization).
- [ ] **Disk.** `events.jsonl` is the bulk of each run folder — around 46 MB for a 60-second,
      16-concurrency run, scaling with sample count. A 600-second point reaches several hundred MB.
- [ ] **Upload bandwidth.** Every run folder is archived and uploaded before the submission is
      assembled.
- [ ] **Someone available for six weeks.** Once review starts, you have **3 business days** to
      respond to any objection. After 10 business days of no response your submission is withdrawn
      ([Submission Rules §6.3][srules-6.3]). This is an easy way to lose a submission after all the hardware time is already spent.

## Decisions to make now, not later

- [ ] **`C_max`** — your maximum supported concurrency. It sets your region boundaries and therefore
      which concurrency levels are legal. Getting it wrong means re-running. [Step 3](plan-your-curve.md).
- [ ] **Publication status** — Available, Preview or RDI. Preview commits you to achieving
      availability within **180 days** and re-submitting, or the result is invalidated. See
      [Publication status](../rules/publication-status.md).
- [ ] **Publication mode** — confidential review (the default), confidential with an embargo date,
      or provisional publication with a "peer review pending" tag. You **can't change it after
      submitting**, so decide now. An embargo on a provisional submission also delays the start of
      review. See [When your results become public](../understand/how-submission-works.md#when-your-results-become-public).
- [ ] **Speculative decoding?** Only with a drafter on the benchmark's approved list. No list has
      been published yet, so for now it isn't available for any benchmark. See
      [Model equivalence](../rules/model-equivalence.md#speculative-decoding).
- [ ] **Who signs off on disclosure.** Standardized requires publishing your configuration, launch
      scripts and integration code. Get that cleared internally before you run, not after.

## Read these before running anything

- [ ] [Requirements you must meet](../rules/requirements.md) — the binding constraints
- [ ] [Model equivalence](../rules/model-equivalence.md) — if you are submitting Standardized, this
      governs every optimisation decision you are about to make
- [ ] [Why submissions get rejected](../rules/rejection-reasons.md) — the failure modes
- [ ] [Open questions and WIP rules](../help/open-questions.md) — what could still move under you

--8<-- "draft-rules-warning.md"

## Verify

You're ready to start when every item above is ticked, or you know who is getting it and when.

## Next

→ [1. Get access and an API key](register.md)
