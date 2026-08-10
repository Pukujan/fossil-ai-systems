# Cortex V4 / FOSSIL Assurance Research — Session Handoff v2

**Date:** 2026-08-10
**Primary research repo:** `Pukujan/fossil-ai-systems`
**Research PR:** `Pukujan/fossil-ai-systems#4` — `Capture V4 hypothesis-search and assurance research`
**Research branch:** `agent/v4-assurance-research-capture`
**Implementation repo:** `Pukujan/cortex-v4`
**Implementation PR:** `Pukujan/cortex-v4#6` — `Add durable V4 assurance and LiteLLM preflight contracts`
**Implementation branch:** `agent/durable-v4-control-plane`
**Status:** LiteLLM operational gate observed; first deterministic V4 route-manifest and assurance reference contracts implemented; architecture claims remain proposals unless mechanically or externally verified.

## Start here

Do not reconstruct the earlier conversation. Treat FOSSIL as the preserved research lineage and this file as the current continuation pointer.

Read first:

- `docs/handoffs/2026-08-10-v4-assurance-session-handoff.md`
- `docs/research/2026-08-10-v4-assurance-working-synthesis.md`
- `docs/research/2026-08-10-litellm-secret-multimodel-wiring-audit.md`
- `docs/research/2026-08-10-litellm-operational-gate-route-manifest-pass2.md`
- the current `Pukujan/cortex-v4#6` diff

The core architecture remains:

```text
Cortex V4
= adversarial hypothesis/evidence search
+ a crash-recoverable, provenance-preserving, mechanically gated assurance substrate
```

Model agreement is metadata, not proof. Transport multiplicity is not epistemic independence.

## LiteLLM gate — now mechanically observed

The first LiteLLM audit correctly recorded that source wiring existed but live authentication had not yet been observed in that session. That is now superseded for the operational gate by GitHub Actions evidence.

The protected workflow history separates two changes:

1. commit `a3d8058fd0ced9624907501e12b02f6544e5470c` introduced the workflow with `LITELLM_MASTER_KEY` used by the diagnostic while Codex still used `OPENAI_API_KEY`;
2. commit `84b9c7697b93e5826bfb11bfacbbd3cb941328d7` later changed the Codex step to receive `LITELLM_MASTER_KEY`, pointed it at LiteLLM `/v1/responses`, and selected `gpt-5.6-sol`.

Therefore the `DEPLOYMENT.md` / executable-workflow mismatch was introduced by the later routing commit, not present in the original protected workflow.

A staging-labelled no-spend workflow run successfully exercised:

```text
/health/liveliness -> HTTP 200
/v1/models         -> HTTP 200, 41 public aliases
/model/info        -> HTTP 200, 71 deployment rows
```

The run proves that a credential named `LITELLM_MASTER_KEY` and the selected LiteLLM URL were usable by that job. The connector still does not expose GitHub settings-level secret metadata, so environment/repository/organization storage scope remains unknown and must not be guessed.

The observed staging/eval host is `litellm-eval-production.up.railway.app`. The hostname substring `production` is not an authority for environment classification; use workflow/deployment policy instead.

The full pass is preserved in:

`docs/research/2026-08-10-litellm-operational-gate-route-manifest-pass2.md`

with content-addressed artifact:

`art_aa9a22527d42143ea16e9c86d0b87a1f7f62`

and derived snapshot:

`snap_f044ce9c6f3d93967a4d5039`

Five conclusions were seeded as `claim.proposed`, not promoted.

## Critical fallback/provenance constraint

`litellm-ckff-ops` subsequently merged a Responses bridge that can perform bounded sequential cross-model fallback and emits:

```text
bridge_requested_model
bridge_actual_model
bridge_attempts[]
```

So an HTTP 200 for requested model A may have actually been produced by model B.

V4 must therefore:

- preserve requested model, actual model, and route attempts;
- attribute model-specific evidence to the actual model;
- reject model-specific verification credit when fallback can occur but actual-model provenance is missing;
- reject an explicit substitution when exact-model mode was required;
- never count multiple routes/keys/hosts/deployments as independent reviewers merely because they are distinct transport entries.

## Cortex V4 executable slice — PR #6

Use the existing draft branch `agent/durable-v4-control-plane`; do not create a parallel architecture branch unless there is a concrete reason.

PR #6 now contains two deliberately small, stdlib-only executable contracts.

### 1. `cortex_v4.control.litellm_manifest`

This is an offline normalizer/admission primitive, not yet the network preflight client.

It:

- consumes already-fetched `/v1/models` and `/model/info` payloads plus endpoint statuses;
- refuses a fresh manifest unless all metadata endpoints are HTTP 200;
- sanitizes observations instead of storing arbitrary gateway bodies or credentials;
- keeps `AliasRecord`, `DeploymentRecord`, `EpistemicIdentityRecord`, and `GatewayReceipt` separate;
- groups only explicit configured upstream-model hints where available;
- leaves unresolved model identity as `UNKNOWN` rather than guessing from names/routes;
- gives transport multiplicity zero independent-verifier credit by default;
- normalizes inference receipts around requested model, actual model, attempts, and fallback policy.

Important acceptance behavior already covered by tests:

```text
4 Kimi deployments != 4 epistemic identities
3 Sol routes != 3 independent reviewers
requested A / actual B -> evidence attributed to B
fallback possible + no actual model -> no model-specific credit
unknown upstream identity -> UNKNOWN
failed inventory endpoint -> no fresh admitted manifest
exact-mode explicit substitution -> no model-specific credit
```

This is only a partial implementation of `cortex-v4` issue #3. The full source-ranked provider preflight still needs official/provider source reconciliation, freshness/confidence/contradictions, key-pool/streaming policy, and observable live probe receipts.

### 2. `cortex_v4.control.assurance`

This is a small deterministic **reference oracle**, not the production durable controller/store.

It defines:

```text
AssuranceWorkOrder
WorkEvent
AuthorityState
MutationPhase
AssuranceSnapshot
AssuranceReferenceModel
```

Current authority ladder:

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

Current mutation phase:

```text
NONE
-> INTENT_DURABLE
-> EFFECT_OBSERVED
-> DECISION_DURABLE
```

The reference model enforces:

- `legacy.completed` and model verdicts are lineage metadata only;
- authority transitions cannot skip rungs;
- work-order artifact id/version must match;
- event CIDs are deterministic/content-addressed;
- listed causal parents must already exist;
- exact duplicate event replay is idempotent;
- lease takeover must increase epoch and change the fencing token;
- stale epoch/fence events are rejected;
- independent verification requires a separate actor plus evidence;
- mutating work cannot promote before a durable `commit` decision;
- mutation events must follow intent -> observed effect -> durable decision.

Do not overinterpret “separate verifier actor” as proof of epistemic independence. Actual model/family/error-process independence remains a different evidenced property.

## Validation status

The new LiteLLM + assurance slices have **22 isolated deterministic pytest tests passing**.

Coverage includes:

- pooled route vs model identity;
- requested/actual fallback provenance;
- exact-model substitution refusal;
- unknown identity fail-closed behavior;
- failed inventory refusal;
- `completed != committed`;
- authority-skip refusal;
- intent/effect/decision ordering;
- artifact-version mismatch;
- stale-worker fencing after takeover;
- missing causal-parent refusal;
- duplicate replay idempotence;
- separate verifier evidence;
- no mutating promotion before durable commit;
- reference-model restart/resume convergence at every event boundary.

The crash-boundary test is deliberately limited: it discards the live oracle at every event cut, reconstructs from the durable prefix, and resumes. It proves deterministic event-replay convergence only. It does **not** yet prove recovery, re-observation, compensation, or exactly-once external effects around a real mutation.

No GitHub Actions workflow run was attached to the current PR #6 head at the time of this handoff. Do not claim repository-wide CI.

## Immediate next executable gates

Proceed in this order unless new evidence falsifies it:

1. Complete the source-ranked provider preflight around the manifest:
   - collect current gateway/config/provider facts;
   - preserve source + retrieval time + freshness + contradictions;
   - generate the normalized `LiteLLMRouteManifest`;
   - perform one bounded observable probe where required;
   - never expose credentials.
2. Add a durable event store/transaction boundary for the assurance protocol rather than process-local state.
3. Implement one direct controller against the same `AssuranceWorkOrder` / `WorkEvent` contract.
4. Build an independent matched adapter across a plugin/process boundary.
5. Run conformance scenarios against:
   - reference oracle;
   - direct controller;
   - plugin/process surface.
6. Inject crash/restart before and after every real state-changing boundary, especially:
   - before durable intent;
   - after intent / before external effect;
   - after effect / before observation;
   - after observation / before durable decision;
   - after decision / before client success;
   - during lease takeover;
   - when a stale worker resumes.
7. Require explicit recovery disposition:
   - replay safely;
   - re-observe;
   - compensate;
   - block;
   - escalate.
8. Add deterministic joined-convergence scenarios combining worker loss, fallback, stale artifacts, late verification, projection lag, closeout/recovery races, and process death.
9. Only then benchmark:
   - single model;
   - homogeneous multi-agent;
   - heterogeneous vote;
   - hypothesis search;
   - hypothesis search + full assurance substrate.

Primary outcome metrics remain false-success rate, consequential unresolved uncertainty, edge-case discovery, recovery correctness, and human specialist intervention — not agreement rate.

## Current adjudication

The LiteLLM gateway reachability/inventory uncertainty is no longer the main blocker. The current blocking questions are now about trustworthy model identity/provenance at admission time and whether V4 can preserve its authority/mutation invariants across real process and external-effect failures.

The first reference contracts exist and are falsifiable. Keep them small. Do not add broad conceptual machinery unless it creates an executable hypothesis, a discriminating test, or a recovery/conformance receipt.
