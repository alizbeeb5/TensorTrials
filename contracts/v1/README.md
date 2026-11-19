# TensorTrials pilot contracts

Use [the JSON examples](./examples.json) to develop each component independently. All examples are synthetic public fixtures. The numeric problem version and string IDs are opaque to consumers. All HTTP payloads are JSON except responses marked 204.

| Producer | Consumer | Handoff |
| --- | --- | --- |
| Ali, problem author | Frontend and API | Versioned public problem at `problems/stable-softmax/v1/problem.json`, matching the `problem` example. Frontend imports it at build time. API uses it to validate the ID/version and choose the judge. |
| Ali | Judge | Private case bundle keyed by the same ID/version. Trusted worker reads `{"cases":[{"input":[0,0],"expected":[0.5,0.5],"failure_feedback":"Check equal logits."}]}` from restricted storage. Use this public example locally. |
| Layal, frontend | Micheal, API | Authenticated create, poll and history requests below. |
| API | Judge worker | Worker claims one job through the internal API. API owns PostgreSQL and lease state. |
| Judge worker | API | Worker returns counts, verdict and safe feedback with its lease. API persists the result and serves the learner. |

## Browser API

- `POST /api/submissions` takes `submit_request`; `mode` is `run` or `submit`. The API derives identity from the session. Return 202 with `submit_response`. Each successful POST creates a fresh attempt. Poll by ID after creation.
- `GET /api/submissions/{id}` returns 200. All states include the ID, original request fields (including source so the editor can reopen it), and status. While queued or running, omit verdict fields. When finished, include `verdict`, `passed`, `total`, and `feedback` as in `poll_finished`. A finished public run may also include `examples`; omit actual when the function failed to return a value. Omit example details on private submissions.
- `GET /api/submissions?problem_id=stable-softmax&problem_version=1` returns 200 with `history_response` for the session user, newest first. Each item includes ID, problem ID/version, mode and status; include verdict only when finished. `solved` is true if any finished submit for that version was accepted. Initially return `{"items":[],"solved":false}`.
- Public errors use `{"error":"invalid_request"}` (400 for invalid fields, unknown problem/version or mode), `unauthenticated` (401), or `not_found` (404 for a missing attempt or one owned by another user).

## Worker API

Both internal endpoints require a worker service credential, kept in the trusted worker and API. The worker has no database dependency. For local development, use the examples as an in-memory transport.

1. `POST /api/internal/jobs/claim` with an empty object returns 200 with `claim_response` or 204 when idle. In one transaction the API claims a queued or expired job, sets it running, and issues a new lease ID. For the pilot, use a 60-second lease and bound each complete execution attempt to 30 seconds.
2. Resolve the public or private suite by problem ID, version and mode. Send only the source and the current input into the sandbox. Grade in the trusted process. Return finite JSON values for public outputs; describe non-finite results in feedback. Check shape and finiteness before the agreement’s numerical tolerance.
3. `POST /api/internal/jobs/{id}/complete` takes `complete_request`. The URL supplies the job ID. Accept `examples` only for public runs. Return 204 for the first valid completion and an identical retry from the winning lease. Reject a stale lease or conflicting repeat with 409. Reject invalid credentials (401), unknown jobs (404) and malformed results (400).
4. The API persists terminal state and strips lease information from learner responses. On worker loss, expired leases can be reclaimed. After three failed execution attempts due to worker loss, finish as `judge_error` with a retry message.

Verdicts are `accepted`, `wrong_answer`, `runtime_error`, `time_limit`, and `judge_error`. Counts are integers with `0 <= passed <= total`; total is suite size and accepted requires every case of a nonempty suite to pass. For infrastructure failure before loading the suite, report zero counts. Feedback is a learner-safe string. Private inputs, expected answers, stack traces from the grader and service credentials stay out of learner responses.

## Parallel development and integration

| Issue | Owned paths | Independent first check |
| --- | --- | --- |
| #1 Problem | `problems/`, `contracts/` | Validate the public problem and trusted grading data against these examples. |
| #2 Frontend | `apps/web/` UI and web manifest/lockfile, excluding API routes and server modules | Mock the HTTP calls with the JSON fixtures and reopen an attempt. |
| #3 API | `apps/web/app/api/`, `apps/web/server/`, `db/` | Drive create → claim → complete → poll with a fake worker and two test users. |
| #4 Judge | `services/judge/`, including Python dependencies | Load the job fixture from disk and emit a completion matching the example using public sample data. |

Branch from the commit containing this directory. The frontend owner initializes the Next.js app around existing API files and coordinates shared JS dependency edits. Contract changes go through #1 with review from the producer and consumer. First integrate browser + API with the fake worker, then connect the real worker. For the shared demo, run the public example, submit a numerically unstable solution, fix it, refresh, and reopen the accepted submission.
