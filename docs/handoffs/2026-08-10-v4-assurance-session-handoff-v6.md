# Cortex V4 / FOSSIL Assurance Research — Session Handoff v6

**Date:** 2026-08-10  
**Supersedes:** v5 for current trajectory and next action  
**Research repo/PR:** `Pukujan/fossil-ai-systems` #4, branch `agent/v4-assurance-research-capture`  
**Implementation repo/PR:** `Pukujan/cortex-v4` #6, branch `agent/durable-v4-control-plane`  
**Last known mechanically exercised V4 head:** `a2a3431ec235f143c8aadeb98bc47e1126c45327`  
**Last known V4 CI at that head:** 80 passed in 7.66s

## Trajectory correction

Do not continue v5's seven engineering gates as an arbitrary implementation checklist.

The near-term trajectory is now:

```text
make the existing V4/SSC worker path trustworthy
    -> use Gravebuster as temporary research worker
    -> fork/pin Endomorphosis/JusticeDAO executable subjects
    -> run and attack upstream harnesses directly
    -> preserve receipts/minimized divergences in FOSSIL
    -> synthesize transfer decisions
    -> run upstream/reference/V4 differential conformance
    -> only then freeze more V4 architecture
```

The broader V4 architecture is still:

1. adversarial evidence/hypothesis search;
2. crash-recoverable, provenance-preserving mechanical assurance underneath it.

Model agreement, vendor count, LiteLLM route multiplicity, and requested aliases do not gain proof authority.

## New durable checkpoint

Read:

`docs/research/2026-08-10-v4-executable-lab-storage-worker-topology-checkpoint.md`

FOSSIL identity:

- artifact: `art_c8d1c45fc9b5a3ddfee568c40b19e558da189768`
- SHA-256: `c8d1c45fc9b5a3ddfee568c40b19e558da189768239980ad7ccd8cbd4264f693`
- source snapshot: `snap_753b6af8c8ee6cc36dc2eaaf`
- parent synthesis: `snap_95fc700129099b2709dd5631`

This is a **derived reconstructed planning checkpoint**, not a verbatim transcript and not a promoted architecture claim.

## Immediate next action

Use **Gravebuster as the temporary local FOSSIL/V4 research worker**.

The exact local-session handoff is:

`docs/handoffs/2026-08-10-v4-executable-lab-gravebuster-bootstrap.md`

Do not start the parallel Endomorphosis fork campaign until this local readiness gate is PASS.

## Why the readiness gate exists

The current V4 assurance substrate is mechanically useful, but the old live LiteLLM runner remains legacy-shaped:

- hard-coded Windows paths;
- Desktop `.env` discovery;
- SSC runtime dependency;
- old Grok-specific route example;
- remote receipt not queryable in the preserved runner.

Therefore the bootstrap must inspect the **current** SSC summon path and **current** LiteLLM configuration, then prove:

- deterministic FOSSIL/V4 tests;
- explicit isolated worker workspace;
- no-spend LiteLLM metadata preflight;
- one bounded real worker;
- requested/actual/fallback provenance;
- no secret leakage;
- compact durable receipt;
- retry/crash does not create false completion.

If those cannot be shown, return BLOCKED with evidence.

## Planned parallel sessions after readiness PASS

- **S0:** campaign protocol and exact revision freeze.
- **S1:** `ipfs_datasets_py` authority/differential/metamorphic/fixed-point harnesses.
- **S1b:** JusticeDAO data/retrieval/recovery corpus.
- **S2:** `ipfs_kit_py` WAL/recovery/idempotency/compensation.
- **S3:** MCP++ causal DAG/leases/fencing.
- **S4:** negative controls / weak HA-agent patterns.
- **S5:** assured V4/SSC LiteLLM research-worker wrapper.
- **S6:** mechanism transfer synthesis.
- **S7:** upstream vs V4 reference vs V4 implementation differential conformance.
- **S8:** FOSSIL ingest/authority reconciliation.

S1/S1b/S2/S3/S4 should become independent specialist sessions after S0. S5 can proceed in parallel once the bootstrap clarifies the current worker surface.

## Storage direction

Do not store large Hugging Face datasets or model caches in GitHub.

Current intended layering:

```text
GitHub
  code / schemas / issues / lockfiles / compact receipts / small reproducers

worker/storage host
  repo mirrors / worktrees / HF cache / selected datasets / experiments
  FOSSIL operational objects / rebuildable local DBs and indexes

FOSSIL canonical boundary
  immutable evidence identity + stable IDs + append-only events + provenance/history
```

Vector PQ/SQ, HNSW, CSR and similar compression/index mechanisms are candidates for **rebuildable projections**. Lossy projection compression must not redefine canonical evidence identity.

Large canonical blobs may later use lossless storage encoding such as Zstandard while retaining identity over original/decompressed bytes; this is not yet mechanically implemented/proven.

## Infrastructure direction

Longer term, use a persistent execution/storage node plus independent versioned/offsite backup. The user's main GPU PC is not assumed to be an always-on worker; its spare storage is a future backup/replica candidate.

For now, do not block on final distributed storage topology. Gravebuster is the temporary worker used to learn and validate the research-worker contract.

## What remains from v5

The v5 gates are not discarded. They are deferred/reframed:

- real mutation boundary;
- compensation execution;
- lease handoff receipt;
- joined convergence;
- closeout/promotion;
- live provider gate;
- matched benchmark.

They should be resumed when executable upstream research shows which mechanisms and invariants actually deserve transfer into V4.

## Next-session terminal condition

The next local session must return:

```text
Gravebuster readiness = PASS or BLOCKED
exact repo SHAs
FOSSIL/V4 test results
sanitized LiteLLM preflight
current summon/retry/fallback findings
bounded worker receipt hash/path
requested vs actual model/fallback trace
secret-leak scan
retry/crash result
concrete blockers / next issue candidates
```

Do not return only narrative confidence.
