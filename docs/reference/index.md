# Reference

Tables to look things up in. For step-by-step instructions, see the
[Submission Workflow](../workflow/index.md) instead.

## What you can submit

| Page | Covers |
|---|---|
| [Benchmarks and models](benchmarks.md) | The v1.0 models, their datasets, and the accuracy the checker expects |

## Artifacts you author

| Page | Covers |
|---|---|
| [Submission package layout](package-layout.md) | The run folder, the assembled bundle, and what is shared vs per-point |
| [`point.yaml`](point-yaml.md) | Per-measurement-point disclosure fields |
| [`system_desc.json`](system-desc-json.md) | Hardware and software description: how to capture it or fill in the template |
| [`system_power.json`](system-power-json.md) | Provisioned-power descriptor, one per system |

## Measurement

| Page | Covers |
|---|---|
| [Metrics and regions](metrics-and-regions.md) | Metric definitions, the region algorithm, pre-computed boundary tables |

## Tools

| Page | Covers |
|---|---|
| [Benchmark runner CLI](cli-inference-endpoint.md) | `inference-endpoint` — runs the benchmark |
| [Submission CLI](cli-submission.md) | `endpoints-submission-cli` — registers runs, creates submissions |

## Process

| Page | Covers |
|---|---|
| [Compliance checks](compliance-checks.md) | Every check, cross-walked from checker rule ID to rules clause |
| [Submission states](submission-states.md) | Status values and what moves between them |
| [Glossary](glossary.md) | Terms and acronyms |

!!! note "These pages are hand-written"
    Each CLI page names the upstream document it tracks. Generating them from `--help` output is the
    right end state; until then, the upstream repository is authoritative and these pages can lag.

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
