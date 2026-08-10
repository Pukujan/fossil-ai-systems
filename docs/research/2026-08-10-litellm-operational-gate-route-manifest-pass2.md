# LiteLLM Operational Gate and V4 Route Manifest — Pass 2 — 2026-08-10

**Scope:** `Pukujan/litellm-ckff-ops`, the previously captured LiteLLM wiring audit in FOSSIL, and the V4 model-reservoir integration boundary  
**State:** source + GitHub Actions execution evidence; secret values remain intentionally undisclosed  
**Authority:** operational gate evidence plus proposed V4 contract; architecture claims remain `claim.proposed`

## Research question

What changed since the first LiteLLM wiring audit, what is now mechanically proven about the staging gateway, and what minimum route/model identity contract should Cortex V4 implement before using the gateway as an epistemic model reservoir?

## Sources inspected

- `Pukujan/litellm-ckff-ops` commit `a3d8058fd0ced9624907501e12b02f6544e5470c` — `ops: add protected Sol repair workflow`
- `Pukujan/litellm-ckff-ops` commit `84b9c7697b93e5826bfb11bfacbbd3cb941328d7` — `ci: route Sol repair through LiteLLM`
- GitHub Actions run `31423085160`, especially the successful `Collect redacted live diagnostics`, `Run supervised Sol through LiteLLM`, and `Validate candidate` steps
- `sol/litellm-repair-31423085160:artifacts/live-diagnostics.json`
- `sol/litellm-repair-31424309476:artifacts/live-diagnostics.json`
- merged PR `Pukujan/litellm-ckff-ops#4` and current `API-CONTRACT.md`
- prior FOSSIL artifact `art_dbc207057fcfe6582eab34b4c949b413a2d2`

No credential value was requested, reconstructed, or recorded.

## Gate result

### G1 — exact workflow secret name

**PASS for runtime availability; settings storage scope remains unknown.**

The protected workflow was originally introduced at commit `a3d8058...`. From its first version:

```text
job environment = selected staging|production environment
diagnostic LITELLM_URL = selected GitHub variable
diagnostic LITELLM_MASTER_KEY = secrets.LITELLM_MASTER_KEY
Codex key = secrets.OPENAI_API_KEY
```

The first source-level appearance therefore already establishes the intended exact secret name: `LITELLM_MASTER_KEY`.

A later staging-labelled workflow run reached the diagnostic step with `LITELLM_MASTER_KEY` redacted by Actions and the step succeeded. Because `scripts/live_diagnose.py` exits if either `LITELLM_URL` or `LITELLM_MASTER_KEY` is empty, the run is stronger evidence than source wiring alone: a secret with the expected name was available to that job and was accepted well enough for the authenticated inventory requests to succeed.

The available connector does not expose the GitHub settings page or secret metadata needed to distinguish whether that same-name secret is stored at environment, repository, or organization scope. Do not invent that scope from the workflow expression.

### G2 — staging URL

**PASS for the executed staging-labelled target.**

The successful run records:

```text
LITELLM_URL = https://litellm-eval-production.up.railway.app
```

and the redacted diagnostic stores the corresponding host:

```text
litellm-eval-production.up.railway.app
```

The hostname itself contains the word `production`, but the run was used as the staging/eval gate. Environment role must come from the workflow target and deployment policy, not from substring classification of a hostname.

### G3 — no-spend authenticated diagnostic

**PASS.**

The preserved diagnostic records:

```text
/health/liveliness -> 200
/v1/models         -> 200, model_count = 41
/model/info        -> 200, deployment_count = 71
```

It also records the four observed `kimi-k2.7-code` transport hosts and a 240000 input-token cap for each without recording the bearer key.

A second generated branch preserves the same diagnostic blob hash, providing a repeated snapshot of the same 41-alias / 71-deployment inventory state.

**Proposed claim:** The staging-labelled LiteLLM workflow has now mechanically demonstrated an authenticated no-spend gateway inventory path: all three diagnostic endpoints returned HTTP 200, with 41 public aliases and 71 deployment rows.

### G4 — where the Codex/master-key mismatch was introduced

The workflow history cleanly separates the original policy from the later change.

At commit `a3d8058...`, the master key was diagnostic-only and Codex received `OPENAI_API_KEY`. This matched `DEPLOYMENT.md`.

At commit `84b9c76...`, the workflow changed:

```text
Codex openai-api-key:
  OPENAI_API_KEY
  -> LITELLM_MASTER_KEY

Codex endpoint:
  default
  -> selected LiteLLM /v1/responses

Codex model:
  implicit/default
  -> gpt-5.6-sol
```

The deployment documentation was not changed with that authority expansion.

**Proposed claim:** `LITELLM_MASTER_KEY` entered the protected workflow at commit `a3d8058fd0ced9624907501e12b02f6544e5470c` for diagnostic use; commit `84b9c7697b93e5826bfb11bfacbbd3cb941328d7` later expanded that same master key into the Codex model-running step, creating the current documentation/code authority-boundary mismatch.

This pass does not prescribe a credential redesign. It establishes the exact point of divergence so the boundary can be intentionally resolved and then tested.

### G5 — model inventory is not epistemic identity

The 71 rows are deployments, not 71 independent models.

The live snapshot contains repeated aliases such as:

```text
gpt-5.5         x3
gpt-5.6-sol     x3
gpt-5.6-terra   x3
kimi-k2.7-code  x4
glm-5-turbo     x4
mimo-v2.5-pro   x4
minimax-m3      x4
qwen-3.6-max    x4
qwen3.6-plus    x4
qwen3.7-flash   x4
qwen3.8-max     x4
```

The checked-in configuration explains that repeated `model_name` rows form route/key pools and that a pool is not intended to span different upstream model ids.

Therefore counts must remain separated:

```text
deployment count != alias count != distinct model identity count != independent verifier count
```

**Proposed claim:** A LiteLLM alias or transport deployment must not be counted as an independent epistemic model identity; independence is a separately evidenced property of the actual model/family and its measured error behavior.

### G6 — requested alias is no longer sufficient provenance

After the first FOSSIL LiteLLM audit, `litellm-ckff-ops` merged PR #4, adding a Responses-to-Chat bridge with bounded sequential cross-model fallbacks.

Its API contract explicitly emits:

```text
bridge_requested_model
bridge_actual_model
bridge_attempts[]
```

A request can therefore succeed with HTTP 200 while being executed by a different model from the requested alias.

Example contract shape:

```json
{
  "bridge_requested_model": "[aws]deepseek-v3.2",
  "bridge_actual_model": "qwen3.7-flash",
  "bridge_attempts": [
    {"model": "[aws]deepseek-v3.2", "status": 503},
    {"model": "qwen3.7-flash", "status": 200}
  ]
}
```

This is operationally useful for availability but dangerous if V4 attributes evidence to the requested model.

**Proposed claim:** Every V4 inference receipt must preserve requested model, actual model, and fallback attempts, and any epistemic attribution must follow the actual model used rather than the requested alias.

Exact-identity tests should disable fallback or reject substitutions at admission/closeout.

## Proposed minimal `LiteLLMRouteManifest`

The manifest should be a normalized observation of the gateway, not a hand-authored list of “agents”.

```text
LiteLLMRouteManifest
  schema_version
  observed_at
  gateway_receipt
    base_host
    health_status
    models_status
    model_info_status
    source_digest

  aliases[]
    alias_id
    public_name
    deployment_ids[]
    mode/capabilities if observed

  deployments[]
    deployment_id
    alias_id
    upstream_model_hint
    transport_host
    route_label
    capability_metadata
    source_row_digest

  epistemic_identities[]
    epistemic_identity_id
    canonical_model_id | UNKNOWN
    family_id | UNKNOWN
    supporting_deployment_ids[]
    identity_evidence_refs[]
    independence_group_id | UNKNOWN

  routing_policy
    fallback_semantics
    exact_identity_supported
    requested_actual_receipt_required
```

The key rule is fail-closed identity resolution. Route labels such as `[aws]`, `[grok]`, alternate API hosts, multiple credentials, or multiple deployment rows are transport observations. They are not sufficient evidence for a distinct model family or independent error process.

**Proposed claim:** V4 should admit models through a normalized `LiteLLMRouteManifest` that preserves separate alias, deployment, transport-route, and epistemic-identity fields, with unknown identity failing closed rather than guessed from route count.

## Reservoir admission contract

A route becomes eligible for epistemic scheduling only after these checks:

```text
INVENTORY_OBSERVED
  authenticated metadata endpoints succeeded
  exact raw observation is content-addressed

IDENTITY_CLASSIFIED
  requested alias preserved
  deployment/route preserved
  actual model identity either evidenced or UNKNOWN

CAPABILITY_PROBED
  bounded task-specific probe if the role requires tools/JSON/etc.

RECEIPT_CAPABLE
  requested model + actual model + fallback attempts can be recorded

ADMITTED
  scheduler may assign a role subject to policy
```

`UNKNOWN` identity can still be usable for ordinary execution, but it cannot create an “independent verifier” count.

## Falsifiable tests for the first adapter

A first implementation is sufficient if it passes these deterministic cases:

1. Four `kimi-k2.7-code` deployment rows on different hosts produce one alias with four transport deployments, not four epistemic identities.
2. Three `gpt-5.6-sol` deployment rows differing only by credential/route produce one alias pool, not three independent reviewers.
3. An HTTP 200 response with `requested=A`, `actual=B` attributes the observation to B and preserves A plus the failed/attempted route chain.
4. A response without actual-model provenance when fallback is possible is not eligible for model-specific verification credit.
5. A route/model name that cannot be mechanically mapped to a canonical identity remains `UNKNOWN`; string similarity alone does not promote it.
6. A gateway inventory snapshot with failed health or inventory status is not admitted as a fresh reservoir snapshot.

These tests are intentionally narrower than a general multi-agent framework.

## Effect on the earlier FOSSIL audit

The first audit remains a valid historical snapshot. Its statement that secret presence and live gateway authentication were not proven described the evidence available at that time.

This pass changes the adjudication:

```text
source wiring                         PROVEN BY SOURCE
secret exact name available at run   OBSERVED
selected gateway URL usable at run   OBSERVED
no-spend inventory authentication    OBSERVED
41 aliases / 71 deployments          OBSERVED SNAPSHOT
GitHub secret storage scope          UNKNOWN
Cortex V4 direct secret consumer     STILL NOT ESTABLISHED
epistemic model identity count       NOT DERIVABLE FROM ROUTE COUNT
requested==actual under fallback     FALSE IN GENERAL
```

## Current adjudication

The LiteLLM operational gate is sufficiently proven to stop treating gateway reachability as the blocking V4 uncertainty.

The next implementation gate should be the small, deterministic route-manifest/model-reservoir adapter above, with requested-vs-actual provenance and conservative identity classification. After that, the assurance work can move to `AssuranceWorkOrder`, `WorkEvent`, authority states, artifact/version identity, leases/fencing, durable intent/effect/decision, and the independent reference model.

Do not benchmark “heterogeneous multi-model” behavior until the harness can prove which actual model executed each contribution and can distinguish transport multiplicity from epistemic independence.
