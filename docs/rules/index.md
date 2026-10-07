# Rules & Compliance

A plain-language summary of the rules that apply to you. **These pages don't copy the policy text**
— each requirement is summarised and linked to the clause it comes from.

--8<-- "precedence-notice.md"

| Page | Covers |
|---|---|
| [Requirements you must meet](requirements.md) | The binding constraints, by category, with how to tell whether you comply |
| [Model equivalence](model-equivalence.md) | Standardized-division rules on weights, quantization, speculation, KV cache and sparsity |
| [Publication status](publication-status.md) | Available, Preview and RDI — the availability tests and the clocks attached to each |
| [Why submissions get rejected](rejection-reasons.md) | Failure modes, as symptom → cause → fix |

## The source documents

| Document | Governs |
|---|---|
| [`endpoints_rules.md`][rules] | Technical: divisions, metrics, power normalization, Pareto methodology and the Offline point, run requirements, package contents, compliance checks |
| [`endpoints_submission_rules.md`][srules] | Process: registration, review, cohorts, publication modes, objections, publication status, disputes, audits |
| [`MLPerf_Endpoints_Audit_Guidelines.md`][audit] | What an auditor checks, and what you provide during an audit |

These links go to the `v1.0_rules_dev` branch, which is where v1.0 is being written. The
repository's `main` branch still carries the v0.7 text.

The two rules documents build on the [MLPerf General Submission
Rules](https://github.com/mlcommons/policies/blob/master/submission_rules.adoc), and Endpoints wins
where they conflict ([Submission Rules §1][srules-1]).

--8<-- "draft-rules-warning.md"

*Last verified against: `mlcommons/endpoints_policies@v1.0_rules_dev` (6b0b1ef), 2026-09-24.*
