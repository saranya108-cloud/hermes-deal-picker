# Hermes Deal Picker

Standalone Deal Picker architecture foundation.

## Baseline and authorization

- **Frozen architecture:** Hermes Deal Picker — Revised Architecture (Rev 3), dated November 10, 2026, reproduced verbatim below from the reviewed attachment.
- **Final independent review:** Astra — **ACCEPT WITH NON-BLOCKING NOTES**, for offline selection-only Tranche 1. B1 is resolved; no blocking findings remain within that scope. See [the complete review](docs/reviews/astra-final-review.md).
- **Human Chair authorization:** repository creation and the four foundation files only. Implementation, Tranche BL, Hermes integration and live execution remain unauthorized.
- **Implementation requirements carried forward from the review:** keep simulated evidence and unverified assumptions visible; include every declared input and explicit evaluation time in determinism; retain V1–V9 and L1 as gates for later live use.

The source document's original review-pending wording is preserved as part of the frozen text. The final review above records the subsequent outcome; it does not authorize implementation. The proposed future repository layout inside the architecture is descriptive only.

This repository contains only `README.md`, `LICENSE`, `.gitignore`, and `docs/reviews/astra-final-review.md`.

## Frozen Rev 3 architecture — original text

```text
Hermes Deal Picker — Revised Architecture (Rev 3)
Nov 10, 2026 · @Amy Canfield
Status and scope
This is Revision 3 of the Hermes Deal Picker architecture. It answers Astra's second review (REVISE, limited to the first tranche's budget contract) with a minimal correction; everything Astra accepted stands unchanged. It is for independent review only: it authorizes no code, repository, schema or configuration change in the Deal Picker, MPW, Hermes Agent or Model Council Lab.
• The Deal Picker is a separate add-on project. It consumes MPW evidence but is not part of MPW. It is also distinct from MPW's planned public Hermes commands plugin (/price, /deals and so on); the picker does not depend on that plugin.
• Inputs. The two revision briefs and Astra's two reviews (MPW at commit 8f117612, upstream Hermes sources). Duet's earlier proposal text was not available for this pass; where this document refers to it, it relies on Astra's description.
• Hermes behaviour is taken as Astra read it upstream. Every Hermes behaviour this design depends on is listed as a verification item (V1–V9, section 1.6) to check against the Hermes version Amy actually runs.
• Conventions. MUST and MUST NOT are binding. "Refuse" means return a typed refusal and make no inference call. "Operator" means Amy acting as Human Chair through versioned policy files.
1. Revised architecture
The Deal Picker turns operator policy plus MPW evidence into either one validated, expiring execution plan for Hermes or a typed refusal. It never sends inference, holds secrets or reinterprets prices.
1.1 Invariants
1. Evidence is not permission. MPW evidence can make a route cheaper. Only operator policy can make it allowed.
2. Unknown stays unknown. No field defaults to zero, free, private, equivalent or available.
3. Policy only tightens at runtime. Nothing a model, task, tool or delegated call supplies can lower sensitivity, widen the route set, relax privacy or raise a limit.
4. Refuse rather than weaken. A route, chain, limit or control that Hermes cannot represent faithfully produces a refusal.
5. Every decision expires. A plan carries a validity deadline. Using it later requires a new decision.
6. Worst-case budgets, not estimates. Paid execution needs a provable upper bound, reserved before the call.
7. Deal status and suitability are separate. A promotion ending expires the deal, not the route's approval or the model's trust.
8. Determinism. The same evidence, registry, policy, budget snapshot and evaluation time produce a byte-identical plan or refusal.
1.2 Components and ownership
Component
Owns
Never does
Model Price Watcher
Acquisition, provenance, exact offering identities, timestamps, supported price interpretation, history, unknown/unsupported states
Choose or rank models, apply job or privacy policy, route, infer prices, equate offering IDs, hold route bindings
Deal Picker (this repo)
Job policy and labels, route registry, evidence bindings, privacy evidence records, substitution mappings, eligibility, ranking, budget evaluation (a reservation ledger only from tranche BL), execution plans, decision records, derived cost/success analysis
Parse provider catalogs, reinterpret MPW prices, hold or log secrets, call models, run tools
Model Council Lab (optional)
Scoped quality/trust evidence for named task class, model version and configuration
Suppress deal discovery, approve privacy, approve spending
Hermes Agent
Credentials, requests, retries, runtime fallback, tools, per-attempt usage and outcome events
Change as part of this repo; any Hermes extension is a separately reviewed change
Human Chair (Amy)
Job policies, labels, route registry entries, privacy verifications, budget envelopes, privacy-boundary exceptions
—
MPW and MCL only inform; the operator's files decide eligibility. Hermes's per-attempt records come back to the picker to be reconciled against the plan.
1.3 Core objects
Eight objects carry the design. The first two answer Astra finding 1 directly: what MPW observed is never the thing Hermes calls.
OfferingEvidence — produced by MPW, read-only to the picker.
• Identity: evidence_ref = provider stream + exact offering ID + snapshot ID + observed-at + interpretation-policy version. Exact string match only; no case-folding or normalization.
• Time: evaluated_at (the instant asked about), presence (in latest successful observation, absent from it, never observed) and the stream's last successful refresh time.
• Price components: one entry per billing dimension (input, output, cached input, cache write, reasoning, per request, other named). Each has an amount or null, currency, unit, conditions (batch, tier, time window), support status (supported, unsupported, unknown) and basis (provider-declared or a curated rule with its verified-at date).
• Free execution: confirmed (with basis and conditions such as rate limits), not established, or not free. Zero-valued components alone always yield not established.
• Promotion and time window: declared promotion with valid-until (or none declared, or unknown); whether a time window applies at evaluated_at, with its bounds.
• Diagnostics and contract_version.
• Rules: Cheaper Inference v2 evidence carries price support unsupported. MPW "current" means present in the latest stored successful observation, not freshly verified upstream. MPW's change-event windows are not price guarantees. The record carries no endpoint, privacy, capability or routing claim.
ExecutableRoute — an operator-approved entry in the picker's route registry.
• route_id (opaque, stable), route_version, status.
• Hermes target: provider, model ID exactly as sent (case preserved), base URL, credential_scope_ref (the name of a Hermes-side credential pool or profile, never a secret), service tier, fully explicit request overrides, reasoning effort, maximum output tokens.
• Aggregator routing (required when the provider is an aggregator such as OpenRouter): ordered downstream-provider allowlist, denylist, whether the aggregator may fall back internally, data-collection setting, ZDR requirement and per-dimension maximum price. Each is expressed as the provider's documented request field; the field mapping is verified in tranche 2.
• Capabilities: declared tool use, structured output, context window and modalities, each with source and date.
• Privacy: references to PrivacyEvidence records. The route's maximum sensitivity is computed from them against policy, never declared directly.
• Locality: remote or local. Local routes still need explicit capability, privacy and availability inputs.
• Identity: the hash of the full record. Two routes differing in any field are different routes. An OpenRouter route with downstream constraints and the same model without them are two routes.
PrivacyEvidence — operator-verified, maintained in the picker.
• Subject: a route plus, for aggregators, one record per allowed downstream provider. The route's privacy is the weakest of them.
• Claims: training use (no, yes, unknown); prompt retention (none, bounded with duration and purpose such as safety review, indefinite, unknown); ZDR (strict, with safety retention, none, unknown); human review; for local routes, where logs are written.
• Provenance: source (policy URL, account-setting record, local configuration), verified by, verified at, expires at (at most 90 days, earlier if the source states a change date).
• An expired or missing claim is unknown. A single "private" flag is not accepted.
EvidenceBinding — the validated link between one route and the MPW evidence that prices it. Bindings live in the picker's registry; MPW holds none.
• binding_id, version, route_id, evidence selector (provider stream + exact offering ID).
• Coverage — what the evidence proves about this route's charges:
    ◦ exact: the operator attests, citing provider documentation, that the observed offering price is what this route is billed. This informs estimates only; it never guarantees a future charge or replaces the enforceable bound (1.7);
    ◦ bounded by route max price: the evidence informs ranking, and the route's provider-enforced maximum price is the budgeting bound;
    ◦ informative only: the route's cost is unknown for budgeting.
• Free applies: carried only when MPW reports free execution confirmed and the route's target matches that offering's exact identity and conditions. Never inferred.
• Approved by, approved at, expires at (default 30 days).
• Automatic invalidation: the offering is absent from the latest successful observation, the interpretation policy changes major version, or the route version changes.
• Local routes have no binding. Their commercial basis is operator-declared local: no API charge, labelled as such and never presented as a confirmed free deal.
SubstitutionMapping — operator-approved, in the picker.
• Directional (from model + configuration to route), scoped to named capabilities, model versions and request settings, versioned, with rationale, approver and expiry.
• Grants only "this route may stand in for that model for these capabilities". It transfers no privacy, trust or price evidence; the substitute passes every gate on its own.
• Independent fallbacks need no mapping: a job policy's candidate set can list several acceptable routes without claiming they are equivalent.
• MPW does not currently establish a same-model comparison identity. If it later does, that would be pricing-comparison evidence, citable in a mapping's rationale but never creating one. Nothing in tranche 1 depends on it.
JobPolicy — operator-owned; section 1.4.
BudgetSnapshot — an immutable input describing simulated available budget at the evaluation instant.
• Currency, and the envelope and policy version it reflects.
• Remaining allowance per job, per day and per month.
• Obligations already held (provider-reported or uncertain), already deducted from those allowances.
• In tranche 1 it is a fixture: the selector reads it and never changes it. From tranche BL onward, the ledger produces it from real state.
ExecutionPlan or Refusal — the picker's only outputs.
• Plan (a selection proposal): decision ID; job policy ID and version; registry snapshot hash; evidence references; effective labels and how they were derived; the chosen route with its fully resolved Hermes target; an explicit fallback list (always empty in v1, section 1.6); limits; required_reservation (the worst-case amount), budget_snapshot_ref and simulated_budget_result (fits or not, and the hypothetical remaining budget); valid_until; rationale; every excluded candidate with reason codes; every assertion still unverified, such as the Hermes profile snapshot.
• Refusal: decision ID, typed reason codes, and the per-route exclusion reasons that led to it.
• No spending authority. A tranche-1 plan says the snapshot could accommodate the bound, not that money is reserved. A real reservation_id exists only when tranche BL issues one, and no paid live run may start without it.
1.4 Policy authority, sensitivity and privacy
The operator sets every label and limit; models, tasks, tools and delegated calls can only narrow them.
Authority. Amy, as Human Chair, authors job policies, context-source labels, tool labels, the route registry, privacy verifications, substitution mappings and budget envelopes. They are versioned files in the picker's policy store. A change takes effect only as a new version, and every decision records the versions it used. A model-authored change to these files needs Chair approval through normal review.
Hints. A task or delegated call may pass hints, such as a needed capability. Hints can only remove candidate routes. A hint that would lower a label, add a route, relax privacy or raise a limit is ignored and recorded in the decision.
Sensitivity labels (defaults for Chair confirmation):
Label
Typical content
Route requirement
S0 Public
Public web and market data; operator-written task text with no personal detail
Any approved route; training use allowed only if the job policy says so
S1 Internal
Amy's non-public project material without personal or financial detail
Proposed: no training use, bounded retention, current privacy evidence. Needs Amy's explicit decision before any use beyond S0
S2 Private
Conversation history, memory, portfolio and holdings, personal uploaded files
Strict ZDR on every downstream provider, or a local route with verified logging
S3 Restricted
Credentials, account numbers, legal, health or identity documents
Local only, or a specific Chair exception
Consequentiality (set per job class by the operator): C0 exploratory or scratch; C1 informs Amy's decisions or published writing, such as a market report; C2 triggers or authorizes external action, such as trades, email, file or repository changes, payments. C2 is outside every tranche in this document. A C1 job policy may require scoped MCL trust evidence.
Effective sensitivity is the maximum of the job's floor, the label of every context source the run can include, the label of every tool it can call, and the parent session's label. Default source labels: parent conversation history S2, memory S2, uploaded files S2 unless labelled per file, portfolio data S2, project files S1, public web fetch S0, any unlisted source S3. Missing classification resolves to S3 and C2, which refuses on any route without S3 approval.
High-water mark. Sensitivity can rise during a run and never falls. A tool result above the route's maximum blocks the next transmission and stops the job with sensitivity_escalated. Re-planning onto a route approved for the higher label is a new decision, and only for jobs with no side effects.
Detection before transmission. The check must run before each request leaves Hermes, and Hermes has no established hook for it. v1 therefore enforces it structurally:
1. The pilot runs in a dedicated Hermes profile (or equivalent isolated configuration) whose possible inputs and tools are fixed at S0: no memory, no inherited history, no uploads, read-only public tools.
2. The picker computes the effective label from that declared configuration and refuses if any input or tool is undeclared.
3. The child's toolset is restricted so a higher-labelled tool cannot be called, so tool output cannot escalate within v1.
These are declared assertions, and offline output marks them unverified. Before live entry the pilot must show that the fetch tools cannot reach authenticated, local or private endpoints, that nothing inherits context, memory or files, and that the snapshot matches the configuration actually run. Running above S0 live requires a Hermes pre-request policy hook (H-2), which may prove necessary at S0 too (1.11).
Auxiliary inference. Every inference path a run can trigger is a route and passes the same gates: context compression, summarization, vision, web extraction, title generation and credential rotation. v1 validates a declared snapshot of the pilot profile's auxiliary configuration; any auxiliary path not pinned to an approved route, or not declared, causes refusal.
Logs and telemetry. Picker records hold IDs, labels, hashes, costs and reason codes — never prompts, outputs, tool payloads or secrets. Hermes's own session logs stay local under Hermes's controls and inherit the job's label. The pilot profile exports no telemetry to third parties.
Tool outputs crossing the route boundary. A tool result becomes part of the next request, so it crosses to the route. It may do so only when the tool's label is at or below the route's maximum. In v1 only S0 tools are available.
Privacy evidence upkeep. The operator re-verifies each record before it expires (90 days at most, earlier on a stated policy change). An expired record makes its claims unknown, and the route drops to whatever an unknown-privacy route may carry under policy (S0 at most).
Chair exceptions. Crossing a privacy boundary needs a ChairException naming one run, one route, the specific data sources and the reason. Only Amy can create one; the decision record cites it. The pilot allows no exceptions.
1.5 Eligibility and ranking
Every route passes every mandatory gate before any ranking happens; the gate order affects efficiency only.
1. Resolve policy and labels, or refuse.
2. Candidate set: the job policy's route allowlist plus routes reached through valid substitution mappings.
3. Mandatory gates, each recording a reason code when it excludes a route:
    ◦ binding valid (or local basis declared);
    ◦ capability fit;
    ◦ privacy: route maximum sensitivity at or above the effective label;
    ◦ validity and freshness: evidence inside its cache interval, promotion not expired, any time window covering the worst-case run;
    ◦ trust, when the job policy requires it;
    ◦ availability, where known (an unavailable local route is simply absent);
    ◦ budget: an independent, complete, enforceable worst-case bound exists, and the BudgetSnapshot (tranche 1) or the ledger (tranche BL onward) can accommodate it;
    ◦ Hermes representability (section 1.6).
4. Rank the survivors.
5. Emit a plan for the top route, or a refusal when none survive.
Discovery is never suppressed. Every MPW offering considered appears in the decision record, including those excluded for missing trust evidence. MCL can constrain a job; it cannot hide a deal.
Freshness defaults are cache and recheck intervals, not guarantees of price stability. The picker may tighten MPW's own stale thresholds, never loosen them, and a failed MPW refresh does not extend freshness.
Evidence kind
Default recheck
Free, promotional or launch deal
At job start, never older than about 24 hours
Time-window pricing
Applicability checked at evaluation time and over the worst-case run
Known expiry
Never relied on after expiry
Stable list price
About 7 days
Ranking compares estimated job cost, not a token rate.
1. Cost class, in default order (a job policy may reorder):
    1. confirmed free deal — MPW free execution confirmed, carried by a valid binding;
    2. operator-declared local route (Mac Mini, then DGX Spark, by Amy's stated preference);
    3. paid route with a complete estimate;
    4. paid route with an incomplete estimate, flagged "comparison incomplete".
2. Within a class, expected job cost = the sum over billing dimensions of price × projected usage, times (1 + expected retry factor). Projected usage (input/output mix, cache-hit ratio, reasoning tokens, batch eligibility) comes from an operator-supplied usage profile per job class. Any dimension without a supported price makes the estimate incomplete.
3. Tie-break by operator preference rank, then route_id.
Estimate completeness is not bound completeness. The expected-cost estimate ranks routes; the worst-case bound decides whether a paid route may run. A route with an incomplete estimate but a complete enforceable bound (a provider-enforced cap on every billable dimension, plus capped usage) stays eligible, ranks in class 4 and is budgeted on its bound. A paid route without a complete enforceable bound is excluded, whatever its estimate.
Locality earns no rank by itself: a local route still needs declared capability, privacy and availability.
Temporary free routes. A route whose binding carries confirmed free execution is a legitimate free deal while it lasts. When MPW evidence shows the promotion has ended, the deal and the binding's free applies expire. The route's approval and the model's trust stay as they were, and the route re-enters ranking at its paid cost if a paid binding exists. Because current MPW evidence never establishes free execution, the first tranche may return no confirmed free candidates. That is a correct result.
1.6 Execution contract with Hermes
v1 makes every inference attempt a single-route plan with an explicitly empty fallback, so nothing Hermes normalizes or inherits can widen what was approved.
Terms. The selected route is the one ExecutableRoute in a plan. A child run is one delegated execution under that plan. An attempt is one inference request. A single child run may make many attempts on its route (agent turns, SDK retries, credential attempts) plus auxiliary requests, so limits apply per attempt, per child run and per logical job (1.7).
route_task() selects a route; it does not enforce policy. The Hermes adapter translates a plan into a route_task() return under these rules:
1. Field fidelity. Map only to fields Hermes is verified to honour: model, provider, base URL, credential resolution, request overrides, reasoning effort, fallback providers. A route attribute with no honoured field makes the route unrepresentable; it is excluded, and if it was the top route the decision refuses. The adapter never adds extra keys expecting enforcement, because Hermes discards unknown keys.
2. No collapse under normalization. The adapter computes Hermes's fallback deduplication key (provider, lowercased model ID, lowercased base URL) only for collections Hermes will actually normalize: in v1, the submitted route together with any fallback entry Hermes could add from the pilot profile. Two distinct entries in such a collection sharing a key, including model IDs differing only by case, make the plan unrepresentable; it is refused, never merged. Unselected registry candidates that share a key with each other stay distinct and are not rejected.
3. Explicitly empty fallback. Every v1 plan sets fallback to an empty list. The pilot profile also has no global fallback chain, and its credential pool contains only the approved scope, so rotation cannot reach another account.
4. Empty eligibility refuses. There is no default model.
5. Picker-level fallback. When an attempt fails with a category that permits reselection (section 1.8), the next route is reached through a new decision: revalidated, budget re-evaluated (and, from tranche BL, re-reserved), again single-route.
6. Price controls. A paid route may run unattended only if its provider-side maximum price is representable in the request it actually sends.
7. Detection. Every Hermes attempt record is reconciled against its plan. An attempt on any unplanned route, account or downstream provider is a contract_violation: the pilot halts and the incident is recorded.
This gives prevention by construction (single route, no inheritance, restricted credentials and tools) plus detection by reconciliation. Per-attempt prevention for longer in-run chains needs Hermes changes H-1 and H-2.
Verification items — to check against Amy's installed Hermes before tranche 3; a failed item blocks live use until resolved.
ID
Behaviour to verify
If it fails
V1
Which route_task() return fields are honoured, and how a credential reference resolves to a pool
Restrict the adapter to the honoured subset, or refuse
V2
An empty fallback_providers list means no fallback, not "inherit global"
Hard gate: an empty global chain counts only if every inheritance and default path is shown absent; otherwise H-1
V3
Whether fallback entries carry their own request overrides and provider routing
Stay single-route (already the v1 design)
V4
Request overrides reach the provider request body unchanged, including OpenRouter provider routing, maximum price, data collection and ZDR fields
Aggregator routes are unrepresentable; refuse them
V5
What a delegated child inherits (history, memory, files) and how its toolset is restricted
Delegated pilot is blocked
V6
Which auxiliary inference paths a child can trigger and how each is routed
Refuse until each is pinned or disabled
V7
Which per-attempt events Hermes emits (route, downstream provider, usage, cost, error detail)
Live use blocked; a reduced-audit pilot would need a separate, explicit scope decision
V8
What route_task() receives: can it identify the calling profile, job or parent
A fixed profile fixes policy but not job identity; without a stable logical job ID across restarts, live use is blocked
V9
Credential rotation within a pool stays inside that pool
Hard gate: one credential suffices only if resolution cannot reach any other scope or default source
1.7 Economic contract
Paid live execution is allowed only when a provider-enforced worst case fits a real reservation already taken from Amy's envelope (tranche BL); otherwise the picker refuses. Tranche 1 only evaluates whether that worst case would fit a supplied snapshot.
Decision validity. valid_until is the earliest of: decision time plus a plan TTL (default 15 minutes), any evidence cache expiry, any promotion's valid-until, and the expiry of any binding, privacy record or mapping used. A run may not start after valid_until. Each retry-by-reselection is a new decision, so revalidation is automatic.
Long runs and time windows. A window active at job start proves nothing about the end. The plan's deadline is a latest-start time. The validity gate prices the job at the worst rate applicable at any point from decision time to that deadline plus maximum elapsed time, so a delayed start is covered; a running job stops at its elapsed-time limit. If that rate is unknown or over budget, the route is ineligible for this job.
Provider-side price protection. Where a provider enforces a maximum price (for example OpenRouter's documented max-price routing control), the route sets it, and that bound feeds the reservation. A local estimate is never treated as a cap.
Budget evaluation (tranche 1). The selector compares the route's worst-case bound with an immutable BudgetSnapshot and reserves nothing.
• Worst-case bound per job: for each billable dimension, the provider-enforced maximum price × the maximum usage the job's limits allow, plus per-request fees, × maximum attempts. Every billable dimension needs both an enforceable price cap and a usage cap; a missing one means no bound, and a paid route without a bound is refused.
• Output: required_reservation, budget_snapshot_ref and simulated_budget_result (fits or not, and the hypothetical remaining allowance). The selector refuses with budget_insufficient when the bound exceeds any remaining allowance.
• No mutation: evaluation changes no state, and repeating it gives the same answer.
• Envelopes: per job, per day and per month. The default unattended paid envelope is $0 until Amy sets values, so only confirmed-free and local routes can pass unattended.
• Limits per job: maximum output tokens per attempt, maximum attempts (Hermes retries and reselections together), maximum elapsed time, maximum job spend (equal to the bound). None can be raised by the run.
Budget ledger (tranche BL, later). A separately scoped stateful tranche with its own interface and its own review, required before any paid live execution. It owns:
• persistent reservation transactions that issue a real reservation_id;
• atomic concurrency, so two jobs cannot reserve the same remainder;
• defined semantics for repeated reservation requests (for example, idempotent by request key);
• holds for provider-reported and uncertain charges;
• ingestion of provider-reported cost, reconciliation, and release or write-off of obligations (write-off by Amy only);
• producing the BudgetSnapshot that selection reads.
Tranche 1 contains none of this, directly or indirectly.
Obligations outlive plans. Once an attempt may have been billable, its reported or uncertain obligation stays held until reconciliation resolves it. A pricing change, an expired decision or a revoked plan ends permission for further execution; it never erases or releases an obligation already incurred. Tranche 1 models obligations only as amounts already deducted in the BudgetSnapshot.
Five cost figures stay distinct: estimated (ranking only), required or reserved, provider-reported, reconciled and uncertain.
Live-entry gate L1 (not a tranche-1 blocker). Before any live run, paid or free, show that each of these is enforced, not just declared:
• request count per child run and per logical job, including internal retries and auxiliary calls;
• input and context growth across turns;
• output and reasoning-token limits on every request;
• auxiliary calls, routed and counted;
• the parent invocation's route, privacy and cost, inside the same job budget;
• cumulative reselections across the logical job;
• maximum elapsed time, with a running job stopped at its limit.
If existing Hermes mechanisms cannot enforce these at S0, H-2 is needed at S0 too.
1.8 Failure semantics
Each failure is classified only on supporting evidence; anything ambiguous is unknown, and unknown stops.
Category
Classify only when
Action
Effect on route or deal
Route unavailable
Error detail shows connection failure, timeout before response, or a server error
Bounded Hermes retry on the same route, then a new decision excluding it for a cooldown
Route unavailable for the cooldown
Promotion ended
MPW evidence shows the promotion gone or expired, or the provider error says so explicitly
New decision, with a fresh budget evaluation at the paid rate
Deal and free applies expire; approval and trust unchanged
Quota exhausted
Provider error identifies a rate limit or quota
No immediate retry; record route, credential scope and any reset time; new decision excluding that scope until reset
Scope on cooldown
Pricing changed
MPW evidence shows a change, or a price cap rejects the request
Stop; invalidate the old estimate and revoke further execution under the old plan; reassess as a new selection and budget evaluation. Incurred or uncertain obligations stay held
Binding needs revalidation
Charge exceeds enforced bound
A provider-reported charge is above the bound the route enforced
Contract violation: halt the pilot; no automatic reselection; obligation held at the reported amount
Route suspended pending operator review
Authentication
Credential rejected
Stop; no retry; alert the operator
Route suspended pending operator
Configuration
Invalid model ID, base URL or parameter
Stop; no retry
Route suspended pending operator
Policy
Provider refuses on data policy, or a picker pre-check fails
Stop; never fall back to a weaker route
None
Capability or API incompatibility
API rejects a required feature (tools, schema, context length)
New decision
Route ineligible for that requirement only
Task-quality failure
API succeeded but the job's acceptance check failed
Record; a new decision only if the job policy allows retry on quality
Quality signal to audit or MCL; not a route failure
Unknown
Anything else
Stop; keep sanitized diagnostic detail; v1 has no approved conservative retry rule
None
An HTTP status alone does not establish a commercial cause such as a promotion ending.
Cause and billing are separate facts. Every failed attempt records why it failed and, independently, whether it may have been billed: billed, not billed or uncertain. A timeout before a response is a route-unavailable failure and also potentially billable, because it does not prove the provider did no work. It is recorded as both, and its obligation stays uncertain until reconciled.
Retry versus restart. An inference retry repeats the same request on the same route inside Hermes, within the attempt limit. A job restart is a new child run through a new decision. Restart is allowed only for jobs with no side effects; v1 contains no externally mutating tools, so restart stays within max_attempts. A partial or interrupted response is discarded and its billing is uncertain.
API success is not task success. Each job class has an operator-defined acceptance check, for example required report sections, sources present and schema valid.
1.9 Audit contract
Hermes originates execution facts; the picker originates decision facts and derives analysis from both.
• DecisionRecord (picker): decision ID and time; policy and registry versions; evidence references; effective labels with derivation; every candidate with its gate results; ranking inputs and cost completeness; chosen route; fallback list; limits; required reservation and simulated budget result (a reservation ID only from tranche BL); valid_until; rejected hints; any Chair exception.
• AttemptRecord (Hermes-originated; the picker ingests it, never authors it): attempt ID and decision ID; actual provider, model, base URL, credential scope and, where reported, downstream provider; start and end; status, failure cause, billing status (billed, not billed or uncertain) and sanitized error detail; usage per dimension; provider-reported cost; retry index; any fallback or rotation activation with from and to.
• JobOutcome (picker-derived): every decision and attempt in the job; API success; task success with acceptance-check version; the five cost figures; contract violations.
No record holds prompts, outputs, tool payloads or secrets. Provider diagnostics are sanitized before storage: only error codes, provider error types and bounded, scrubbed messages are kept. A final route plus a fallback flag is not a valid substitute for per-attempt records.
1.10 Cron and top-level routing
Cron routing already works; what is missing is a way to give a pinned cron job the picker's chain without breaking pinned-job semantics.

Option A: Hermes per-job fallback field
Option B: fixed cron parent delegates to a picker child
Hermes change
Yes (H-3), separately reviewed
None expected, conditional on V1–V9
Parent model cost
None
One small parent run per job, reserved in the same job budget
Parent privacy exposure
None
Parent sees the cron prompt and the child's result; acceptable only when both are S0
Failure dependency
None
Job fails closed if the parent's pinned route fails
Context inheritance
Not applicable
Depends on V5
Audit and budget
One run
Two levels, joined by job ID; needs V8 or a per-profile envelope
Recommendation. The first live pilot is a manually triggered delegated job, not cron. When a scheduled pilot follows, use Option B: it should need no Hermes change, conditional on V1–V9, and reuses the one integration the picker already depends on. The parent runs on a fixed local or capped route with no fallback, so it fails closed. Pursue Option A as a separate Hermes proposal for the long term, with: absent field keeps today's behaviour; empty list means no fallback; a valid list authorizes only those routes; identical enforcement at startup, preflight and mid-run.
Recurring jobs store job policy, never a chosen route. The route is selected at each execution, so a temporary deal is never pinned.
Top-level routing is out of scope for v1. It needs supported per-invocation route construction before the first model call, covering retries and auxiliary calls (H-4). Rewriting global configuration is rejected.
1.11 Dependencies outside this repository
Each is a separately scoped, separately reviewed change. None is implemented here.
ID
Owner
Change
Needed for
D-MPW-1
MPW
Versioned evidence interface: evidence for explicit offering selectors at a given evaluation time, returning OfferingEvidence v1. A natural fit for MPW's planned api module; a model_deals() view can sit on top later
Live MPW data (tranche 3)
D-MPW-2
MPW
Read-only database opening path that never creates or migrates a database
Any picker read of a real MPW database
D-MPW-3
MPW
Authoritative sources for confirmed free execution
Any free deal at all
H-1
Hermes
Per-entry request settings in fallback entries; strict empty-list semantics
In-run chains longer than one route
H-2
Hermes
Pre-request policy hook: sensitivity, route set, validity and budget checked per attempt
Live runs above S0; in-run fallback; possibly S0 too, if existing mechanisms cannot enforce validity, request limits or budgets
H-3
Hermes
Per-job declared fallback chain for pinned cron jobs (Option A)
Cron without a parent
H-4
Hermes
Per-invocation route construction for top-level runs
Top-level routing
2. Resolution of Astra's blocking findings
None of the six findings remains blocking for the offline tranche; two have live-use dependencies that are now named, scoped and kept out of this repo.
#
Finding
Status
How
1
Offering evidence is not an executable route
RESOLVED IN ARCHITECTURE
OfferingEvidence and ExecutableRoute are separate objects joined only by an operator-approved, expiring EvidenceBinding in the picker (1.3). Aggregator downstream routing is part of route identity, and privacy is per downstream provider. Missing endpoint, privacy or free-status evidence stays unknown and makes the route ineligible where it matters.
2
Privacy needs an authority across the whole execution
RESOLVED IN ARCHITECTURE for S0; DEPENDENCY REQUIRES SEPARATE CHANGE above S0
Operator-owned labels, conservative defaults, monotone high-water mark, auxiliary calls and logs under the same gates, Chair exceptions per run (1.4). v1 detects violations before transmission structurally, by fixing inputs and tools at S0. Live runs above S0 need a Hermes pre-request hook (H-2).
3
route_task() selects; it does not enforce
RESOLVED IN ARCHITECTURE; live use gated on V1–V9
Single-route plans with explicit empty fallback, picker-level reselection as new decisions, refusal on unrepresentable fields or normalization collisions, and reconciliation of every Hermes attempt (1.6). Each assumed Hermes behaviour is a named verification item; longer in-run chains need H-1 and H-2.
4
Freshness is not a spending cap
RESOLVED IN ARCHITECTURE
Freshness is demoted to cache policy. Spending is bounded by decision deadlines, worst-rate pricing across the run's maximum duration, provider-side price caps, and worst-case bounds evaluated against an immutable BudgetSnapshot in tranche 1, then an atomic reservation ledger with held obligations in the separately reviewed tranche BL before any paid live run (1.7). No bound means refusal. Default unattended paid envelope is $0.
5
Failure handling needs typed semantics
RESOLVED IN ARCHITECTURE
Ten categories, each classified only on evidence, with unknown stopping (1.8). Inference retry and job restart are separate; restart is allowed only without side effects; API success and task success are recorded separately.
6
Cron fallback integration
DEPENDENCY REQUIRES SEPARATE CHANGE (Option A); not blocking v1
Cron is outside the first releases. Option B is recommended for the first scheduled pilot because it needs no Hermes change; Option A is kept as a separate Hermes proposal (1.10).
Astra's non-blocking recommendation is adopted: the picker's foundation is a versioned evidence interface for explicit offering selectors, with any deals view layered on top (D-MPW-1), plus a deliberate read-only open path (D-MPW-2).
Astra's second review raised one further blocker, B1: the tranche-1 budget contract mixed a pure selector with a stateful ledger. It is resolved in sections 1.7 and 4 and summarized in section 7.
3. Frozen decisions
Each decision deliberately limits the first release; widening any of them is a new architecture decision.
1. Pilot scope
• Execution type: delegated child work only, through DelegationProvider.route_task(). Cron and top-level routing are excluded.
• Trigger: manual, from a dedicated Hermes profile used for nothing else.
• Hermes version: the exact release or commit installed on Amy's machine, recorded at the start of tranche 2 in a HERMES_COMPAT.md file and re-checked against V1–V9. The offline tranche needs no Hermes at all.
• Allowed inputs: operator-written task text and public web data. No conversation history, memory, uploads, portfolio data or project files.
• Allowed tools: read-only public web search and fetch. No tool that writes, sends, trades, edits files or calls other services.
• Labels: S0, at most C1.
2. Evidence contract — OfferingEvidence v1 (1.3).
• Identity: provider stream + exact offering ID + snapshot ID + observed-at + interpretation-policy version; exact string match.
• Interpretation authority: MPW alone. The picker never derives a price, a free status or an equivalence.
• Unknown states: explicit per price component, per free status and per promotion. Absent means unknown, never zero.
• Timestamps: observed-at, evaluated-at and last successful stream refresh, all UTC.
• Versioning: contract_version on every record. The picker refuses any major version it does not know.
3. Route contract — ExecutableRoute, PrivacyEvidence, EvidenceBinding and SubstitutionMapping (1.3).
• Endpoint binding: an exact Hermes target per route; a binding to MPW evidence is required for any paid route.
• Account scope: a named Hermes credential pool reference only; the registry rejects any field that could hold a secret.
• Privacy controls: per-downstream-provider claims with provenance and expiry; aggregator routing fields are part of route identity.
• Request and routing settings: fully explicit in the record and hashed into route identity.
• Substitution: directional, scoped, expiring mappings that transfer nothing.
4. Policy authority
• Sensitivity and consequentiality: S0–S3 and C0–C2, set by the operator per job class and per context source (1.4).
• Trust: required only where a job policy names it, scoped to task class, model version and configuration.
• Approval boundaries: only Amy changes policy (as a new version) or grants a per-run privacy exception; the pilot allows none.
• Refusal behaviour: missing labels resolve to S3/C2; an empty eligible set refuses; every refusal carries typed reason codes.
5. Execution enforcement
• Revalidation: every plan has valid_until (default TTL 15 minutes); every reselection is a new decision.
• Budgets: tranche 1 evaluates the worst-case bound against an immutable BudgetSnapshot and reserves nothing. Atomic reservations against per-job, daily and monthly envelopes come only with tranche BL, before any paid live run. Unattended paid envelope is $0 until Amy sets it; attended paid runs also need a provider-side bound and an attended envelope Amy sets.
• Retries: Hermes retries on the same route only, counted against max_attempts.
• Fallback: explicitly empty in every v1 plan; reselection only through the picker.
• Uncertain charges: held at the full bound until reconciled or written off by Amy (tranche BL); a pricing change never releases them. Tranche 1 sees them only as amounts already deducted in the BudgetSnapshot.
• Runtime enforcement: structural prevention plus reconciliation; any contract violation halts the pilot.
6. Audit contract
• Decision records: written by the picker for every plan and refusal.
• Per-attempt records: originated by Hermes and ingested, never authored, by the picker.
• Cost uncertainty: estimated, reserved, provider-reported, reconciled and uncertain kept as separate fields.
• Task-success criteria: an operator-defined acceptance check per job class, versioned, recorded separately from API success.
4. First implementation tranche: offline selection only
Tranche 1 is a pure, deterministic selector. It performs no persistent mutation: no network, no Hermes runtime, no live MPW database, no ledger and no financial state change.
Inputs
• OfferingEvidence fixtures in the proposed v1 consumer contract, without asserting that MPW has implemented or approved it. Some are mapped from captured outputs of MPW's existing selected-offering queries at a pinned commit; mapping preserves absent and unsupported fields and adds no pricing authority. Confirmed-free cases are synthetic and labelled synthetic. The picker opens no MPW database.
• Operator registry files: routes, privacy evidence, bindings, substitution mappings, context-source and tool labels.
• Job policy files.
• A simulated BudgetSnapshot (section 1.3), treated as ordinary immutable input.
• The evaluation time.
• A declared Hermes capability profile: honoured route_task() fields and the normalization key, as Astra read them upstream, marked unverified.
• A declared pilot-profile snapshot: inputs, tools, auxiliary paths, global fallback chain, credential pools, marked unverified.
Outputs: an ExecutionPlan proposal or a Refusal as canonical JSON, plus a DecisionRecord, returned to the caller; the selector writes nothing. A thin CLI wraps one function, for example picker select --job <file> --budget <snapshot> --at <time>.
Modules: evidence contract and fixture mapper; registry loader and validator; policy and label resolution; eligibility gates; ranking and cost estimate; budget evaluation against the snapshot; plan builder with the Hermes representability and collision checks; decision records.
Tests — each must produce the stated outcome with a typed reason code:
Case
Expected result
Stale evidence
Route excluded: stale
Unknown price component, no enforceable bound
Paid route excluded: no bound
Unknown price component, complete enforceable bound
Eligible; ranked as comparison incomplete; budgeted on the bound
Cheaper Inference v2 evidence
No price; route excluded or informative only
Zero prices without confirmed free execution
Not ranked as free
Synthetic confirmed-free fixture
Ranked as a free deal; output marks the evidence synthetic
Missing privacy evidence on an S1 job
Route excluded
Expired route binding
Route excluded
Expired substitution mapping
Substitute not a candidate
Empty eligible set
Refusal
Explicitly empty fallback
Plan fallback list is empty; a profile snapshot with a global chain refuses
Submitted route collides with a profile fallback entry under Hermes normalization, including case-only model differences
Refusal, never merged
Two unselected candidates share a normalization key
Both stay distinct candidates; no refusal
BudgetSnapshot with insufficient remaining allowance
Refusal: budget insufficient
BudgetSnapshot already reduced by an uncertain prior obligation
Selector sees the lower allowance and may refuse
A series of BudgetSnapshots representing states after hypothetical reservations
Each gives its own deterministic result; real concurrency is not tested here
Any plan
Carries required reservation, snapshot reference and simulated result; no reservation ID
Sensitivity downgrade hint
Ignored and recorded
Hint that adds a route or raises a limit
Ignored and recorded
Missing classification
Resolves to S3/C2; refusal
Time window ending before the plan deadline plus maximum elapsed time
Priced at worst rate or excluded
Promotion expiring before the plan TTL
Deadline shortened to the expiry
OpenRouter route without downstream constraints on an S1 job
Excluded
Undeclared auxiliary path in the pilot snapshot
Refusal
Required trust evidence missing
Excluded, still listed in discovery
Unavailable local route
Absent from the plan
Registry entry with a secret-shaped field
Load fails
Same inputs, including identical BudgetSnapshot and evaluation time, run twice
Byte-identical output; no state changed
Acceptance: all tests pass; a test transport fails on any network use; a test filesystem fails on any write outside the test's own output capture; no module imports Hermes or opens a database; golden JSON for plan, refusal and decision record; every refusal has at least one reason code.
After tranche 1, in order, each separately reviewed and approved:
1. Astra reviews the offline core.
2. Tranche 2: a simulated Hermes delegated adapter against a fake that models upstream semantics, plus read-only checks of V1–V9 on Amy's installed Hermes.
3. Tranche BL: the stateful budget ledger (section 1.7), with its own interface and review. It can proceed alongside tranche 2 and is required before any paid live run.
4. Independent review of the runtime-enforcement assumptions, including live-entry gate L1.
5. Only then, a separately approved, manually triggered public-data pilot (tranche 3), which also needs D-MPW-1 and D-MPW-2, and tranche BL if it admits any paid route.
5. Remaining open questions
Five questions need evidence or an upstream decision; none blocks tranche 1.
1. Installed Hermes behaviour. Which version Amy runs, and whether V1–V9 hold on it. Answered by read-only inspection in tranche 2.
2. MPW scheduling of D-MPW-1 and D-MPW-2. The Human Chair decides whether the evidence interface and read-only open path join MPW's current roadmap (for example alongside its planned api module) or wait. Needed before tranche 3.
3. Where confirmed free execution can come from (D-MPW-3). Which provider sources MPW could interpret authoritatively as free, with conditions such as rate limits. Without one, the picker will find no free deals, which is correct but defeats its commercial purpose. Needs MPW-side research.
4. Where charge reconciliation lives. Reconciled charges need credentialed provider lookups. MPW defers credentialed providers, and the picker holds no credentials, so the likely home is the Hermes adapter. Needs an Astra and Chair decision before any paid pilot.
5. Amy's envelope values. Per-job, daily and monthly ceilings for attended and unattended paid runs. A Chair input, not an architecture question; until set, paid execution is refused.
6. Repository recommendation
Confirmed: a new standalone hermes-deal-picker repository. Nothing in this revision argues for putting it inside MPW or Hermes. The picker owns operator policy and route bindings, which MPW must not hold, and it must stay independent of Hermes so that every Hermes extension is reviewed on its own.
Proposed layout:
hermes-deal-picker/
  README.md            this architecture
  HERMES_COMPAT.md     pinned Hermes version and V1–V9 results (tranche 2)
  contracts/           OfferingEvidence v1, plan, refusal and record schemas
  src/deal_picker/
    evidence/          MPW adapter and fixture mapper (no MPW parsing)
    registry/          routes, bindings, privacy evidence, mappings, labels
    policy/            job policy and label resolution
    select/            eligibility gates and ranking
    budget/            budget evaluation (ledger in tranche BL)
    plan/              plan builder, Hermes representability and collision checks
    hermes_adapter/    route_task() translation (tranche 2)
    audit/             decision records, attempt ingestion, job outcomes
  policy-examples/     non-secret example registry and job policies
  tests/
It does not contain MPW parsers or price semantics, a Hermes inference runtime, or any part of MCL. Python 3.11, matching MPW, with no runtime dependencies in tranche 1.
7. Change from Rev 2
The only blocking architecture change was: Tranche 1 is now a deterministic selection-only operation using an immutable simulated BudgetSnapshot; persistent reservation/accounting is deferred to a separately reviewed stateful tranche.
Edits made only to keep the document consistent with that change:
• BudgetSnapshot added as an object (1.3). Plans carry required_reservation, budget_snapshot_ref and simulated_budget_result, never a reservation ID.
• The ledger moved to tranche BL (1.7, 4). Obligations outlive plans, so a pricing change revokes further execution but releases nothing (1.7, 1.8).
• Tranche 1 inputs, outputs, modules, tests and acceptance updated (4); the finding-4 row and frozen decision 5 updated (2, 3).
Non-blocking clarifications adopted from Astra:
• Collision checks only on collections Hermes actually normalizes; selected route, child run and attempt distinguished (1.6).
• Failure cause kept separate from billing status; diagnostics sanitized; a charge above an enforced bound is a contract-violation halt (1.8, 1.9).
• Estimate completeness separated from bound completeness (1.5).
• Live-entry gate L1, and a latest-start deadline that covers delayed starts (1.7).
• V2, V7, V8 and V9 stated as hard live gates; H-2 possibly needed at S0 (1.6, 1.11).
• The S1 retention rule and the S0 tool assertions marked as needing Amy's decision or live proof (1.4).
• Option B's no-change claim made conditional on V1–V9 (1.10).
• MPW comparison identity stated as hypothetical; exact bindings stated as non-guarantees (1.3).
Nothing Astra accepted was reopened.
8. Verdict
READY FOR FINAL INDEPENDENT REVIEW. B1 is resolved, and no other blocker remains for the offline tranche. This is not implementation approval: tranche 1 still needs Astra's final review and the Human Chair's authorization.
```
