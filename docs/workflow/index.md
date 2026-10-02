# Submission Workflow

Eight steps, in order, from an empty directory to a submission in the review queue. Each page
links to the next, and each ends with a **Verify** section so you can check the step worked before
moving on.

```mermaid
flowchart LR
    R[1 Register] --> I[2 Install] --> P[3 Plan curve] --> X[4 Run points]
    X --> D[5 Author disclosures] --> V[6 Validate] --> S[7 Submit] --> A[8 Review]
    V -.->|errors| D
```

| Step | What it produces |
|---|---|
| [Before you begin](before-you-begin.md) | A decision to proceed, or to stop before it costs anything |
| [1. Register and get a token](register.md) | A PRISM API key in `mlc_…` format |
| [2. Install the tools](install-tools.md) | Reference client, submission CLI, checker |
| [3. Plan your Pareto curve](plan-your-curve.md) | The concurrency levels you'll run, and your plan for the Offline point |
| [4. Run the measurement points](run-the-points.md) | One run folder per point, with accuracy results at the required points |
| [5. Author the disclosure files](author-disclosures.md) | `system_desc.json` and `point.yaml` per point, `system_power.json` per system |
| [6. Validate locally](validate.md) | A clean `submission-checker` report |
| [7. Submit](submit.md) | An uploaded bundle and a review PR |
| [8. After you submit](after-submission.md) | A finalized, published result |

Start step 1 before anything else. It's the only step that waits on someone outside your
organisation, and no turnaround time is published.

!!! tip "Two steps that catch people out"
    **Step 3.** Choosing `C_max` badly means re-running points, and each point is a 600- or
    1,200-second run. Forgetting the Offline point costs another run on top.
    **Step 5.** Three required files are hand-authored, and no tool generates them for you.

--8<-- "draft-rules-warning.md"
