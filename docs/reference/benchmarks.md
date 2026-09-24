# Benchmarks and models

## The v1.0 suite

| Benchmark | Model on Hugging Face | `model_name` the checker accepts |
|---|---|---|
| Llama 3.1 8B | `meta-llama/Llama-3.1-8B-Instruct` | `llama3.1-8b` |
| GPT-OSS 120B | `openai/gpt-oss-120b` | `gpt-oss-120b` |
| DeepSeek-R1 | `deepseek-ai/DeepSeek-R1` | `deepseek-r1` |
| Kimi K3 *(agentic)* | `moonshotai/Kimi-K3` | None yet |
| Qwen3.6-35B-A3B *(agentic)* | `Qwen/Qwen3.6-35B-A3B` | None yet |
| DeepSeek V4 *(agentic, tentative)* | Not settled, see below | None yet |

## Datasets

| Benchmark | Performance dataset | Accuracy dataset | Offline concurrency |
|---|---|---|---|
| Llama 3.1 8B | CNN/DailyMail 3.0.0, validation split: 13,368 articles | The same set | 13,368 |
| GPT-OSS 120B | The MLPerf GPT-OSS set, `perf/perf_eval_ref.parquet`: 6,396 prompts | AIME25 ×8, GPQA ×5 and LiveCodeBench ×3: 4,395 queries issued | 6,396 |
| DeepSeek-R1 | The MLPerf DeepSeek-R1 set: 4,388 prompts drawn from GPQA, MMLU-Pro, MATH500, AIME and LiveCodeBench | The same set | 4,388 |
| All agentic | workato (500) and deepswe (113), shipped as one JSONL file | SWE-bench Verified, first 200 tasks (`princeton-nlp/SWE-bench_Verified`) | No Offline point |

The Offline column is there because a dedicated Offline run reports one full pass over the
performance dataset as its concurrency ([§5.7.1][rules-5.7.1]). If your `C_max` is larger than that
number, read **C7** in [Open questions](../help/open-questions.md) before you plan.

Where to get them:

- **CNN/DailyMail**: the client downloads it when a config names the dataset
  `cnn_dailymail::llama3_8b`.
- **GPT-OSS performance set**: [MLCommons storage](https://inference.mlcommons-storage.org/index.html#gpt-oss-benchmark),
  under "Dataset for GPT-OSS benchmark". The download holds several files; the one you need is
  `perf/perf_eval_ref.parquet` (MD5 `e4cd6cef6dd975f3e50c85b3279b358b`).
- **DeepSeek-R1**: a pre-tokenized copy ships in the client repository at
  `examples/07_DeepSeekR1_Example/data/deepseek_r1_eval.parquet`, stored with git-LFS.
- **Agentic performance set**: [MLCommons storage](https://endpoints.mlcommons-storage.org/index.html#mlperf-agentic-inference).
  Use the file unchanged. Its SHA-256 is
  `1beb24c882122df96571cf11b390acbea388944038bc55c78b891475459014ae`.
- **SWE-bench Verified**: from Hugging Face, loaded by the client's SWE-bench scorer.

## Accuracy the checker expects

Rules [§4.3][rules-4.3] leaves the quality target to each benchmark's definition, and those
definitions aren't published. What the checker enforces today are the MLPerf Inference targets,
carried over unchanged. Every score has to reach 99% of the reference score.

| Benchmark | Metric | Reference | Must reach | Minimum queries |
|---|---|---|---|---|
| Llama 3.1 8B | ROUGE-1 | 38.7792 | 38.39 | 13,368 |
| | ROUGE-2 | 15.9075 | 15.75 | |
| | ROUGE-L | 24.4957 | 24.25 | |
| | ROUGE-Lsum | 35.793 | 35.44 | |
| | Generated length, in tokens | 8,167,644 | Within 10% either way | |
| GPT-OSS 120B | Exact match | 83.13 | 82.30 | 4,395 |
| DeepSeek-R1 | Exact match | 81.3582 | 80.54 | 4,388 |
| Agentic | None defined | | | |

For GPT-OSS the query count includes the repeats, so AIME25 counts eight times.

!!! note "No agentic target yet"
    For a model with no target, the checker reports a warning, `No accuracy thresholds defined`, and
    skips the check. A clean checker run tells you nothing about your agentic accuracy. Tracked
    as **C2** in [Open questions](../help/open-questions.md).

??? info "Caveats"
    - **The third agentic model isn't settled.** The working group's overview lists
      *DeepSeek-V4.1-Flash* and marks it tentative. The client's example config serves
      `deepseek-ai/DeepSeek-V4-Pro-0813`. Don't plan around either until the official list is
      out. Tracked as **C11**.
    - **GPT-OSS runs with two output limits.** The MLPerf reference allows 10,240 output tokens
      with `reasoning_effort` low for performance, and 32,768 with high for accuracy. Reasoning
      effort isn't a client setting: for the performance run it's fixed when the parquet file is
      built.
    - **DeepSeek-R1 has an unchecked second metric.** The MLPerf spec also bounds tokens per sample
      to within 10% of 3,886.2274. The checker doesn't check it, because the number isn't in
      `results.json`.
    - **Llama 3.1 8B needs its chat template.** To match MLPerf output, the server has to accept
      the template the client sends. With vLLM that's `--trust-request-chat-template`.
    - **Agentic runs have extra required settings.** Salting and inline accuracy must both be on,
      and `stop_issuing_on_first_user_complete` must be `false`. The client's agentic example
      README has the details.
    - **Expect the suite to grow.** Adding benchmarks quickly is one of the goals of the rolling
      submission model, so more models are likely soon after v1.0 opens.

*Last verified against: the MLPerf Endpoints v1.0 rules overview (2026-09-22),
`mlcommons/endpoints@main` (e71b928), `mlcommons/endpoints-submission-cli@main` (f25f71e, tag
`v1.0.1.0`) and `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
