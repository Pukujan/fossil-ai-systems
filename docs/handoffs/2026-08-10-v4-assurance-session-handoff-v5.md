# Cortex V4 / FOSSIL Assurance Research — Session Handoff v5

**Date:** 2026-08-10  
**Supersedes:** v4 for current executable state  
**Research repo/PR:** `Pukujan/fossil-ai-systems` #4, branch `agent/v4-assurance-research-capture`  
**Implementation repo/PR:** `Pukujan/cortex-v4` #6 — `Add durable V4 assurance, provider preflight, and fenced process conformance`  
**Implementation head:** `a2a3431ec235f143c8aadeb98bc47e1126c45327`  
**Latest CI:** GitHub Actions run `31437065266` — success, **80 passed in 7.66s**

## What changed since v4

V4 and the separate fenced effect target now have an explicit typed contract for the two **partial cross-system lease handoff** states.

New implementation pieces:

- `cortex_v4.control.direct_assurance_controller`
  - adds `MutationPortError` and `MutationPortLeaseMismatch`;
  - adds `MutationRunStatus.BLOCKED`;
  - when the target rejects the invocation epoch/fence, the controller persists `mutation.recovery_disposition = block` with evidence and leaves the mutation phase unresolved rather than silently retrying or treating the mismatch as success.

- `cortex_v4.control.fenced_effect_port`
  - translates target-process `StaleLeaseError` into the controller-level `MutationPortLeaseMismatch` contract.

- `tests/test_cross_system_lease_handoff.py`
  - exercises both one-sided handoff orderings.

## Cross-system handoff scenarios now mechanically exercised

### A. Target advances first, V4 is still old

Initial state:

```text
V4 epoch/fence     = 0 / fence-0
target epoch/fence = 0 / fence-0
```

During a direct mutation, after recovery observation says the effect is absent but before apply, only the target advances:

```text
V4                 = 0 / fence-0
target             = 1 / fence-1
```

The old controller remains pinned to V4 lease 0. Its target apply is rejected before effect insertion.

Mechanically observed outcome:

```text
target effect count = 0
controller status   = BLOCKED
V4 mutation phase   = INTENT_DURABLE
V4 lineage          = mutation.intent
                      -> mutation.recovery_disposition(block)
```

After V4 later advances to epoch/fence 1, a fresh invocation recovers and commits one target effect.

### B. V4 advances first, target is still old

Initial target remains at epoch/fence 0 while V4 takeover advances to epoch/fence 1.

A new controller invocation correctly uses V4 lease 1. Its target observation is rejected because the target still owns lease 0.

Mechanically observed outcome:

```text
target effect count = 0
controller status   = BLOCKED
V4 mutation phase   = INTENT_DURABLE
V4 lineage          = lease.takeover
                      -> mutation.intent
                      -> mutation.recovery_disposition(block)
```

After the target later advances to epoch/fence 1, recovery proceeds and commits one target effect.

## What this proves — and what it does not

The tested two-authority contract is now fail-closed for both one-sided intermediate states:

```text
V4 lease == target lease
    -> mutation may proceed subject to normal assurance gates

V4 lease != target lease
    -> mutation is blocked
    -> zero new target effects
    -> durable recovery lineage records the mismatch
```

This is materially stronger than assuming lease handoff will be simultaneous.

It does **not** prove a distributed atomic lease transaction. V4 takeover and target takeover remain two distinct durable mutations with a temporary availability gap possible between them.

The current design chooses safety over availability during that mismatch: no authority borrowing, no implicit fence refresh, and no mutation until both sides agree on the lease context.

## Latest CI receipt

Workflow: `V4 contract tests`  
Run: `31437065266`  
Result: success  
Python: 3.11  
Tests: **80 passed in 7.66s**  
Workflow token permissions: `contents: read`; the contract workflow does not request LiteLLM/provider secrets.

The suite now spans:

- LiteLLM route/model identity and fallback provenance;
- provider-source authority/freshness/contradiction and secret sanitization;
- authority/reference invariants;
- durable recovery dispositions;
- V4 SQLite process-death recovery/idempotent retry;
- direct mutation crash/restart/fencing;
- target-side atomic fencing + idempotency + target-process death recovery;
- partial cross-system lease handoff in both orderings;
- direct-vs-process conformance and strict serialization.

## Current remaining gates

1. **Actual OpenCode/plugin mutation boundary**
   - replace the conformance-only JSON process surface with the real plugin/process invocation path;
   - compare exact event traces and final snapshots against the reference/direct surfaces.

2. **Compensation execution**
   - recovery vocabulary already includes `compensate`;
   - build an idempotent compensating-effect target/receipt protocol;
   - crash-inject before/after compensation apply, observation, and durable decision.

3. **Lease handoff protocol receipt**
   - the two mismatch states are fail-closed, but the act of synchronizing V4 and target lease transitions is not itself represented as a higher-level handoff transaction;
   - if a coordinator is added, keep it separate from epistemic authority and test every crash ordering.

4. **Joined convergence**
   - combine worker death, lease mismatch/takeover, provider fallback, artifact revision, late verifier result, FOSSIL projection lag, and closeout/recovery races under deterministic seeds.

5. **Closeout/promotion**
   - mechanically mandatory closeout receipts;
   - no orphan child work;
   - no stale artifact promotion;
   - consequential unresolved uncertainty blocks/escalates.

6. **Cortex V4 live provider gate**
   - provider-specific source fetchers + bounded live probe;
   - direct V4 secret consumption remains unwired and must not be inferred from `litellm-ckff-ops` workflow success.

7. **Matched methodology benchmark** only after the assurance gates above.

Primary metrics remain false-success rate, consequential unresolved uncertainty, edge-case discovery, recovery correctness, and human specialist intervention — not agreement rate.
