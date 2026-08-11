# LiteLLM Secret and Multi-Model Wiring Audit — 2026-08-10

**Scope:** `Pukujan/litellm-ckff-ops` and `Pukujan/cortex-v4`
**State:** source-inspection evidence; credential existence/value not independently observable through the available GitHub connector
**Authority:** working operational finding, not production certification

## Research question

Does adding a LiteLLM credential to GitHub secrets make the current Cortex V4 harness able to use the existing multi-model gateway, and what remains unproven?

## Inspected source surfaces

- `Pukujan/litellm-ckff-ops/.github/workflows/sol-liteLLM-repair.yml`
- `Pukujan/litellm-ckff-ops/scripts/live_diagnose.py`
- `Pukujan/litellm-ckff-ops/config/config.yaml`
- `Pukujan/litellm-ckff-ops/DEPLOYMENT.md`
- `Pukujan/cortex-v4/scripts/run_v4_live_litellm.py`
- `Pukujan/cortex-v4/README.md`

## Finding 1 — the existing GitHub workflow has a real LiteLLM credential path

`sol-liteLLM-repair.yml` reads a GitHub environment secret named exactly `LITELLM_MASTER_KEY`.

The diagnostic step receives:

```text
LITELLM_URL = staging or production URL variable
LITELLM_MASTER_KEY = GitHub secret
```

and runs `scripts/live_diagnose.py`.

The same workflow currently also passes `LITELLM_MASTER_KEY` into the Codex action as `openai-api-key`, while pointing the Responses API endpoint at the selected LiteLLM `/v1/responses` URL.

**Operational claim:** The current `litellm-ckff-ops` workflow has a source-wired authentication path from `LITELLM_MASTER_KEY` to the LiteLLM gateway, provided the secret name and environment URL variables are configured correctly.

## Finding 2 — the diagnostic can prove gateway reachability and inventory without spending model tokens

`live_diagnose.py` requires `LITELLM_URL` and `LITELLM_MASTER_KEY`, sends the key as a Bearer token, and queries:

- `/health/liveliness`
- `/v1/models`
- `/model/info`

It reports status codes, model count, deployment count, and selected route metadata without printing credentials or response bodies.

This is a useful preflight surface for V4 because it can mechanically distinguish:

```text
credential configured
gateway reachable
model inventory visible
specific routes present
```

from:

```text
model inference actually succeeds
required model/tool semantics work
cross-vendor verification is available
```

Those are different gates.

## Finding 3 — the checked-in LiteLLM configuration is genuinely multi-model

`config/config.yaml` defines a large OpenAI-compatible model list spanning multiple families/vendors and route pools, including Claude, Gemini, GPT, Grok, DeepSeek, Kimi, GLM, Qwen, MiniMax, and related aliases.

Some aliases have multiple route/key entries, so transport failover/pooling and epistemic model diversity must remain separate concepts.

**Operational claim:** The LiteLLM service configuration is a credible model-reservoir source for Cortex V4, but route multiplicity must not be counted as independent epistemic verification.

## Finding 4 — Cortex V4 is not yet directly wired to the GitHub LiteLLM secret

The current `cortex-v4/scripts/run_v4_live_litellm.py` does not read a repository GitHub secret directly. It:

1. loads a Windows Desktop `.env` if present;
2. imports the older SSC `cortex_core.model_summon` path;
3. writes a temporary route table;
4. invokes one configured seat using `[grok] grok-4.5`.

Repository search did not surface a direct `LITELLM_URL` / `LITELLM_MASTER_KEY` consumer inside Cortex V4 itself.

**Operational claim:** Adding `LITELLM_MASTER_KEY` to `cortex-v4` repository secrets alone does not currently activate V4 multi-model routing; the existing live V4 path still depends on SSC/local environment wiring and a bounded single-route attachment.

## Finding 5 — secret presence/value is not proven by this audit

GitHub deliberately does not expose secret values, and the available GitHub connector in this session does not expose repository/environment secret metadata or `workflow_dispatch`.

Therefore this audit proves the source wiring, not that the newly added secret exists under the expected name, is attached to the intended environment, or authenticates successfully.

The strongest next no-spend proof is a staging diagnostic run that records:

```text
health/liveliness = 200
/v1/models = 200 with nonzero model_count
/model/info = 200 with expected deployment/model aliases
```

without exposing the key.

## Finding 6 — deployment documentation and workflow disagree about the Codex credential boundary

`DEPLOYMENT.md` says GitHub environments should contain:

- `LITELLM_MASTER_KEY` for the redacted diagnostic step
- `OPENAI_API_KEY` for the Codex Action

and says the agent should not receive the master key.

However the current workflow passes `LITELLM_MASTER_KEY` directly to the Codex action as its API key.

**Operational claim:** The LiteLLM deployment documentation and executable workflow currently disagree about whether the model-running Codex step may receive the LiteLLM master key; this authority boundary should be resolved before the workflow is treated as frozen production policy.

## V4 implication

The desired next architecture is not to hard-code every model into Cortex. V4 should consume a gateway inventory/route manifest and convert it into a controlled model reservoir.

Candidate separation:

```text
LiteLLM
    provider/model transport, aliases, pooling, availability

V4 model reservoir
    stable model identity
    vendor/family metadata
    task capabilities
    historical failure profile
    independence constraints
    allowed epistemic roles

V4 scheduler
    chooses producer/attacker/verifier/test-designer roles

FOSSIL
    records route/model provenance privately
    records evidence, outcomes, error correlations, and verification history
```

The first mechanical gateway integration should therefore be:

```text
PRECHECK
  -> authenticate to LiteLLM
  -> enumerate `/v1/models` and `/model/info`
  -> canonicalize a route manifest
  -> classify aliases vs distinct model families
  -> mark unavailable/unknown routes explicitly

PROBE
  -> bounded no/low-cost model capability probes
  -> collect model/route receipts

ADMIT
  -> only admitted routes enter the V4 model reservoir
```

A fallback route used because another route is unavailable must not automatically count as an independent verifier.

## Current adjudication

The GitHub secret can plausibly unlock the existing LiteLLM workflow if it is named `LITELLM_MASTER_KEY` and the target environment has the corresponding LiteLLM URL variable. The LiteLLM configuration already exposes a broad multi-model fleet.

What is **not yet proven** is that the new secret is present in the intended repository/environment, that the gateway accepts it, or that Cortex V4 directly consumes it. V4's own multi-model reservoir remains a next implementation gate rather than an achieved state.
