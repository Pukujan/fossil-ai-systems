# Cortex V4 / FOSSIL Assurance Research — Session Handoff

**Date:** 2026-08-10
**Primary research repo:** `Pukujan/fossil-ai-systems`
**Draft PR:** `#4` — `Capture V4 hypothesis-search and assurance research`
**Branch:** `agent/v4-assurance-research-capture`
**Status:** research captured; architecture proposals remain `claim.proposed`, not promoted truth

## Why this handoff exists

The previous session accumulated a large amount of architecture research and became slow. Continue from the committed FOSSIL evidence instead of reconstructing the conversation.

## Core V4 direction

Cortex V4 should not primarily be a multi-agent conversation or voting framework.

The working hypothesis is:

> V4 is an evidence-driven, adversarial hypothesis-search and execution-assurance kernel that uses many frontier models as cheap, disposable search workers while treating human specialist attention as the scarce escalation resource.

Central objects are:

- claims
- competing hypotheses
- assumptions
- evidence
- counterexamples
- falsification tests
- observations
- artifacts/revisions
- verifier receipts
- unresolved questions

Agents are workers over those objects, not authorities.

## Epistemic workflow

Preferred loop:

```text
problem
-> competing hypotheses
-> evidence acquisition
-> independent attacks
-> discriminating tests
-> counterexamples / failure mutation
-> observation
-> hypothesis update
-> bounded frontier refill
-> surviving claims
-> human escalation only where uncertainty remains consequential
```

Do not use model agreement as proof.

Repeated verbal debate is not progress unless it produces new evidence, a new falsifiable mechanism, a discriminating test, or a concrete counterexample.

A useful frontier priority hypothesis is approximately:

```text
consequence_if_wrong
× uncertainty
× evidence_gap
× historical_failure_likelihood
× novelty
÷ estimated_test_cost
```

The scheduler should eventually learn these features from FOSSIL history rather than letting a model invent them ad hoc.

## Provenance-blinded verification hypothesis

Control plane/FOSSIL knows true model/provider provenance.

Peer reviewers should normally receive blinded, normalized review projections with ephemeral aliases and without producer identity, vendor, prior verdicts, or peer verdicts before their own initial commit.

Threats already identified:

- self-preference/family-preference
- authorship/style fingerprint leakage
- anchoring/bandwagon effects
- correlated/common-mode model errors

Candidate mechanisms:

- structured semantic IR for claims/evidence/tests
- deterministic canonical rendering
- provenance-leakage classifier/gate
- commit-then-reveal independent review
- metamorphic judge tests for order/style invariance
- model-panel selection by measured complementary failures, not raw vendor count

## Endomorphosis findings already captured

### Formal/logic-harness pass

Useful patterns:

- verifier authority ceilings: model/advisor `success`, confidence, or `passed` cannot become theorem/proof-kernel authority
- differential verifier disagreement becomes `inconclusive`, not majority vote
- bounded fail-closed fuzzing with exact failure spans and deterministic shrinking/minimized counterexamples
- metamorphic/preservation tests
- bounded content-addressed work refill with attempts/depth/cooldown/open-work bounds
- fixed point means no new admissible work, **not truth**

### Reliability/coordination pass

Strong transferable patterns:

- authority/durability ladder: `completed != committed`
- durable intent before external effect
- durable decision/evidence before returning committed success
- idempotent replay/compensation on recovery
- independent reference-model conformance
- crash injection at every state-changing boundary
- joined seeded convergence tests combining failures
- content-addressed causal event DAGs
- leases/epochs/fencing tokens to reject stale workers
- coordination authority is separate from epistemic authority

Important negative findings:

- inspected `ipfs_kit_py` HA code is mixed maturity and should not be copied wholesale
- `ipfs_agents_py` is currently mostly a wiring prototype/TODO and is not evidence of a mature agent framework
- MCP++ is spec-first; its contracts/harnesses are prior art, not proof of production correctness

## Proposed V4 assurance substrate

The epistemic search engine needs a conservative control substrate underneath it.

Candidate authority ladder:

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

Exact names are not frozen.

Candidate mutating WorkOrder protocol:

```text
persist WORK_INTENT
-> verify current artifact version + lease/fence
-> execute external mutation
-> independently observe resulting artifact/state
-> persist evidence/decision
-> only then expose success
```

On crash/restart, disposition must be explicit:

- replay safely
- compensate
- re-observe
- block
- escalate

A stale worker must not be able to overwrite, verify, promote, or close newer work.

## Reference-model testing target

Build a tiny independent deterministic V4 state model.

Run matched scenarios against:

- reference model
- direct controller
- OpenCode/plugin process boundary
- future CLI/MCP surfaces

Inject death/restart before and after every state-changing boundary.

Then add deterministic seeded joined scenarios such as:

```text
worker dies
+ retry
+ provider fallback
+ artifact revision changes
+ stale worker wakes
+ verifier result arrives late
+ FOSSIL projection lags
+ closeout races recovery
+ process crashes
```

Acceptance is convergence to one valid durable state with no stale commit, duplicate effect, false verification, or orphaned child work.

## FOSSIL role

FOSSIL remains the durable evidence/intellectual-lineage system.

Keep separate:

- immutable original evidence
- source snapshots/exact spans
- proposed/disputed/supported claims
- projection/index state
- final promoted conclusions

Do not turn research conversation into truth merely because it was captured.

## LiteLLM status from the latest audit

`Pukujan/litellm-ckff-ops` already contains a broad multi-model LiteLLM configuration spanning Claude, Gemini, GPT, Grok, DeepSeek, Kimi, GLM, Qwen, MiniMax and other aliases/routes.

Existing workflow:

`litellm-ckff-ops/.github/workflows/sol-liteLLM-repair.yml`

uses:

- `LITELLM_MASTER_KEY`
- `LITELLM_STAGING_URL` / `LITELLM_PRODUCTION_URL`

The no-spend diagnostic queries:

- `/health/liveliness`
- `/v1/models`
- `/model/info`

This can prove gateway reachability and model inventory without a full inference run.

But current Cortex V4 is **not directly wired** to the GitHub secret. `scripts/run_v4_live_litellm.py` still loads a local Desktop `.env`, goes through the old SSC summon path, and uses a bounded `[grok] grok-4.5` route.

Also resolve an important documentation/code mismatch:

- `DEPLOYMENT.md` says the Codex action should receive `OPENAI_API_KEY` while the master key is diagnostic-only.
- the actual workflow currently gives `LITELLM_MASTER_KEY` to the Codex action.

Do not freeze this authority boundary until reconciled.

## Immediate next gates for the new session

1. Confirm which repository/environment contains `LITELLM_MASTER_KEY` and whether the exact secret name is correct.
2. Confirm `LITELLM_STAGING_URL` exists.
3. Run/inspect the no-spend staging diagnostic if workflow dispatch is available; record model count and route inventory, never secret contents.
4. Design a V4 `LiteLLMRouteManifest` / model-reservoir adapter from `/v1/models` + `/model/info`.
5. Classify aliases/routes separately from distinct model/family identities.
6. Add `AssuranceWorkOrder`, `WorkEvent`, authority-state and lease/fence contracts.
7. Build the independent reference model before adding large-scale orchestration.
8. Run matched methodology benchmarks:
   - single model
   - homogeneous multi-agent
   - heterogeneous vote
   - hypothesis-search
   - hypothesis-search + full assurance substrate
9. Optimize for reduced false success, unresolved uncertainty, and human intervention—not agreement rate.

## Current adjudication

Promising architecture hypothesis; not proven production methodology.

The strongest emerging formulation is:

> Cortex V4 = adversarial scientific/hypothesis search over claims and tests, running on top of a crash-recoverable, provenance-preserving, mechanically gated authority machine.

Continue by turning this into small executable contracts and falsifiable benchmarks rather than adding more conceptual complexity without tests.
