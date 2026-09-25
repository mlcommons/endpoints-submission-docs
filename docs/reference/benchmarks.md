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

## Accuracy targets

Rules [§4.3][rules-4.3] leaves the quality target to each benchmark's definition, and those
definitions aren't published.

### Legacy benchmarks

The checker enforces the MLPerf Inference targets, carried over unchanged. Every score has to reach
99% of the reference score.

| Benchmark | Metric | Reference | Must reach | Minimum queries |
|---|---|---|---|---|
| Llama 3.1 8B | ROUGE-1 | 38.7792 | 38.39 | 13,368 |
| | ROUGE-2 | 15.9075 | 15.75 | |
| | ROUGE-L | 24.4957 | 24.25 | |
| | ROUGE-Lsum | 35.793 | 35.44 | |
| | Generated length, in tokens | 8,167,644 | Within 10% either way | |
| GPT-OSS 120B | Exact match | 83.13 | 82.30 | 4,395 |
| DeepSeek-R1 | Exact match | 81.3582 | 80.54 | 4,388 |

For GPT-OSS the query count includes the repeats, so AIME25 counts eight times.

### Agentic benchmarks

These targets come from the client's [agentic example
README](https://github.com/mlcommons/endpoints/blob/main/examples/10_Agentic_Inference/README.md#accuracy),
not from the checker (this line to be removed when changes are made to the submission checker).
There are three metrics:

- **Inline accuracy** and **OSL per-turn mean** have to pass at every submitted Pareto point.
- **SWE-bench accuracy** is judged on the average of four results, one from each mandatory region.

| Metric | Kimi K3 | Qwen3.6-35B-A3B | DeepSeek-V4.1-Flash *(proposed)* |
|---|---|---|---|
| Inline accuracy | At least 58.32% (reference 58.9%) | At least 55.86% (reference 56.43%) | At least 51.7% (reference 53.3%) |
| OSL per-turn mean, in tokens | 425–520 (reference 472) | 344–422 (reference 383) | 793–970 (reference 882) |
| SWE-bench accuracy | At least 93.5% (reference 94.83%) | At least 69% (reference 71.7%) | At least 96.4% (reference 97.5%) |

*Last verified against: the MLPerf Endpoints v1.0 rules overview (2026-09-22),
`mlcommons/endpoints@main` (e71b928) and open PR #519, `mlcommons/endpoints-submission-cli@main`
(f25f71e, tag `v1.0.1.0`) and open PR #93, and `mlcommons/endpoints_policies@v1.0_rules_dev`
(6b0b1ef), 2026-09-24.*
