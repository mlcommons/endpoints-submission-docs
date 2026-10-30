<!--
  PROVENANCE SNAPSHOT — do not edit.
  Upstream : endpoints_policies/README.md
  Repo     : mlcommons/endpoints_policies @ main@4ba8224
  Captured : 2026-10-30
  Purpose  : diff this against upstream to find what drifted since the docs were written.
-->

# MLPerf Endpoints Policies

MLPerf Endpoints v1.0 Rules and Policies - 2026

v1.0 Rolling Submissions open on Oct 12th 2026

* This a locked branch that tracks the current rules as applicable for v1.0.
* 
* Please submit all PRs and proposals for v1.0 rules to v1.0_rules_dev branch. 

## Markdown checks

Wrap Markdown prose and headings at 100 characters. Code blocks and tables are exempt, as are
standalone links and link reference definitions. Unbreakable URLs may exceed the limit.
The line-width rule is configured in `.markdownlint.yaml`.

Install the Git hook to check Markdown files before committing:

```sh
python -m pip install pre-commit==4.6.0
pre-commit install
```

Run the same checks used by CI across all tracked files:

```sh
pre-commit run --all-files --show-diff-on-failure
```

If the check reports a long line, wrap it manually while preserving Markdown formatting.
