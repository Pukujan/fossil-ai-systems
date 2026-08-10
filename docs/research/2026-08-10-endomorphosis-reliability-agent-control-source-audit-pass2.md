# Endomorphosis Reliability and Agent-Control Source Audit — Pass 2

**Date:** 2026-08-10  
**Scope:** `endomorphosis/ipfs_kit_py`, `endomorphosis/Mcp-Plus-Plus`, and `endomorphosis/ipfs_agents_py`  
**State:** source-inspection evidence + V4 candidate synthesis  
**Important:** this is not independent certification of the upstream projects and no local upstream test execution is claimed in this pass.

## Research question

Which concrete reliability, recovery, provenance, parallel-work coordination, and fail-closed patterns in Endomorphosis's OSS are transferable to Cortex V4's evidence-driven hypothesis-search/assurance methodology, and which apparent patterns should be rejected or treated as scaffolding?

## Upstream revisions inspected

### ipfs_kit_py

Primary revision used for direct file inspection:

- `endomorphosis/ipfs_kit_py@69091bf8f11a3ef1fb0e04e11a6d8a4c87f3fa78`

Additional source-level commits inspected:

- `48087c9cdaa700d6b45a37919f98a331565f1772` — KITA-018, canonical WAL records/durability states/legacy mappings.
- `3ceb41be735655e80327d9bedf54a9c3e4e17cab` — KITA-009, joined VFS/backend/WAL/crash/interface conformance.
- `3a6ba311ee6fa9cacb326e722ea4b854ae9c9a7d` — KITA-029, joined bucket/WAL/index/cache/backend/replica convergence.
- recent commit history also advertises KITA-021 WAL cutover and KITA-044 bounded backpressure while preserving durability semantics; those were not separately audited to the same depth in this pass.

Concrete files inspected:

- `ipfs_kit_py/core/wal/contracts.py`
- `ipfs_kit_py/core/wal/coordinator.py`
- legacy `ipfs_kit_py/wal.py`
- `ipfs_kit_py/mcp/extensions/ha.py`
- `ipfs_kit_py/mcp/enterprise/high_availability.py`

### MCP++

Primary revision:

- `endomorphosis/Mcp-Plus-Plus@dc3164653a48d059ae9812078359daeafb451c07`

Additional commit:

- `15c1816d6c63a2b11edd505704f6a04a9abc6167` — cross-language canonical conformance fixes.

Concrete files inspected:

- `docs/spec/mcp++-profiles-draft.md`
- `docs/spec/event-dag-ordering.md`
- `docs/spec/risk-scheduling.md`
- `tests-py/validators/models.py`
- `tests-py/harness/profile_g_three_peer.py`
- `docs/testing/profile-g-release/evidence.json`

### ipfs_agents_py

Revision surfaced by source search:

- `endomorphosis/ipfs_agents_py@578b47ae0b4ea15b48f7955edcc2f2bbb627bbec`

Files inspected:

- `README.md`
- `ipfs_agents_py/ipfs_agent.py`

---

## Finding 1 — durability is an authority ladder, not a boolean

The newer `ipfs_kit_py` WAL contract is much stronger than the older `wal.py`.

`core/wal/contracts.py` explicitly separates:

- `buffered`
- `queued`
- `appending`
- `appended`
- `file_synced`
- `parent_synced`
- `prepared`
- `committed`
- `archived`
- `replayed`
- explicit failure states

It also distinguishes `DURABLE_STATES`, `COMMITTED_STATES`, pre-commit media progress, terminal success, and terminal failure.

The crucial migration rule in KITA-018 is that legacy values such as `completed`, `success`, `ok`, and `done` do **not** automatically become `committed`. Legacy `completed` is mapped to the pre-commit `appended` state unless additional durability evidence proves a stronger boundary. Unknown legacy states are preserved explicitly and cannot be silently promoted.

### V4 implication

This is directly analogous to the mistake V4 must avoid with agent work:

```text
model says "done"
tool returned success
file exists
tests ran
reviewer approved
```

are different authority levels.

Candidate V4 ladder:

```text
PROPOSED
PRODUCED
PERSISTED
TESTED
EXTERNALLY_OBSERVED
INDEPENDENTLY_VERIFIED
PROMOTABLE
PROMOTED
```

The exact names are open, but a generic `success=true` must never be allowed to collapse these states.

**Candidate claim:** V4 should model assurance/promotion as a typed authority ladder, with adapters forbidden from translating weaker completion signals into stronger verified/promotion states.

---

## Finding 2 — durable intent before effect; durable decision before success

`core/wal/coordinator.py` implements a small transaction protocol around external effects.

The sequence is materially important:

1. durable `BEGIN`
2. durable intent with stable `effect_id`
3. perform external effect
4. durably write a commit-pending decision
5. durably append `COMMIT`
6. durably record commit decision
7. only then return committed success

Recovery differentiates committed and non-committed intent:

- committed intent can be replayed;
- non-committed durable intent can be compensated;
- a separate replay ledger prevents the same recovery effect from being executed again on a second recovery pass.

The KITA-009 conformance suite injects crashes at named boundaries including before/after begin, intent, effect, and commit. Its stated acceptance includes pre-commit compensation, committed replay once, and second recovery as a no-op.

### V4 implication

This is almost a direct solution to Cortex's current cross-process mechanical-session weakness.

For mutating WorkOrders, V4 can persist:

```text
WORK_INTENT
artifact/version precondition
worker/lease identity
effect id / idempotency key
expected mutation surface
```

**before** allowing the mutation.

After the external mutation, V4 should not emit `VERIFIED` merely because the tool returned success. It should durably record the resulting artifact identity/evidence and only then advance the WorkOrder state.

If V4 crashes between effect and durable decision, recovery must have an explicit disposition:

```text
replay safely
compensate
re-observe external state
block
escalate
```

**Candidate claim:** mutation execution in V4 should use a durable intent/effect/decision protocol with idempotent recovery rather than in-memory "preflight happened" flags.

---

## Finding 3 — joined conformance is more relevant than unit-test count

KITA-009 describes a joined VFS conformance suite in which:

- one canonical VFS service owns runtime semantics;
- an independent reference model acts as an oracle;
- Python, CLI, MCP, and MCP++ adapters must project the same semantics rather than reimplement storage behavior;
- WAL crash injection covers every named transaction boundary;
- generated concurrent schedules are compared with the reference model or explicitly classified unsupported;
- unavailable backend capabilities reject explicitly rather than silently fall back;
- transport-only fields are stripped before semantic parity comparison.

This is much closer to the V4 validation problem than ordinary unit tests.

### V4 implication

A V4 control-plane change should be tested against:

1. a small deterministic reference state machine;
2. the real implementation;
3. multiple invocation surfaces (direct Python/controller, OpenCode plugin, CLI/MCP if present);
4. injected process death at every state-changing boundary;
5. replay/restart;
6. semantic equivalence after stripping transport/runtime noise.

This would have caught the existing "preflight in process A, gate in process B" failure immediately.

**Candidate claim:** V4 should have a reference-model conformance harness that compares control-plane behavior across invocation surfaces and process/crash boundaries.

---

## Finding 4 — convergence tests combine failures instead of testing components in isolation

KITA-029 joins bucket placement, WAL recovery, replica reconciliation, cache invalidation, GraphRAG projection, backend capabilities, and failure compensation.

Its declared invariants include:

- read-back digest/version verification reaches exactly the desired replica count;
- committed WAL recovery produces one external effect and second recovery produces no duplicate effect;
- stale cache bindings are invalidated;
- projections expose the committed version;
- partial placement either compensates or leaves a durable recoverable receipt;
- unsupported capabilities return typed unsupported results and cannot silently fall back.

The test uses deterministic seeded schedule permutations (`64` seeds in the declared receipt), with dependencies deferred until ready.

### V4 implication

V4 should add **joined failure scenarios**, not only component tests:

```text
worker dies
+ retry happens
+ fallback provider changes
+ stale artifact remains
+ verifier is delayed
+ cache/FOSSIL projection is behind
+ closeout races with recovery
```

The acceptance condition should be final convergence of durable state and externally observable artifacts, not "every individual call returned success."

**Candidate claim:** V4 assurance should include deterministic seeded interleaving/convergence tests spanning scheduler, provider routing, artifact versioning, FOSSIL projection, retries, verification, and closeout.

---

## Finding 5 — not all "HA" code in ipfs_kit_py deserves equal authority

The legacy/basic HA extension has useful vocabulary but should not be treated as production evidence.

Concrete limitations observed in `mcp/extensions/ha.py` include:

- DNS discovery branch explicitly not implemented;
- multicast discovery branch explicitly not implemented;
- node statistics are populated with random simulated values;
- event persistence is probabilistic (`~10%` chance on each event), not an append-before-ack durability contract;
- leader selection is local deterministic sorting over each process's `known_nodes`, not a demonstrated distributed consensus protocol;
- automatic failover mutates local leadership state;
- in the manual-target failover branch, the event's `previous_leader` is populated after `leader_id` is overwritten, so the recorded previous leader can equal the new leader.

The newer `mcp/enterprise/high_availability.py` introduces Redis-backed heartbeat/state and direct health checking, but still contains explicit dummy metric values and does not by itself establish a consensus-grade leader-election protocol.

### V4 implication

Do **not** import "leader election / HA / quorum" vocabulary as evidence that the implementation is safe.

The transferable lesson is the opposite:

> reliability features themselves require authority receipts and adversarial validation.

For V4, provider fallback, worker takeover, retry, queue ownership, and verifier takeover should be tested mechanically rather than declared by architecture.

**Negative conclusion:** Endomorphosis's HA modules are useful hypothesis generators and failure examples, but the inspected code is not a production pattern V4 should copy wholesale.

---

## Finding 6 — MCP++ Profile F is a strong provenance/control-plane analogue

MCP++ is explicitly documentation/spec-first, so it should not be mistaken for a drop-in orchestrator. However, Profile F's contract is highly aligned with V4/FOSSIL.

Profile F defines content-addressed execution events whose parent references express causal dependencies. Independent events need not receive a fake total order.

The contract also separates:

- hot event retention;
- durable archive;
- compaction certificate;
- archive/inclusion verification;
- bounded traversal;
- causal replay/rollback/attribution.

Critically, the specification says compaction must not silently discard the underlying event records, and a hash/Merkle root/simulated proof must not be mislabeled as a real zero-knowledge proof. If the actual proving machinery/key is absent, ZK availability must fail closed.

### V4 implication

V4's execution history should probably be a content-addressed **partial-order event DAG** rather than one giant chronological agent transcript.

Example:

```text
WorkOrder
  -> hypothesis H1
  -> test T1
       -> observation O1
  -> hypothesis H2
       -> attack A7
       -> test T9
  -> artifact revision R3
       -> verifier V8
```

Only causal dependencies need ordering. Parallel independent attacks remain concurrent.

This aligns naturally with FOSSIL's append-only evidence model and would improve replay, stale-artifact detection, causal attribution, and bounded history handling.

**Candidate claim:** V4 should represent execution provenance as a content-addressed causal DAG with bounded hot projections and durable underlying evidence, rather than forcing global total ordering.

---

## Finding 7 — MCP++ Profile G contains a useful stale-worker solution

The generic `risk-scheduling.md` is mostly non-normative, but the Profile G three-peer harness is concrete enough to inspect.

The harness:

- uses content-addressed events;
- rejects events whose claimed CID does not match their canonical body;
- rejects missing causal parents;
- persists peer events and rebuilds materialized state by replay;
- models explicit network partition/heal;
- requires a reachable majority for an exclusive-lease resolution;
- assigns leases and monotonic fencing tokens;
- requires a prior lease-expiry event before a takeover epoch;
- rejects completion from a stale/expired fence;
- makes repeated identical completion/rejection paths idempotent;
- reconciles peers by causal event exchange and checks converged frontiers;
- can recreate a peer from its durable event store.

Its release evidence also makes operational failure states explicit: `degraded`, `denied`, `conflicted`, `expired`, `stale_fence`, `partitioned`, `blocked`, and `unavailable`.

### V4 implication

This is directly applicable to the problem:

```text
worker A owns artifact R2
worker A stalls
V4 reassigns work to worker B
worker B produces R3
worker A wakes up and attempts to commit R2
```

A process-local "cancelled" flag is insufficient.

Use a monotonic **artifact/work fencing token**:

```text
work_id: W17
epoch: 4
fence: 19
artifact_base: sha256(...)
lease_expires: ...
```

Any mutation, verification promotion, or closeout from fence `<19` is rejected mechanically.

This is particularly relevant to V4's current process-boundary state loss.

**Candidate claim:** parallel V4 work that can mutate/promote shared state should use leases/epochs plus monotonic fencing tokens so stale workers cannot commit after takeover.

---

## Finding 8 — Profile G is coordination evidence, not epistemic truth

The same mechanism has an important boundary.

Profile G's majority is used to establish an **exclusive execution lease** among three coordination peers. It does not establish that a hypothesis is true.

This distinction maps cleanly to V4:

```text
coordination quorum:
    who may execute/commit this work right now?

epistemic verification:
    which claim survived evidence and falsification?
```

These must remain separate.

### Critique / limits

- The inspected harness is exactly three peers.
- It intentionally uses no real sockets and no sleeps; partitions are simulated through explicit link sets.
- Its clock is deterministic.
- Its release evidence is project-produced evidence, not independent certification.
- The reported `2.752294` throughput gain was read from committed release evidence, not reproduced in this audit.
- A three-peer majority harness cannot be generalized into arbitrary distributed consensus correctness.
- Fencing prevents stale writers; it does not prove the accepted writer's output is correct.

**Candidate claim:** V4 should separate concurrency/commit authority from epistemic authority; coordination quorums and fencing may control who can mutate state but must never promote claims merely because a quorum agreed.

---

## Finding 9 — risk can be derived from immutable failure history, but the scheduler remains experimental

MCP++ Profile G proposes computing risk from immutable event history including policy violations, anomaly rates, divergence from predicted outcomes, rollbacks/disputes, missed obligations, and disputed receipts.

It proposes a frontier keyed by expected value, risk-adjusted cost, dependency readiness, and local frontier distance.

This is conceptually close to V4's proposed:

```text
consequence_if_wrong
× uncertainty
× evidence_gap
× historical_failure_likelihood
× novelty
÷ test_cost
```

### V4 implication

The important reusable pattern is **not** Fibonacci heaps, LSH, or neighborhood consensus.

It is:

> derive scheduling features from durable historical evidence instead of letting a model invent its own notion of priority every turn.

FOSSIL could eventually provide task-conditioned features such as:

- model/role historical miss rate;
- verifier disagreement rate;
- known failure mechanism frequency;
- branch novelty;
- expected discrimination power of a test;
- stale-evidence risk;
- consequence class.

The scheduling algorithm can remain replaceable and benchmarked.

**Candidate claim:** V4 frontier scoring should consume durable measured failure/evidence features from FOSSIL while keeping the actual scheduling algorithm replaceable.

---

## Finding 10 — ipfs_agents_py is a negative result

`ipfs_agents_py` should not currently be cited as a mature multi-agent architecture.

Its README states `TODO:: implement after libp2p protocol`.

The main `ipfs_agent.py` mostly wires model manager, OrbitDB, libp2p, and config objects. Its `__call__` returns immediately after `connect_orbitdb`, leaving subsequent sample mutation code unreachable.

### V4 implication

Do not spend additional architecture budget on this repository unless later history shows a materially newer implementation.

**Negative conclusion:** `ipfs_agents_py` currently contributes almost no evidence for Cortex's orchestration methodology.

---

# Cross-project synthesis

The strongest Endomorphosis patterns now appear to form one coherent reliability philosophy:

```text
typed contracts
-> bounded authority
-> content-addressed identity
-> intent before mutation
-> durable receipts
-> independent reference/conformance checks
-> crash/failure injection
-> idempotent replay/compensation
-> causal event history
-> explicit blocked/unavailable/conflicted states
-> stale-writer fencing
-> convergence tests
-> no silent fallback or authority upgrade
```

This is much more relevant to Cortex than "multi-agent agents talking to each other."

## Proposed V4 additions after this pass

### A. Assurance authority ladder

Separate:

- model proposal
- durable artifact
- executable test result
- external observation
- independent verification
- promotion authority

No adapter may silently upgrade authority.

### B. Durable mutation transaction

For state-changing WorkOrders:

```text
persist intent
-> authorize current lease/fence
-> execute effect
-> observe artifact/result
-> persist decision/evidence
-> expose success
```

Recovery must replay, compensate, re-observe, block, or escalate.

### C. Work/artifact fencing

Every mutating/reviewing worker should receive a current epoch/fence. A stale fence cannot:

- overwrite artifact;
- approve a newer revision;
- close a task;
- update claim state;
- publish closeout.

### D. Causal WorkEvent DAG

Persist event identity + parent CIDs for:

- task decomposition;
- hypotheses;
- attacks;
- tests;
- observations;
- mutations;
- revisions;
- verifier receipts;
- recovery;
- closeout.

Use projections for convenient timelines/search; preserve causal source events durably.

### E. Reference-model + crash conformance suite

Maintain a tiny deterministic V4 model separate from production implementation.

Run the same scenario against:

- reference;
- direct controller API;
- plugin/process boundary;
- future CLI/MCP surfaces.

Inject crash/restart at every state-changing boundary.

### F. Joined convergence scenarios

Generate seeded interleavings combining:

- worker loss;
- stale completion;
- provider fallback;
- duplicated invocation;
- artifact revision;
- delayed verifier;
- FOSSIL projection lag;
- closeout race;
- recovery.

Require convergence to one valid durable state.

---

# What this pass does NOT establish

It does not establish that:

- `ipfs_kit_py` as a whole is production-correct;
- its HA stack is consensus-safe;
- MCP++ is an adopted standard;
- Profile G's benchmark numbers independently reproduce;
- three-peer coordination proves arbitrary distributed correctness;
- any Endomorphosis component validates V4's epistemic methodology;
- content addressing alone proves semantic correctness.

The observed code provides **mechanism prior art and testable candidate patterns**.

# Current adjudication

This pass strengthens a narrower hypothesis:

> Cortex V4 should combine adversarial hypothesis/evidence search with distributed-systems-grade authority boundaries around the harness itself: durable intent, explicit commit authority, causal provenance, stale-worker fencing, independent reference-model conformance, crash injection, idempotent recovery, and joined convergence tests.

It does **not** strengthen "more agents = more truth."

The strongest next build/test question is whether these control-plane mechanisms measurably reduce false success, stale commits, unrecoverable ambiguity, and human intervention in Cortex's own tasks under matched workloads.
