# Code Reviewer report format

Use this schema when writing the Markdown artifact. Add a column only when it materially improves ownership, verification, or release decisions.

## Required sections

1. **Review scope and conclusion**
   - PR/branch, exact head and base SHAs
   - full diff and since-last-review delta statistics
   - methodology and focus dimensions
   - old finding status totals, new finding totals, verification result, merge/release recommendation
   - counts of confirmed process-crash/startup-loop findings, service-wide failure risks, and isolated access-only failures
   - put confirmed crash/startup/readiness/service-wide-unavailability blockers first and state that they have the highest fix priority

2. **Probability definitions**
   - define conditional occurrence ratio
   - define production trigger estimate and its evidence limitations

3. **Prior checkpoint re-review**

   | ID | Priority | Current status | Key description and code location | Detailed user scenario | Resource/impact chain | Trigger and occurrence ratio | Recommended fix | Expected result |
   |---|---|---|---|---|---|---|---|---|

4. **New findings**

   Use the same columns. Allocate stable new IDs and explain why each issue is new rather than a duplicate of an existing checkpoint.

5. **Fix effectiveness and residual costs**

   | Change | What it fixed | Residual or newly introduced cost |
   |---|---|---|

6. **Capacity quantification**

   | Scenario | Formula and theoretical cost | Main risk |
   |---|---|---|

7. **Recommended fix order**
   - order by release safety and dependency, not merely numeric ID
   - put confirmed process crash, startup/readiness failure, and service-wide unavailability first as `P0` mandatory release blockers
   - put credible but unproven service-wide resource-exhaustion risks next as `P1`, unless capacity tests establish a safe hard bound
   - separate release blockers from follow-up hardening

8. **Suggested tracking fields**
   - Owner
   - Target release
   - Fix PR/commit
   - Verification evidence
   - SLO/capacity budget
   - Residual risk/accepted-by
   - Rollback/reconciliation runbook
   - Production occurrence data

9. **Verification evidence and limitations**
   - exact commands or test classes, totals, failures/errors/skips
   - untested production conditions

## Finding quality rules

Each retained finding must contain:

- a concrete trigger and observable consequence;
- a tight current-code location;
- an explicit failure class and blast radius in the description, scenario, or impact chain without adding a special report column;
- a realistic user or operator scenario with ordered steps;
- the complete CPU/memory/disk/DB/network/thread/external-resource chain that applies;
- conditional occurrence ratio plus a calibrated production estimate;
- a fix that closes the failure window rather than merely adding logging;
- a testable expected result.

For a process-crash, startup-loop, or service-wide-unavailability finding, the recommended fix is mandatory and must address the root cause. Put it at the top of the normal finding table and fix order; do not create a separate crashability section or report format. For an isolated user/key access failure, say explicitly that unrelated traffic remains healthy and identify the alerting, reconciliation, and recovery SLO needed if the risk is accepted.

Use `P0` for confirmed process crash, startup/readiness failure, service-wide unavailability, data destruction, or deterministic deployment/release blockers. Use `P1` for credible but unproven service-wide resource exhaustion and other serious security, correctness, availability, or capacity risks. Use `P2` for bounded or lower-likelihood operational debt. Do not inflate priority solely because a path is complex or contains the word "crash".

## Status rules

- `FIXED`: the original failure chain is closed and verification supports it.
- `PARTIALLY_FIXED`: meaningful risk reduction exists, but a reproducible failure or unbounded resource chain remains.
- `NOT_FIXED`: the original trigger and consequence remain substantially present.
- `REGRESSED`: the change worsened the original issue or added a more severe path.

When a fix creates a distinct new problem, keep the old issue's honest status and add a separate new finding. Do not hide the new risk inside a “fixed” row.
