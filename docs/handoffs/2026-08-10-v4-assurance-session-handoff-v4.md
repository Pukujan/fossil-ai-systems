# Cortex V4 / FOSSIL Assurance Research — Session Handoff v4

**Date:** 2026-08-10  
**Supersedes:** v3 for current executable state  
**Research repo/PR:** `Pukujan/fossil-ai-systems` #4, branch `agent/v4-assurance-research-capture`  
**Implementation repo/PR:** `Pukujan/cortex-v4` #6 — `Add durable V4 assurance, provider preflight, and fenced process conformance`  
**Implementation head:** `8dc46cd7cb4c51b36eb18b4690a96c6dea561919`  
**Latest CI:** GitHub Actions run `31436833421` — success, **78 passed in 5.16s**

## Read order

Read `2026-08-10-v4-assurance-session-handoff-v3.md` for the full implementation state through durable recovery dispositions. This v4 handoff records the next stronger result: target-side fencing/idempotency is now mechanically exercised across a separate process boundary.

The architecture hypothesis remains:

```text
Cortex V4
= adversarial evidence/hypothesis search
+ a crash-recoverable, provenance-preserving, mechanically gated assurance substrate
```

Do not convert model agreement, route multiplicity, content addressing, or coordination quorum into epistemic proof.

## New result since v3: target-side effect authority

PR #6 now adds:

`cortex_v4.control.fenced_effect_target`

This is a **separate-process conformance test authority** for the external-effect boundary that V4's own SQLite event store cannot enforce on behalf of third-party systems.

The target owns its own durable lease state and effect table. Its mutation transaction is:

```text
BEGIN IMMEDIATE
-> read target current epoch/fence
-> reject stale caller if epoch/fence mismatch
-> look up idempotency key
-> if already present, return duplicate receipt
-> otherwise insert effect + applied epoch/fence
-> COMMIT
```

Target storage uses file-backed SQLite WAL + `synchronous=FULL`.

The key property is stronger than “stale V4 event commit rejected”:

> For this test authority, lease validation and external-effect insertion occur in the same target-side transaction, so a stale caller is rejected before the effect exists.

## Mechanically exercised target scenarios

The CI suite now includes these additional cases:

1. **Stale apply before effect**
   - initialize target at epoch/fence 0;
   - advance target to epoch/fence 1;
   - stale epoch/fence 0 apply is rejected with `StaleLeaseError`;
   - target effect count remains zero.

2. **Target idempotency**
   - applying the same idempotency key twice under the current lease creates one target effect.

3. **New lease observes old legitimate effect**
   - effect is committed under lease 0;
   - target lease advances to 1;
   - lease 1 observes the same effect/evidence without reapplying it;
   - stale lease 0 observation is rejected after takeover.

4. **Target process death after commit**
   - a subprocess initializes the target, commits an effect, then immediately calls `os._exit(17)` without closing the store;
   - a later process recovers exactly one target effect and observes it as applied.

5. **Direct controller through separate target process**
   - V4 direct controller uses a subprocess mutation-port adapter;
   - intent/effect observation/decision complete with one target effect.

6. **Takeover before stale apply**
   - old controller persists intent under lease 0;
   - before its effect apply, both target authority and V4 authority advance to lease 1;
   - old controller remains pinned to lease 0 and target rejects its apply before mutation;
   - target effect count is zero;
   - new lease 1 controller applies once and commits.

7. **Old legitimate effect, then takeover/recovery**
   - old controller commits the target effect under lease 0 and dies before V4 observation;
   - target and V4 authorities advance to lease 1;
   - new controller observes the existing effect using the same durable intent/idempotency key;
   - V4 completes without a second target apply; target effect count remains one.

## Latest implementation CI

Workflow: `V4 contract tests`  
Run: `31436833421`  
Result: success  
Python: 3.11  
Tests: **78 passed in 5.16s**  
Token permissions: `contents: read`; no LiteLLM/provider secret is requested by this contract workflow.

This includes all earlier LiteLLM/provider/assurance/store/direct-controller/process-boundary tests plus the target-side fencing cases above.

## Updated adjudication

The previous broad limitation:

```text
external stale-worker exclusion is only assumed
```

can now be narrowed to:

```text
V4 has an executable target-side fencing/idempotency contract
and a separate-process test authority that satisfies it;
real external systems/adapters must demonstrate equivalent semantics
or be classified as weaker/unfenced.
```

Do **not** generalize the test authority's property to arbitrary SaaS/provider APIs.

Also do not claim general exactly-once semantics. The tested property is idempotent effect identity plus atomic target-side fence validation for this adapter contract.

## Remaining hard boundaries

1. **Real OpenCode/plugin mutation path**
   - wire the actual plugin/process invocation surface to the same durable controller/event contract;
   - compare traces/snapshots against reference + direct execution.

2. **Cross-system lease handoff**
   - V4 lease takeover and target lease takeover are still two distinct mutations;
   - no distributed atomic transaction between them is claimed;
   - test orderings where only one side advances before crash and define fail-closed recovery.

3. **Real compensation executor**
   - `reobserve`, `replay`, `compensate`, `block`, `escalate` are durable typed recovery lineage;
   - compensation execution itself is not implemented;
   - add idempotent compensation receipts and crash injection before/after compensation effect/observation/decision.

4. **Joined convergence**
   - worker death + target takeover + provider fallback + artifact revision + late verifier + FOSSIL projection lag + closeout/recovery race.

5. **Closeout/promotion integration**
   - mandatory closeout receipt;
   - no orphan child work;
   - no stale artifact promotion;
   - unresolved consequential uncertainty blocks/escalates.

6. **Provider live preflight in Cortex V4**
   - LiteLLM gateway no-spend reachability is observed in `litellm-ckff-ops`;
   - Cortex V4 still is not directly wired to the GitHub `LITELLM_MASTER_KEY`;
   - provider-specific source fetchers and a bounded live probe remain unimplemented.

7. **Methodology benchmark only after assurance gates**
   - single model;
   - homogeneous multi-agent;
   - heterogeneous vote;
   - hypothesis search;
   - hypothesis search + full assurance substrate.

Primary metrics remain false-success rate, consequential unresolved uncertainty, edge-case discovery, recovery correctness, and human specialist intervention — not agreement rate.
