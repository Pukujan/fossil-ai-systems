# Gravebuster Bootstrap Handoff — Cortex V4 / FOSSIL executable lab

**Date:** 2026-08-10  
**Host role:** temporary local FOSSIL/V4 research worker  
**Do not treat as:** permanent availability node, sole canonical storage, or production deployment  
**Parent planning checkpoint:** `docs/research/2026-08-10-v4-executable-lab-storage-worker-topology-checkpoint.md`  
**Planning source snapshot:** `snap_753b6af8c8ee6cc36dc2eaaf`  
**Planning artifact:** `art_c8d1c45fc9b5a3ddfee568c40b19e558da189768`

## Objective

Establish whether Gravebuster can serve as a **reproducible, isolated, provenance-complete temporary research worker** for Cortex V4/FOSSIL before the Endomorphosis/JusticeDAO executable-lab campaign is split across parallel specialist sessions.

The terminal condition is not “finish Cortex V4.”

Terminal condition:

```text
local repos pinned and healthy
    + deterministic FOSSIL/V4 tests green
    + isolated worker/workspace root
    + current LiteLLM no-spend preflight green
    + one bounded real model worker executed
    + requested/actual/fallback provenance captured
    + no secret leakage
    + compact durable receipt preserved
```

If any required piece cannot be established, finish as **BLOCKED with evidence**, not by weakening the gate.

## Read first

1. `Pukujan/fossil-ai-systems` PR #4, branch `agent/v4-assurance-research-capture`.
2. `docs/research/2026-08-10-v4-executable-lab-storage-worker-topology-checkpoint.md`.
3. `docs/handoffs/2026-08-10-v4-assurance-session-handoff-v5.md` for the last mechanically exercised V4 assurance baseline.
4. `Pukujan/cortex-v4` PR #6, branch `agent/durable-v4-control-plane`.
5. Current SSC `cortex_core.model_summon` / `cortex_core.agent_runtime` implementation.
6. Current LiteLLM configuration/documentation; do not assume the 2026-08-06 local route example is still current.

## Important current facts

The last known mechanically exercised Cortex V4 assurance head before this bootstrap was:

- repo: `Pukujan/cortex-v4`;
- branch: `agent/durable-v4-control-plane`;
- head: `a2a3431ec235f143c8aadeb98bc47e1126c45327`;
- CI: 80 tests passed in 7.66s.

Do not fail merely because the current branch has advanced and the test count is larger. Record the actual current head/test count and investigate regressions if the current suite fails.

The old V4 live LiteLLM script must **not** be used unchanged. It still reflects legacy mechanics: hard-coded Windows paths, Desktop `.env` discovery, SSC delegation, a Grok-specific example route, and a non-queryable remote receipt field.

The existing SSC `summon_agent(...)` path is still the candidate worker execution mechanism. The bootstrap must inspect its **current** behavior rather than reimplementing an agent framework.

## Phase A — inventory Gravebuster without mutating repos

Start by recording:

```text
hostname
OS / architecture
available disk
RAM
Python version
Git version
current working directories
existing repo locations
```

For every relevant repository, record:

```bash
git status --short --branch
git remote -v
git rev-parse HEAD
```

Repositories:

- `fossil-core`;
- `fossil-ai-systems`;
- `cortex-v4`;
- `stupidly-simple-cortex` / SSC;
- any already-present Endomorphosis repos.

**Do not hard reset, clean, stash, or switch branches before inspecting dirty state.** Preserve unrelated local work.

## Phase B — create an isolated local lab root

Do not use hard-coded project paths in new code.

Choose a configurable root, for example:

```bash
export V4_LAB_ROOT="$HOME/v4-lab"
mkdir -p "$V4_LAB_ROOT"/{repos,worktrees,datasets,hf-cache,model-cache,experiments,fossil-objects,db,tmp}
```

If the host uses a different preferred disk/path, use that instead and record it in the bootstrap receipt.

Suggested environment variables:

```bash
export HF_HOME="$V4_LAB_ROOT/hf-cache"
export HF_HUB_CACHE="$V4_LAB_ROOT/hf-cache/hub"
```

Do not commit machine-specific absolute paths or credentials.

## Phase C — pin/update code safely

Fetch remote refs first. Reuse existing clones where healthy.

For active work, prefer isolated worktrees rather than switching a dirty primary checkout.

Required working surfaces:

- current `fossil-core` main;
- `fossil-ai-systems:agent/v4-assurance-research-capture`;
- `cortex-v4:agent/durable-v4-control-plane`;
- current SSC main/source branch used by the summon runtime.

Record exact resolved SHAs in the receipt. Do not assume a branch name proves a specific revision.

## Phase D — deterministic baseline before provider access

### FOSSIL

Create/use an isolated Python environment and run the current normal `fossil-core` test suite according to the repository instructions.

Record:

```text
repo SHA
test command
pass/fail count
duration
runtime versions
```

Do not add hosted-model dependencies merely to make the core suite run.

### Cortex V4

Run the current contract suite on the PR #6 branch.

At minimum the assurance surface should still cover:

- route/model fallback provenance;
- provider-source authority/freshness/contradiction;
- secret sanitization;
- durable assurance store recovery;
- direct mutation crash/restart/fencing;
- target-side fencing/idempotency;
- partial cross-system lease mismatch blocking;
- direct/process conformance and strict serialization.

If current tests fail, stop the live-provider phase until the regression is understood.

## Phase E — inspect the current LiteLLM/SSC worker path

Before making a real model request, inspect the current implementations/configuration for:

- `summon_agent` entry point;
- seat/model resolution;
- dispatch chain;
- timeout ownership;
- retry ownership;
- fallback behavior;
- workspace/tool write restrictions;
- how actual upstream model identity is exposed, if at all;
- streaming behavior;
- current environment variable/config names.

Do not assume route multiplicity is epistemic independence.

Do not assume a requested alias identifies the actual upstream model.

## Phase F — explicit current LiteLLM configuration

Do not read a personal Desktop `.env` file as a hidden dependency.

Use explicit local environment/configuration supplied to the worker process. Keep secrets out of committed files, shell history where practical, logs, and artifacts.

Resolve the current staging/eval gateway configuration from the maintained repo/docs/environment. The expected secret/config names may have evolved; inspect before setting them.

Before any paid/generative request, run only the no-spend metadata probes:

```text
/health/liveliness
/v1/models
/model/info
```

Required outcome:

- all required metadata probes succeed;
- sanitized route manifest can be built;
- no bearer key/provider credential is preserved in the manifest;
- alias/deployment multiplicity is not counted as verifier independence.

If metadata is contradictory, incomplete, stale, or under-authority, fail closed.

## Phase G — repair/build the assured research-worker boundary

Do not create a new multi-agent framework.

Wrap the existing worker mechanism so one bounded invocation produces an inspectable V4/FOSSIL-compatible receipt containing at least:

```text
work_order_id
worker/host identity
role
input/prompt CID
workspace/artifact revision
requested model/alias
actual model if observable
fallback attempts
start/end timestamps
deadline/timeout
result/output CID
tool/workspace writes
status/outcome
trace/run reference
```

Rules:

- model output has candidate/testimony authority only;
- missing actual identity under fallback means no model-specific epistemic credit;
- explicit substitution must remain visible;
- no success is returned merely because the model process ended;
- any external mutation must go through the existing durable/fenced assurance path or be excluded from the bootstrap task.

The first live task should therefore be **workspace-local and bounded**.

## Phase H — one real bounded model worker

Choose the model from the **current observed gateway inventory**, not from the old hard-coded Grok example.

Prefer a cheap current model for the smoke test unless a stronger model is required to exercise the response bridge. Record requested and actual identities separately.

The task should only read/write inside its isolated experiment workspace and produce a tiny deterministic artifact contract that can be checked mechanically.

Example shape:

```text
read 1-3 local fixture files
produce one JSON receipt/artifact
read it back
mechanically validate schema/hash
finish
```

Do not let the worker modify production repos, GitHub, external services, or the FOSSIL canonical store during this first smoke test.

## Phase I — security/provenance checks

After the live run:

- inspect generated files/logs for credential-shaped strings;
- confirm the master key/provider secrets are absent;
- confirm requested vs actual model/fallback state is not silently collapsed;
- confirm all workspace writes are within the declared write set;
- hash the input and output artifacts;
- record timing/attempts without storing raw secret-bearing HTTP metadata.

A transport success without a trustworthy actual-model receipt is **operational success with provenance limitation**, not independent model evidence.

## Phase J — bounded retry/crash check

Exercise at least one interrupted/retried worker execution path.

The goal is not yet full joined convergence. The bootstrap only needs to demonstrate that an interrupted worker does not become a false completed/committed result and that a retry is identifiable/recoverable.

If external effects are introduced by the wrapper, stop and route them through the durable mutation contract before continuing.

## Required bootstrap receipt

Write one compact machine-readable receipt under an isolated experiment directory, for example:

```text
$V4_LAB_ROOT/experiments/gravebuster-bootstrap-<timestamp>/bootstrap-receipt.json
```

Minimum fields:

```json
{
  "schema": "cortex.v4.gravebuster-bootstrap.v1",
  "host": {},
  "repo_revisions": {},
  "tests": {},
  "litellm_preflight": {},
  "worker_probe": {},
  "security_checks": {},
  "retry_or_crash_probe": {},
  "result": "PASS | BLOCKED",
  "blockers": []
}
```

Then preserve the compact receipt/result back into FOSSIL with explicit authority. Do not ingest huge transient logs as canonical knowledge; keep trace references to them.

## PASS criteria

PASS only if:

- deterministic FOSSIL and V4 baselines are green;
- current LiteLLM metadata preflight is green;
- worker workspace isolation is demonstrated;
- one bounded real worker completes its mechanical artifact contract;
- requested/actual/fallback provenance is preserved to the extent the gateway exposes it;
- secrets are absent from preserved outputs;
- interrupted/retried execution does not create false success;
- a compact receipt is durably preserved.

## BLOCKED criteria

Return BLOCKED with evidence if, for example:

- current LiteLLM configuration cannot be resolved without hidden local assumptions;
- metadata endpoints fail or contradict each other;
- actual model identity is unavailable in a way that prevents the intended attribution;
- SSC/V4 timeout/retry/fallback ownership is ambiguous enough to risk duplicate work;
- local tests regress;
- secrets appear in logs/artifacts;
- the worker cannot be confined to a declared workspace;
- retry/crash can be mistaken for committed success.

Do not bypass the blocker just to start the fork campaign.

## After PASS

Only after this bootstrap is green:

1. create the durable executable-lab campaign issue tree;
2. freeze S0 protocol/revisions;
3. start independent specialist sessions for `ipfs_datasets_py`, `ipfs_kit_py`, MCP++, JusticeDAO data, and negative controls;
4. use the assured worker wrapper for bounded LiteLLM hypothesis/attack/test generation;
5. keep mechanical harness outcomes separate from model testimony;
6. later move the same worker contract to a persistent VPS/self-hosted runner without changing the evidence semantics.

## Final handoff back to the coordinating session

Return exactly these items:

- Gravebuster environment summary;
- exact repo SHAs;
- commands executed;
- FOSSIL/V4 test results;
- sanitized LiteLLM metadata/preflight result;
- current summon/retry/fallback findings;
- worker receipt path/hash;
- requested and actual model identity/fallback trace;
- credential-leak scan result;
- retry/crash result;
- PASS or BLOCKED;
- concrete blockers or next issue candidates.

Do not return only narrative confidence. Return inspectable artifacts/receipts.
