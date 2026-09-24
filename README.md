# MLPerf Endpoints — Submitter Documentation

The submitter-facing documentation site for MLPerf Endpoints: how to prepare, validate and submit a
benchmark result. Built with MkDocs Material.

**Published at:** <https://mlcommons.github.io/endpoints-submission-docs/>

- **Open questions and WIP rules:** [`docs/help/open-questions.md`](docs/help/open-questions.md)
- **Contributing:** [`CONTRIBUTING.md`](CONTRIBUTING.md)

The design brief, task plan and the MLCommons handoff document are kept in the authoring
workspace and are deliberately not published here.

## Authoritative sources

This site is a guided path over, not a replacement for:

| Source | Authoritative for |
|---|---|
| [`mlcommons/endpoints_policies`](https://github.com/mlcommons/endpoints_policies) | All rules and process |
| [`mlcommons/endpoints-submission-cli`](https://github.com/mlcommons/endpoints-submission-cli) | Submission CLI and checker behaviour |
| [`mlcommons/endpoints`](https://github.com/mlcommons/endpoints) | The reference benchmark client |

Snapshots of the exact documents mined, with their refs, are in [`_sources/`](_sources/).

## Building locally

```bash
pip install -r requirements.txt
mkdocs serve
```

`mkdocs build --strict` is what CI runs; a broken internal link or anchor fails the build.

## Deployment

Pushing to `main` runs [`.github/workflows/docs.yml`](.github/workflows/docs.yml), which builds
strictly and then publishes to the `gh-pages` branch via `mkdocs gh-deploy`. GitHub Pages serves
that branch. The `site/` directory is build output and is not committed.
