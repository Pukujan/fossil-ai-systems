# Cortex V4 executable-lab, storage, and worker-topology checkpoint

**Date:** 2026-08-10  
**Status:** reconstructed planning checkpoint; not a verbatim chat transcript  
**Authority:** derived planning evidence only; no architecture or deployment claim is promoted by this document  
**Parent synthesis:** `snap_95fc700129099b2709dd5631` (`v4-assurance-working-synthesis-2026-08-10`)

## Why this checkpoint exists

The active conversation moved beyond the prior implementation-only gate list. The near-term trajectory is now to use existing Endomorphosis/JusticeDAO code and data as an **executable research corpus**, drive the real upstream harnesses adversarially, and use Cortex V4/FOSSIL to preserve reproducible evidence before freezing more V4 architecture.

This checkpoint preserves the decisions and open questions that were otherwise only present in the live conversation. It is intentionally reconstructed. Exact lost wording is not claimed.

## Current trajectory

The working architecture remains two-layered:

1. **Epistemic search:** competing hypotheses, attacks, evidence acquisition, counterexamples, discriminating tests, and bounded search.
2. **Execution assurance:** typed authority, durable intent/effect/decision, causal provenance, epochs/fencing, crash recovery, independent reference-model conformance, and fail-closed promotion.

The new execution strategy is:

```text
fork / pin upstream
    -> run real harness
    -> instrument
    -> attack / crash / mutate
    -> minimize divergence
    -> preserve receipt
    -> compare with V4 reference model
    -> adapt / rewrite / reject mechanism
```

The goal is not to downstream-fork the whole Endomorphosis stack. Upstream repositories are experimental subjects. Literal source reuse is decided only after behavior, licensing, hidden assumptions, and V4 semantic requirements are understood.

## Planned specialized-session campaign

### S0 — campaign/bootstrap protocol

Freeze:

- campaign ID;
- exact upstream revisions;
- common experiment/receipt schema;
- authority classes;
- bounded time/cost/concurrency limits;
- required minimized reproducer for failures;
- transfer outcomes: `reuse_candidate`, `adapt`, `rewrite_small_reference`, `reject`, `needs_more_evidence`;
- prohibition on model testimony upgrading evidence authority.

S0 is the only hard dependency for the parallel fork sessions.

### S1 — `ipfs_datasets_py` logic/harness specialist

Reproduce and attack:

- verifier authority ceilings;
- differential disagreement -> inconclusive;
- metamorphic/preservation checks;
- fuzzing/minimization;
- objective refill/fixed point;
- bounded resource gates.

Deliver baseline receipts, adversarial traces, minimized failures, hidden assumptions, and transfer recommendations.

### S1b — JusticeDAO executable-data/retrieval specialist

Candidate datasets observed during the conversation include:

- `justicedao/ipfs_uscode`;
- `justicedao/legal-ir-autoencoder-checkpoints`;
- `justicedao/patent-legal-ir-graphrag`;
- the decomposed patent corpus/BM25/vector/graph datasets;
- `justicedao/wetwijzer_netherlands_legal_corpus`;
- `justicedao/ipfs_court_rules`;
- larger later-stage corpora such as state laws/admin rules/federal register/caselaw.

Use Hugging Face as the upstream object store. Before every experiment, pin exact repository revisions and selected paths/hashes. Dataset sizes, viewer failures, licenses, and availability discussed in the live conversation must be revalidated at execution time rather than treated as frozen facts.

`ipfs_uscode` is especially interesting as a naturally evolving recovery corpus because prior inspection surfaced recovery manifests, candidate patches, validation/application state, parent-patch fields, and schema drift.

### S2 — `ipfs_kit_py` WAL/recovery specialist

Reproduce durable intent/effect/decision, replay, compensation, and idempotency. Crash at every state-changing boundary; duplicate and corrupt recovery state; identify the exact assumptions required for recovery correctness.

### S3 — MCP++ DAG/fencing specialist

Drive causal event DAGs, canonical CIDs, missing parents, duplicates, partitions, lease expiry, takeover, fencing, stale completion, and reconciliation. Distinguish deterministic specification/harness evidence from production distributed-system claims.

### S4 — negative-control specialist

Deliberately study weaker or incomplete components (including prior HA and `ipfs_agents_py` negative findings) and build regression cases for systems that appear durable/distributed/verified but are not.

### S5 — V4 research-worker specialist

Do **not** invent a new agent framework. Start from the existing SSC/V4 `summon_agent` path into LiteLLM, then wrap it in the new V4 assurance contracts:

- WorkOrder identity;
- requested model;
- actual model;
- fallback attempts;
- role;
- input/output CIDs;
- deadlines;
- artifact revision;
- lease/fence;
- durable execution receipt.

The wrapper supplies workers; it does not grant epistemic authority.

### S6 — synthesis

Consume standardized outputs from S1/S1b/S2/S3/S4. Produce a mechanism transfer matrix. Architectural decisions happen here, not inside the fork-specialist sessions.

### S7 — differential conformance

Run matched scenarios through:

```text
upstream implementation
V4 small reference model
V4 real implementation
```

Normalize traces. Any semantic divergence is preserved and minimized instead of voted away.

### S8 — FOSSIL ingest

Preserve exact upstream revisions, experiment receipts, minimized failures, transfer decisions, and evidence authority. This remains separate from experimental session churn.

## V4 readiness warning

The current V4 assurance substrate is useful, but the old live LiteLLM runner is **not yet an acceptable research-driver interface**.

The currently preserved legacy runner still has:

- hard-coded Windows paths;
- dependency on SSC as the live summon implementation;
- Desktop `.env` loading;
- a Grok-specific route-table example;
- `remote_receipt: "not_queryable"` in its observation output.

Therefore **V4 research-driver readiness is a gate before parallel executable-lab work**.

The gate must demonstrate, on the chosen worker host:

1. deterministic V4/FOSSIL local tests pass;
2. worker identity and workspace isolation are explicit;
3. current LiteLLM no-spend metadata preflight succeeds using explicit environment/config, not Desktop `.env` discovery;
4. requested-vs-actual model and fallback attempts are recorded;
5. one bounded real summon succeeds through the current gateway;
6. provider secrets do not appear in logs/artifacts;
7. result is converted into the V4 durable receipt/WorkEvent surface;
8. retry/crash behavior does not silently duplicate external effects.

## Storage architecture retained from the conversation

Do not put large Hugging Face datasets, model caches, or rebuildable indexes into the Git repositories.

### GitHub

Keep:

- code and schemas;
- issues/PRs/project tracking;
- experiment definitions;
- dataset lockfiles;
- hashes/CIDs;
- compact FOSSIL events/receipts;
- small fixtures and minimized reproducers.

### Worker/storage host

Keep:

- upstream repo mirrors/worktrees;
- Hugging Face caches;
- selected datasets;
- model caches when needed;
- experiment workspaces;
- FOSSIL operational object store;
- local metadata/index databases.

### FOSSIL canonical boundary

Canonical knowledge remains based on immutable evidence identity + stable IDs + append-only validated events + provenance/history. SQL, BM25, vector indexes, graph databases, and other retrieval projections remain replaceable/rebuildable.

### Compression

The user-supplied `AI Retrieval & Graph Architectures Guide` introduced useful **projection** candidates: metadata prefiltering, scalar/product quantization for vectors, HNSW-style vector search, and compressed graph layouts such as CSR. These are candidate benchmark mechanisms for rebuildable projections, not a license to lossy-compress canonical source evidence.

For canonical large blobs, the current proposal is lossless storage encoding (for example Zstandard) while identity remains the hash of the original/decompressed bytes. This is a proposal; FOSSIL does not yet mechanically prove a canonical blob-compression layer.

## Worker-topology direction

Longer term, the preferred topology is:

```text
GitHub
  -> persistent execution/storage node
      -> FOSSIL objects + datasets + experiments + rebuildable DBs
      -> optional compute workers
      -> independent versioned/offsite backup
```

The user's main GPU PC should not be assumed to be an always-on worker because it is also the daily-use machine and has constrained RAM. Its spare storage is a strong future backup/replica candidate.

A persistent node plus a versioned/offsite backup is preferable to making any one workstation the sole copy of FOSSIL. Replication and backup are distinct: a naive mirror can replicate accidental deletion, so durable evidence eventually needs versioned/immutable retention and verified content hashes.

## Immediate bootstrap decision: Gravebuster first

For the next local step, **Gravebuster is the temporary FOSSIL/V4 research worker**.

Do not wait for the final VPS/backup topology before learning whether the local research-driver contract actually works. Do not treat Gravebuster as the permanent availability or canonical-storage solution.

The first Gravebuster milestone is purely a readiness proof:

```text
local repos healthy
    -> deterministic FOSSIL/V4 tests
    -> isolated v4-lab workspace
    -> current LiteLLM metadata preflight
    -> one bounded assured model worker
    -> compact durable receipt
    -> no secret leakage
```

Only after this is green should S1/S1b/S2/S3/S4 begin in parallel.

## Suggested Gravebuster local layout

Use a configurable root rather than hard-coded machine paths:

```text
$V4_LAB_ROOT/
  repos/
  worktrees/
  datasets/
  hf-cache/
  model-cache/
  experiments/
  fossil-objects/
  db/
  tmp/
```

Initial databases should stay small and replaceable (for example SQLite/DuckDB for catalogs/experiment state). Do not introduce distributed databases, Kubernetes, Ceph, or multi-master storage during this bootstrap.

## Gravebuster bootstrap acceptance checklist

- [ ] Identify OS, available disk, Python, Git, and current local repository paths.
- [ ] Fetch/pin current `fossil-core`, `fossil-ai-systems`, `cortex-v4`, and SSC revisions.
- [ ] Preserve dirty/uncommitted local work before changing branches.
- [ ] Create `V4_LAB_ROOT` and isolated worktree/experiment directories.
- [ ] Run current deterministic FOSSIL core tests.
- [ ] Run current Cortex V4 contract tests and compare with the last known 80-test CI baseline.
- [ ] Do not invoke the legacy `run_v4_live_litellm.py` unchanged.
- [ ] Inspect current SSC `summon_agent`, LiteLLM config, timeout/retry/fallback ownership, and model receipt behavior.
- [ ] Wire explicit current LiteLLM URL/key config for the local worker without reading personal Desktop `.env` files.
- [ ] Run only no-spend metadata endpoints first.
- [ ] Build/repair the assured research-worker wrapper.
- [ ] Execute one bounded real worker request.
- [ ] Verify requested model, actual model, fallback attempts, timing, input/output identity, and workspace writes are captured.
- [ ] Scan output/log artifacts for credential-shaped data.
- [ ] Preserve a compact FOSSIL/V4 execution receipt.
- [ ] Crash/retry the bounded worker path and verify no false completion.
- [ ] Write the exact observed result back to FOSSIL before starting fork experiments.

## Do not do yet

- Do not download the largest JusticeDAO corpora.
- Do not put raw datasets in GitHub.
- Do not turn a vector or graph database into canonical FOSSIL truth.
- Do not launch many LiteLLM workers before requested/actual model provenance is reliable.
- Do not create the full issue tree until the Gravebuster readiness surface is understood well enough to avoid tracking obsolete assumptions.
- Do not treat upstream test claims as locally reproduced until they have actually run on the worker.

## First local-session objective

A fresh local session on Gravebuster should receive the companion handoff:

`docs/handoffs/2026-08-10-v4-executable-lab-gravebuster-bootstrap.md`

Its terminal condition is **not** “finish Cortex V4.” It is:

> Establish whether Gravebuster can serve as a reproducible, isolated, provenance-complete temporary V4/FOSSIL research worker, and leave a mechanically inspectable receipt of that result.

Once this gate is green, create the executable-lab campaign issue tree and start the independent fork-specialist sessions.
