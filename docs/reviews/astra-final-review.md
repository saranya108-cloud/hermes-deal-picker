# Astra final review — Hermes Deal Picker Rev 3

Review text reproduced verbatim from Amy's supplied final Astra review in the referenced conversation. The review's scope and authorization statements are preserved as issued.

---

**Verdict: ACCEPT WITH NON-BLOCKING NOTES — for offline selection-only Tranche 1.**

**B1 is resolved. Tranche 1 is architecturally ready for Human Chair authorization.** I found no remaining blocker within that scope.

### Blocking findings

**None.**

Rev 3 consistently resolves both parts of B1:

| Check                                         | Finding                                                                                                                                            |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Explicit budget input                         | `BudgetSnapshot` is immutable, simulated and included in the determinism contract.                                                                 |
| No reservation side effect                    | Tranche 1 evaluates affordability without changing balances or creating reservations.                                                              |
| Outputs distinguish simulation from authority | Plans carry `required_reservation`, a snapshot reference and a simulated result—not a real reservation ID or spending authority.                   |
| Stateful work deferred                        | Transactions, concurrency, repeated-request semantics, reconciliation and obligation release belong exclusively to separately reviewed Tranche BL. |
| Obligations survive invalidation              | Sections 1.7 and 1.8 explicitly preserve incurred and uncertain obligations when pricing changes or a plan expires.                                |
| Tests match the scope                         | Section 4 replaces ledger/concurrency tests with immutable snapshot scenarios and requires deterministic outputs and no persistent mutation.       |

The correction is reflected in component ownership, core objects, economic rules, frozen decisions, tranche contents and acceptance criteria. It is not merely a change to the summary.

### Non-blocking notes

1. **Preserve the simulation distinction in implementation.** Synthetic evidence and unverified Hermes assumptions must remain visible in outputs. A successful offline selection must not be presented as proof that a route is currently executable or its controls are enforced.
2. **Make determinism cover every declared input.** Canonical output and decision identifiers should depend on the supplied inputs—including capability and pilot-profile snapshots—and the explicit evaluation time. Ambient time, random identifiers and filesystem ordering must not change the result. This implements the existing requirement; it adds no new architecture gate.
3. **Read the summary claims subject to the detailed live gates.** Section 2 still describes Option B as needing no Hermes change, while section 1.10 correctly makes that conditional. Similarly, S0 structural protection remains something to demonstrate before live use. These summary shortcuts do not affect Tranche 1 because V1–V9 and L1 explicitly gate later execution.

The versioned evidence boundary and standalone `hermes-deal-picker` repository remain appropriate. MPW dependencies, Hermes integration, live routing and Tranche BL remain outside the first offline tranche.

### First-tranche verdict

**Yes: Amy may now consider authorizing the offline selection-only Tranche 1 as defined in section 4.** No further architecture revision is required before that authorization decision.

This acceptance does **not** authorize implementation, repository creation, Tranche BL or any live execution. It also does not certify installed Hermes compatibility.

I reviewed the attached document against the previous B1 finding. No code was executed and no repository, file, schema or configuration was modified.
