---
name: code-reviewer
description: Review a pull request or branch as one full diff, re-check earlier findings, analyze resource/load costs, distinguish service crashes and service-wide failures from isolated access failures, and write a new evidence-backed Markdown report. Use for repeat PR reviews where fixes landed and operational regressions or occurrence ratios matter.
---

# Code Reviewer

Produce a fresh, actionable review report without modifying product code. Treat the current source and current PR diff as authoritative; use an earlier report only as a checklist whose conclusions must be revalidated.

## Establish the review state

1. Resolve the PR HEAD and base with the hosting CLI or existing remote refs.
2. Update the local review checkout safely. Preserve uncommitted and untracked user files. If the PR branch was force-pushed and cannot fast-forward, create a new local review branch at the current remote HEAD instead of resetting or deleting the old branch.
3. Record exact head/base SHAs, timestamps, full diff statistics, and the delta from the previously reviewed SHA when available.
4. Review the full merge-base diff by default. Narrow or chunk it only when the user requests that scope.
5. If a prior Markdown report exists, extract every finding ID and expected fix. Preserve IDs across reviews; assign new IDs only to newly discovered issues.

## Review with independent lenses

Use three independent passes. When collaboration agents are available, delegate these passes in parallel; otherwise perform them sequentially without blending their initial conclusions.

- **Blind defect pass:** inspect the full diff without relying on the previous report. Look for correctness, authorization, data loss, concurrency, migration, durability, and operational failures.
- **Edge-case pass:** walk failure boundaries: crashes between state changes, timeouts, lease expiry, retries, duplicate delivery, partial external success, null/empty/max-size input, multi-pod races, deployment from older schema states, and cleanup after rollback.
- **Acceptance pass:** re-check every prior finding against current lines and classify it as `FIXED`, `PARTIALLY_FIXED`, `NOT_FIXED`, or `REGRESSED`. A passing unit test does not by itself prove a finding fixed.

Merge duplicate findings only after the passes complete. Prefer the most specific evidence and keep unique consequences or locations from every pass.

## Analyze resource and server-load chains

For every applicable path, trace user input or background work through:

`entry -> validation -> transaction/lock -> rows/index/WAL -> heap/CPU -> external calls -> retry/outbox -> scheduler/request threads -> retention/cleanup`

Check at least:

- CPU: loops, repeated serialization/encryption, sorting, repeated snapshots, retry amplification.
- Memory: unbounded collections, full-table/list reads, payload accumulation, executor queues, retained snapshots.
- Disk/database: row and index growth, WAL/TOAST, transaction size, N+1 queries, lock duration, connection occupancy, migration scans and DDL locks.
- Network: calls per user/task/key, serial fan-out, payload bytes, retry storms, authenticated URL handling.
- Threads: synchronous post-commit work, servlet/executor monopolization, scheduler pool capacity, head-of-line blocking.
- External resources: orphaned keys, stale grants, delete idempotency, durable compensation, Mercury/event delivery.

Quantify costs from enforced limits when possible. Show the formula and assumptions, for example `members × (task keys + project key)`, rows per operation, calls per minute, or theoretical drain time. Clearly label estimates; do not present them as observed production rates.

## Prioritize crash and service-availability impact

Perform an explicit crashability checkpoint on every review, even when the user only asks about resource cost. This checkpoint changes triage and fix order, not the report schema. Keep the normal review format and describe the largest supported blast radius inside each finding:

- **Process crash or startup loop:** directly terminates/restarts the process, prevents startup/readiness, or deterministically causes an orchestrator restart.
- **Service-wide unavailability:** makes the service broadly unable to serve traffic, including database unavailability, exhausted connection/thread pools, disk-full conditions, or a required dependency failure that blocks most requests.
- **Service-wide degradation/resource-exhaustion risk:** can saturate CPU, heap, disk, database, network, request threads, executors, or schedulers, but the evidence does not prove process termination. Do not call this a confirmed crash without a demonstrated termination mechanism.
- **Request-scoped failure:** fails or times out the triggering request without materially affecting unrelated traffic.
- **User/key-scoped access failure:** prevents only the affected user, tenant, object, or encryption key from being accessed while the rest of the service remains healthy.
- **No availability impact:** correctness, security, or durability impact without an availability consequence.

For the crashability checkpoint:

1. Identify every direct crash/startup-loop path and give the exact exception, migration failure, OOM/kill mechanism, or readiness failure when supported by evidence.
2. Identify indirect resource-exhaustion chains that could become service-wide failures. State the missing capacity or production metric when a crash cannot be proven from source alone.
3. Explicitly separate isolated access failures from service-wide failures. Do not promote a single-user decryption/access failure to a service crash.
4. For every process-crash, startup-loop, or service-wide-unavailability finding, trace the root cause to a concrete permanent fix and a verification plan. Include safe rollout/rollback or data-recovery steps when migration or destructive data behavior is involved. Logging or alerting alone is not a fix.
5. Give confirmed process crash, startup/readiness failure, and service-wide unavailability the highest review priority: classify them as `P0`, place them first in the executive conclusion and fix order, and treat them as mandatory release blockers.
6. Keep credible but unproven service-wide resource-exhaustion paths at `P1` unless a deterministic crash/outage mechanism is established. Place them immediately after confirmed crash blockers and block release unless a tested hard bound proves the service remains within its capacity budget.
7. Do not promote request-scoped or user/key-scoped access failure solely because a process crash can occur inside its failure window. Prioritize it from its actual blast radius, security impact, and recoverability.
8. If no direct crash path exists, state that explicitly and list any remaining service-wide saturation risks separately.

Continue the investigation until each crash or service-wide blocker has a recommended fix that closes its failure chain. The review does not authorize implementing that fix; implementation still requires a separate user request.

Do not create a separate crashability report, addendum, mandatory crash section, or crash-specific table/column. Integrate the conclusion, classification, scenario, resource chain, fix, and expected result into the same standard report sections and finding rows used for every review. When re-reviewing crashability at an unchanged SHA, produce or update the normal-format review artifact requested by the user rather than inventing a parallel format.

## Describe occurrence probability correctly

For each issue report both concepts:

- **Conditional occurrence ratio:** what happens once the stated trigger has occurred. Use `100%` only when control flow makes the consequence deterministic.
- **Production trigger estimate:** `high`, `medium`, `low`, or `unknown`, with the missing metric needed to replace the estimate (such as p99 latency, error rate, object-size distribution, or backlog age).

Never invent an absolute production incident percentage from source code alone.

## Evidence and verification

- Cite current repository-relative file paths and verified one-based lines.
- Run focused tests proportional to risk and record the exact total and outcome. Add migration validation, concurrency tests, or scale tests when available.
- State what tests do not cover, especially real dependency latency, multi-pod races, production data size, migration history, and capacity.
- Distinguish confirmed facts, source-based inference, and assumptions.
- Do not fix code, push branches, post PR comments, or mutate external systems unless the user separately asks.

## Write a new report

Read [references/report-format.md](references/report-format.md) before writing the deliverable.

Use the user's requested language. Create a new filename containing the current short SHA, such as `review-latest-<sha>.md`; never overwrite earlier reports. Open the result for the user when supported.

The executive conclusion must state:

- old-finding status counts;
- new-finding counts by priority;
- confirmed process-crash/startup-loop findings, service-wide failure risks, and isolated access-only failures;
- test evidence;
- whether merge/release is recommended;
- the smallest set of release blockers.
