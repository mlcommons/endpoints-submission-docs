# Troubleshooting

Indexed by what you see. Search this page for a fragment of your error message.

!!! note "Derived from documented behaviour, not from observed support cases"
    MLPerf Endpoints has not completed a submission round under these rules. Every entry is derived
    from a documented failure mode in the tooling or a stated rule, not from failures seen in the
    wild. Once real support threads exist, this page should be rewritten from them.

## Authentication

??? failure "Every command fails with an authentication error"
    **Cause:** the token is missing, mistyped, or not scoped correctly.

    **Fix:**

    1. Confirm the variable is set and starts with `mlc_`:
       ```bash
       echo "${PRISM_USER_API_TOKEN:0:4}"
       ```
    2. Confirm the key was created with **Service Scope: MLPerf Endpoints**. A key scoped to another
       service authenticates against a different surface.
    3. Try `--token` explicitly to rule out a shell-profile problem.

    See [Set your token](../workflow/register-runs.md#set-your-token).

??? failure "Connection errors, or requests going somewhere unexpected"
    **Cause:** `MLPERF_API_BASE_URL` is set.

    **Fix:** unset it. The CLI has the production API built in, and the variable should only be set
    for dev or staging environments.
    ```bash
    unset MLPERF_API_BASE_URL
    ```

## Installation

??? failure "`inference-endpoint: command not found`"
    **Cause:** the venv is not activated, or you installed with `uv sync` and dropped the prefix.

    **Fix:** either `source .venv/bin/activate`, or prefix every command with `uv run`.

??? failure "Dependency versions differ from what the project expects"
    **Cause:** you installed with `pip` rather than `uv sync`. The pip path does **not** use
    `uv.lock`.

    **Fix:** for a submission, prefer `uv sync` so your dependency set matches the lockfile. Record
    the commit SHA you built from either way, since you need it for disclosure.

## Running the benchmark

??? failure "`result_summary.json` shows `complete: false`"
    **Cause:** the run drained out or was interrupted. The metrics are partial.

    **Fix:** the point is **not usable** — re-run it. Check whether you hit `run_timeout_s`, a drain
    timeout, or `endpoint_response_idle_timeout_s`.

    A `state` of `"interrupted"` means the run was aborted; `"complete"` with pending tasks means a
    drain timeout.

??? failure "The Offline run has no per-token timing, or `streaming-config` fails"
    **Cause:** the client's `streaming: auto` default resolves to off for offline runs.

    **Fix:** set streaming on explicitly in the Offline point's config. `stream_all_chunks` must be
    `true` for every performance run.

??? failure "The run ends far sooner than the region minimum"
    **Cause:** sample-count sizing. With no explicit count and no minimum issue duration, the client
    issues the dataset **once** and stops.

    **Fix:** set `settings.runtime.n_samples_to_issue`, a multiple of the dataset size, so the run
    sustains 600 s (Ultra Low) or 1,200 s (other regions) of steady state ([§6.2 of the
    rules][rules-6.2]).

    Priority order: `n_samples_to_issue` > Poisson QPS × min issue duration > dataset size.

??? failure "A percentile lookup in `result_summary.json` returns nothing"
    **Cause:** percentile keys are **decimal strings** — `"50.0"`, `"90.0"`, `"99.9"`, not `"50"` or
    `"90"`.

    **Fix:** use the decimal form.

??? failure "Dataset validation fails saying samples cannot be salted"
    **Cause:** salt requires a dict sample with a text `prompt` field. A dataset whose samples carry
    `messages`, or multimodal content parts, cannot be salted, and the client validates every sample
    up front rather than shipping an unsalted payload.

    **Fix:** use a dataset with a text `prompt`, or disable the warmup salt, but if warmup uses the
    performance dataset, **salting is mandatory** ([§6.3.1 of the rules][rules-6.3.1]), so the
    dataset must support it.

??? failure "The endpoint stops responding and the run hangs"
    **Cause:** no liveness deadline set.

    **Fix:**
    ```yaml
    settings:
      timeouts:
        endpoint_response_idle_timeout_s: 300   # >= 300; raise for long requests
    ```

## Disclosure files

??? failure "The checker reports missing `point.yaml` or `system_desc.json`"
    **Cause:** you expected the reference client to generate them. It doesn't.

    **Fix:** author `point.yaml` by hand. Capture `system_desc.json` with
    [`mlperf-sysinfo`](https://docs.mlcommons.org/mlperf-sysinfo/) or write it from the §8.2.1
    template. Place both at the top level of every run folder. See [step
    5](../workflow/author-disclosures.md), [`point.yaml`](../reference/point-yaml.md) and
    [`system_desc.json`](../reference/system-desc-json.md).

??? failure "Fields you set in `config.yaml` do not appear in the submission"
    **Cause:** `point.yaml` is copied **verbatim**. The CLI does not derive it from `config.yaml`
    and does not fill in missing fields.

    **Fix:** put every required disclosure field in `point.yaml` itself.

??? failure "`shared-path-resolution` fails"
    **Cause:** `shared_src` or `shared_docs` does not resolve to an existing directory under the
    submission root.

    **Fix:** correct the pointers in every `point.yaml`. This check **rejects** the submission.

??? failure "`tps-utilization` fails"
    **Cause:** the value is not `system_tps / max(system_tps)` across your own curve.

    **Fix:** recompute after **every** point has run. You cannot fill it in before then.

## Validation failures

??? failure "`point-count` fails"
    **Cause:** too few points, or points were withdrawn. The minimum is 8 when a point declares
    `offline: dedicated`, and 7 otherwise.

    **Fix:** run more. Note withdrawn points do not count toward the minimum and **cannot be
    replaced**, because there's no `add-run`. If the curve needs a different set of runs, create a
    new submission.

??? failure "`offline-point-present` warns that no point declares `offline`"
    **Cause:** no dedicated Offline run and no elected `C_max` point. The checker only warns because
    it can't tell whether your benchmark is agentic.

    **Fix:** if the benchmark isn't agentic, this **will** be rejected. Add `offline: elected` to
    your `C_max` point's `point.yaml`, or run a dedicated Offline point and declare `offline:
    dedicated`. See [step
    3](../workflow/plan-your-curve.md#6-decide-how-to-meet-the-offline-requirement).

??? failure "`offline-point-present` fails on an `elected` point"
    **Cause:** `elected` is declared on a point whose concurrency isn't your declared `C_max`.

    **Fix:** elect the point at exactly `max_supported_concurrency`. If you don't have one, run it,
    or run a dedicated Offline point instead.

??? failure "`offline-ordering` warns"
    **Cause:** the dedicated Offline run's `system_tps` is below 0.98× your `C_max` point's, or its
    concurrency is below `C_max`.

    **Fix:** a low throughput usually means the Offline run didn't saturate the system; re-run it
    with more passes, or elect your `C_max` point instead. A low concurrency means your dataset is
    smaller than `C_max`, which the rules haven't resolved yet — see **C7** in [Open
    questions](open-questions.md).

??? failure "`power-descriptor` fails"
    **Cause:** `results/<system>/system_power.json` is missing, or it states neither a total nor any
    component group a total can be computed from.

    **Fix:** put a `system_power.json` in at least one run folder of that system. See
    [`system_power.json`](../reference/system-power-json.md).

??? failure "The build fails saying runs declare different `system_power.json` contents"
    **Cause:** two run folders of the same system carry different copies.

    **Fix:** make every copy identical, or keep it in one run folder only.

??? failure "`approved-drafter` fails"
    **Cause:** the point declares `speculative_decoding`, and the drafter isn't on the benchmark's
    approved list. Approved heads exist only for the agentic benchmarks, and the checker's list is
    still empty, so every drafter fails for now.

    **Fix:** re-run the point without speculative decoding.

??? failure "A concurrency-coverage check fails"
    **Cause:** no point in one of Low, Medium or High Concurrency — often because `C_min` changed.

    **Fix:**
    ```bash
    python -c "from submission_checker.cli import main; main()" regions --max-concurrency <C_max> --min-concurrency <your lowest point>
    ```
    Remember `C_min` is **derived from your own lowest point**, so dropping that point moves every
    other boundary. And a point in the 10% margin does **not** satisfy High Concurrency.

??? failure "`region-placement` warns"
    **Cause:** the `region` you declared in `point.yaml` disagrees with the computed one.

    **Fix:** correct the declared value. It is only a warning, but it is exactly the kind of thing a
    reviewer files a methodology objection about.

??? failure "`seed-set-adoption` reports SKIP"
    **Cause:** a checker older than `v1.0.1.0`, whose bundled seed-set file carries no cohort keys.

    **Fix:** `pip install -U endpoints-submission-cli`. From `v1.0.1.0` the adoption test runs
    against the published set.

??? failure "`accuracy-coverage` fails"
    **Cause:** no accuracy results at a point in one of the four mandatory regions, or none at the
    Offline point.

    **Fix:** the message names the region or the Offline point. Run that accuracy validation. An
    elected `C_max` point's accuracy run covers both High Concurrency and Offline.

??? failure "`accuracy-gate` fails"
    **Cause:** the accuracy run missed the benchmark quality target.

    **Fix:** there is **no tolerance** — this rejects the submission. Check whether an approximation
    permitted under [model equivalence](../rules/model-equivalence.md) pushed you below the target;
    dynamic approximate sparsity and aggressive PTQ are both gated on exactly this.

## Registering runs

??? failure "`Run folder error: … is missing required file(s): performance/result_summary.json`"
    **Cause:** a flat run folder with the summary at the top level.

    **Fix:** use the layout the reference client writes — the summary belongs under `performance/`.
    Flat layouts are not accepted. See [Submission package layout](../reference/package-layout.md).

??? failure "A run cannot be deleted"
    **Cause:** it belongs to an active submission.

    **Fix:** `submissions withdraw` first, then `runs delete`.

## Submission failures

??? failure "The build fails naming a specific run"
    **Cause:** the builder cannot tell whether the run is an accuracy or a performance run. It reads
    `datasets[].type` from `config.yaml`, then falls back to `point.yaml`'s `dataset_type`, but only
    when that is exactly `Accuracy` or `Performance`.

    **Fix:** supply `config.yaml`, or set `dataset_type` to exactly `Accuracy` or `Performance`.
    `Accuracy + Performance` describes the dataset, not the run, and is rejected on purpose:
    guessing used to file accuracy runs as performance runs and silently drop the accuracy results.

??? failure "`submissions update --run-ids` is rejected"
    **Cause:** the list would **add** a run. The post-submission window for adding points was
    removed.

    **Fix:** the list may only shrink. To remove one point, use `submissions remove-run`. For a
    different set of runs, create a new submission.

??? failure "The upload failed and you are unsure of the state"
    **Cause:** partial failure. The CLI rolls back automatically — a failed run-archive upload
    deletes the run record; a failed bundle upload withdraws the submission.

    **Fix:** if rollback also failed, the CLI prints the orphaned ID. Clean it up:
    ```bash
    endpoints-submission-cli runs delete --run-id <orphaned-id>
    endpoints-submission-cli submissions withdraw --submission-id <orphaned-id>
    ```

??? failure "`pr_url` is empty after a successful submission"
    **Cause:** not a bug. The CLI no longer opens the review pull request.

    **Fix:** nothing. `pr_url` and `pr_number` populate once whatever opens it has set them.

??? failure "`submissions create` succeeded but status is not `REVIEW_PENDING`"
    **Cause:** the final PATCH step failed. Both submission and bundle exist — the CLI treats this
    as a warning, not a fatal error.

    **Fix:** the status can be set manually. Confirm with `submissions get`.

## During review

??? failure "You missed the 3-business-day response window"
    **Cause:** no one was watching the review thread.

    **Fix:** respond immediately. A penalty already incurred stays, but responding stops further
    escalation. At 10 business days the submission is withdrawn ([Submission Rules
    §6.3][srules-6.3]).

    See [After you submit](../workflow/after-submission.md).

??? failure "A measurement point turns out to be wrong during review"
    **Cause:** an error found after compliance passed.

    **Fix:** results may **not** be changed during peer review ([Submission Rules
    §8.1][srules-8.1]). Withdraw the point (`submissions remove-run`) or the whole submission. A
    withdrawn point doesn't count toward the minimum and can't be replaced.

## Still stuck?

[Getting support](support.md).
