# Cortex V4 / FOSSIL Assurance Research — Session Handoff v3

**Date:** 2026-08-10  
**Supersedes:** `docs/handoffs/2026-08-10-v4-assurance-session-handoff-v2.md` for current execution state  
**Research repo:** `Pukujan/fossil-ai-systems`  
**Research PR:** #4 — `Capture V4 hypothesis-search and assurance research`  
**Research branch before this handoff commit:** `agent/v4-assurance-research-capture` @ `53b1d4afd61f3f6849256b4369fc719eb48a95ca`  
**Implementation repo:** `Pukujan/cortex-v4`  
**Implementation PR:** #6 — `Add durable V4 assurance, provider preflight, and process conformance`  
**Implementation branch:** `agent/durable-v4-control-plane` @ `7a420d8d143689da4978a5fb8bbf2ff9d3eb2062`  
**Latest implementation CI:** GitHub Actions run `31436581492` — success, **72 passed in 2.49s**

## Continue from here

Do not reconstruct the earlier conversation. Read the prior v2 handoff for the research synthesis and LiteLLM gate history, then use this v3 handoff as the current executable-state pointer.

The central hypothesis is unchanged:

```text
Cortex V4
= adversarial evidence/hypothesis search
+ a crash-recoverable, provenance-preserving, mechanically gated assurance substrate
```

Models are workers over claims/tests/evidence. Model agreement is metadata, not proof. Transport multiplicity is not epistemic independence.

## LiteLLM operational gate

The staging/eval no-spend gate is no longer the blocking uncertainty.

Previously established and preserved in FOSSIL:

```text
/health/liveliness -> HTTP 200
/v1/models         -> HTTP 200, 41 public aliases
/model/info        -> HTTP 200, 71 deployment rows
```

The exact runtime secret name `LITELLM_MASTER_KEY` was usable by the workflow job. The GitHub settings storage scope remains unobserved. The historical workflow mismatch is also located: the master key was originally diagnostic-only, then commit `84b9c7697b93e5826bfb11bfacbbd3cb941328d7` expanded it into the Codex/model-running step while `DEPLOYMENT.md` retained the older authority description.

The later LiteLLM Responses bridge can perform cross-model fallback and exposes requested model, actual model, and attempt chain. V4 must therefore attribute model-specific evidence to the actual model used, not merely the requested alias.

## Executable V4 contracts now on PR #6

### `cortex_v4.control.litellm_manifest`

Offline, stdlib-only normalization of already-fetched `/v1/models` and `/model/info` observations.

Mechanically enforced:

- all no-spend metadata endpoints must succeed before a fresh inventory is admitted;
- aliases, transport deployments, configured upstream-model hints, and epistemic identities remain separate objects;
- multiple routes/keys/hosts do not create independent-verifier credit;
- unresolved identity stays `UNKNOWN` instead of being guessed from route count;
- inference receipts preserve requested model, actual model, and fallback attempts;
- fallback-without-actual-model provenance receives no model-specific verification credit;
- exact-model mode rejects an explicit substitution for model-specific credit.

### `cortex_v4.control.provider_preflight`

Source-ranked provider fact reconciliation plus a no-spend LiteLLM inventory collector.

Mechanically enforced:

- official/provider source attempts are themselves preserved in the preflight report;
- model-authored receipts cannot establish provider truth;
- stale evidence cannot satisfy a fresh fact requirement;
- contradictory evidence at the highest eligible authority fails closed;
- under-authority evidence remains unverified;
- the inventory collector accepts a bearer credential only as transient input and does not preserve it in the returned manifest/receipt;
- arbitrary provider secrets in raw `/model/info` parameters are not retained;
- observable probe receipts have typed node/model/status/first-byte/total/retry/stream fields.

Still missing here: provider-specific official-source fetchers and a production inference-probe runner.

### `cortex_v4.control.assurance`

Independent deterministic reference oracle for:

```text
AssuranceWorkOrder
WorkEvent
AuthorityState
MutationPhase
AssuranceSnapshot
```

Authority ladder:

```text
PROPOSED
-> PRODUCED
-> PERSISTED
-> TESTED
-> EXTERNALLY_OBSERVED
-> INDEPENDENTLY_VERIFIED
-> PROMOTABLE
-> PROMOTED
```

Mutation phase:

```text
NONE
-> INTENT_DURABLE
-> EFFECT_OBSERVED
-> DECISION_DURABLE
```

The reference oracle enforces:

- `legacy.completed` and model verdicts are lineage metadata only;
- authority rungs cannot be skipped;
- artifact id/version must match the work order;
- event CIDs are deterministic/content-addressed;
- causal parents must already exist;
- exact duplicate replay is idempotent;
- lease takeover must advance epoch and change the fence token;
- stale epoch/fence events are rejected;
- independent verification requires a different actor plus evidence;
- mutating work cannot promote before a durable commit decision;
- mutation ordering is durable intent -> observed effect -> durable decision.

A new typed event now records unresolved mutation recovery disposition while the phase remains `INTENT_DURABLE`:

```text
reobserve
replay
compensate
block
escalate
```

Recovery disposition requires evidence and **does not** itself advance mutation phase or create committed success.

### `cortex_v4.control.assurance_store`

Concrete file-backed SQLite event store for the reference contract.

Current durability mechanism:

```text
SQLite WAL
+ synchronous=FULL
+ foreign keys
+ BEGIN IMMEDIATE validate-and-append transaction
```

Mechanically exercised:

- immutable work-order registration;
- event CID primary-key idempotence;
- each append replays the independent reference model inside the serialized write transaction before insert;
- invalid transitions roll back without partial append;
- takeover fencing survives restart;
- durable mutation decisions survive restart;
- a subprocess can commit an event and immediately terminate via `os._exit(17)` without closing the store; restart still recovers exactly one event and a retry is classified as duplicate.

This proves the event transaction survived that tested process-death boundary. It does not prove an arbitrary external system is exactly-once.

### `cortex_v4.control.direct_assurance_controller`

First direct execution surface for mutating work.

Protocol:

```text
persist mutation.intent
-> observe before replay
-> if absent, apply through idempotent + fenced mutation port
-> independently observe effect
-> persist effect receipt
-> persist durable decision
-> only then expose committed success
```

Important fencing rule: each invocation pins the epoch/fence observed on entry and never adopts a newer lease after takeover.

Mechanically exercised:

- crash injection after/before each major controller boundary;
- recovery does not reapply an already observed/idempotent effect;
- if the real effect occurs and the transport raises before the receipt is persisted, the next invocation observes first and does not blindly replay;
- ambiguous observation blocks mutation and now persists `mutation.recovery_disposition = escalate` before returning;
- a takeover during the effect window prevents the old invocation from attaching a receipt under the new fence;
- a new invocation under the new lease can observe the prior idempotent effect and complete without a second apply;
- non-commit durable decisions do not become committed success.

Strong stale-worker exclusion at the **external effect** boundary still depends on the target adapter/system atomically enforcing the supplied idempotency key and epoch/fence. The SQLite event store alone cannot prevent an external system from accepting a stale mutation.

### `cortex_v4.control.assurance_process`

Minimal JSON subprocess conformance surface. This is deliberately not a plugin framework.

Operations are limited to registering work orders, appending typed events, and reading snapshots through the same durable store/reference oracle.

Mechanically exercised:

- direct store calls and subprocess calls converge on the same authority snapshots and event CIDs;
- both surfaces reject the same invalid authority transition;
- duplicate replay remains idempotent across the process boundary;
- takeover fencing is preserved;
- process-boundary deserialization is strict: string booleans, null identities, and boolean-as-integer epochs are rejected instead of silently coerced.

The real OpenCode/plugin mutation path is not yet wired to this contract.

## Latest CI receipt

GitHub Actions workflow: `V4 contract tests`  
Run: `31436581492`  
Result: success  
Python: 3.11  
Tests: **72 passed in 2.49s**  
Workflow token permissions: `contents: read`; the contract test workflow does not request the LiteLLM/provider secret.

The suite currently spans:

- LiteLLM route/model identity and fallback provenance;
- provider-source authority/freshness/contradiction and secret sanitization;
- authority/reference invariants;
- deterministic replay and crash-prefix convergence;
- explicit recovery dispositions;
- SQLite WAL process-death recovery/idempotent retry;
- direct mutation crash/restart and fencing;
- direct-vs-process conformance and strict serialization.

Treat this as evidence for these small contracts/test scenarios only, not as proof of production correctness or provider/model epistemic independence.

## Remaining hard boundaries

Do not spend the next session adding conceptual layers before these are exercised.

1. **Real plugin/process mutation surface**
   - run the same mutation protocol across the actual OpenCode/plugin/process boundary;
   - compare its event trace and final snapshot to the reference oracle/direct controller.

2. **External fencing/idempotency adapter**
   - build a separate-process effect target that atomically stores idempotency key + epoch/fence with the mutation;
   - prove stale effect attempts are rejected at the target, not merely at FOSSIL/V4 event commit.

3. **Recovery execution, not only disposition vocabulary**
   - `reobserve`, `replay`, `compensate`, `block`, and `escalate` are now durable typed lineage;
   - real compensation and replay executors are not implemented yet;
   - inject death before/after each compensation/re-observation boundary.

4. **Joined convergence**
   - deterministic seeded scenarios combining worker loss, provider fallback, artifact revision, stale-worker wakeup, late verifier result, FOSSIL projection lag, closeout racing recovery, and process death.

5. **Closeout and promotion integration**
   - mechanically mandatory closeout receipt;
   - no child/orphan work left live;
   - no stale artifact promoted;
   - unresolved consequential uncertainty blocks or escalates rather than being hidden by model agreement.

6. **Provider live probe / V4 secret consumer**
   - the no-spend gateway path is observed in `litellm-ckff-ops`, but Cortex V4 itself is still not directly wired to the GitHub secret;
   - complete source-ranked provider fetchers and one bounded observable live probe without exposing credentials.

7. **Benchmarks only after the above**
   - single model;
   - homogeneous multi-agent;
   - heterogeneous voting;
   - hypothesis search;
   - hypothesis search + full assurance substrate.

Primary metrics remain false-success rate, consequential unresolved uncertainty, edge-case discovery, recovery correctness, and human specialist intervention — not model agreement.

## Current adjudication

The architecture has crossed from research-only into a small mechanically tested assurance kernel, but production claims are still intentionally bounded.

What is now supported by executable tests is narrow: the current contracts preserve typed authority, durable event identity, selected crash/restart behavior, route/model provenance distinctions, and matched direct/process semantics under the exercised scenarios.

What is not yet supported: real provider/V4 credential wiring, real plugin mutation conformance, target-side fencing guarantees, real compensation, joined convergence, production closeout, or superiority over alternative multi-agent methodologies.
