# TensorTrials team agreement

TensorTrials is AI Journey’s practice platform for ML, embedded AI, and computer hardware. The first milestone is one complete ML problem. A student opens Stable Softmax, edits Python in the browser, runs an example, submits against private cases, receives feedback, and can reopen the saved submission after signing in again. Once this works, expand the ML Foundations catalog toward ten problems.

## How we work

GitHub Issues tracks owners, acceptance criteria, dependencies, and blockers. Use Slack for quick discussion and record decisions that change an interface or grading rule on the relevant issue. Each implementation PR links its issue, explains how to verify the change, and gets a teammate’s review. Close the issue when its acceptance criteria work on main. Propose changes to this agreement through a PR.

## What each part owns

| Part | Responsibility |
| --- | --- |
| Problem authors | Version the prompt, signature, examples, runtime, comparison rules, and feedback. Keep private cases and expected answers in access-controlled judge storage. |
| Web app | Next.js and TypeScript serve the problem and editor. The API authenticates requests, records jobs, and returns results only to the submitting user. |
| Judge | A Python worker claims queued jobs, invokes a fresh resource-limited sandbox with network access disabled, grades returned values outside the sandbox, and persists a final result. Keep credentials and private expected answers outside submitted code. |
| Data | PostgreSQL holds the user, source, problem ID and version, run mode, job status, and verdict. Only an accepted submission counts toward solved progress. |

## Shared interface for the first milestone

- A problem declares `id`, `version`, `judge_type`, function signature, starter code, and public examples. The first judge is `python_numpy`. Later, embedded C and RTL simulation can return the same result format with additional measurements.
- `POST /api/submissions` takes `problem_id`, `problem_version`, `mode` (`run` or `submit`), and `source`. The server derives the user from the session and returns a job ID.
- `GET /api/submissions/{id}` returns `status` (`queued`, `running`, or `finished`). A finished job includes `verdict`, `passed`, `total`, and learner-facing `feedback`. Verdicts are `accepted`, `wrong_answer`, `runtime_error`, `time_limit`, or `judge_error`. A judge failure is distinguishable from a student error.
- Run uses public examples and can show expected and actual values. Submit uses private cases and reports a count and a diagnostic. Both modes use the same pinned runtime.

## First problem and demo

`softmax(x)` accepts a nonempty one-dimensional NumPy array of finite real logits and returns probabilities of the same shape. For `[0, 0]`, return `[0.5, 0.5]`. Compare against a stable reference using `rtol=1e-5` and `atol=1e-8`, checking shape and finiteness first.

The milestone demo exercises four outcomes: a correct solution passes, a numerically unstable solution gets actionable feedback, an exception gets a runtime verdict, and an infinite loop times out. Refresh the page and verify that the accepted submission and solved status persist. Embedded C and SystemVerilog challenges follow the ML pilot.
