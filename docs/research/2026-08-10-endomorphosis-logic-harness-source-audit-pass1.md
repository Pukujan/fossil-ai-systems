# Endomorphosis Logic-Harness Source Audit — Pass 1 — 2026-08-10

**State:** `source-inspection / proposed`  
**Execution status:** source and commit diffs inspected; **not yet reproduced locally**.

## Scope

This pass examines concrete implementation changes in `endomorphosis/ipfs_datasets_py`, rather than relying on project README descriptions, for patterns relevant to Cortex V4's cross-vendor hypothesis/assurance methodology.

Pinned commits inspected:

- `1cd374ad09c329d13b1e2f146f0d68da9d2bc7b3` — LFP-041 parser fuzzing/resource gates
- `92d4474df33846002e1fa867bc2884789378a274` — LFP-042 authority-boundary audit
- `333628e1baa85df41a3b9097f6c174b9ac4bbf62` — LFP-043 differential/metamorphic/reconstruction/E2E evidence
- `110fc946474e6936776269d9dcdd4e3ed9f560e6` — LFP-046 bounded objective refill / fixed point

The current `ipfs_datasets_py` head observed during the broader pass was `fc49cbb3e0e96bf07b367859da32123187d706c1`.

## Finding 1 — fuzzing is treated as bounded fail-closed conformance, not just random input generation

LFP-041 adds an explicit `ParserResourcePolicy@1` test surface.

The code states acceptance conditions including:

- parsers terminate within declared bounds;
- exhaustion of input/token/depth/diagnostic/time/memory budgets fails closed;
- resource and syntax failures preserve exact source spans;
- unsupported constructs cannot be silently dropped;
- resource violations expose stable reduced counterexamples.

The tests use finite ceilings and a hard wall-time check. They also contain a deterministic delta-debug-style reducer that repeatedly shrinks a failing source while preserving the same failure code.

### V4 implication

For V4, "fuzzing" should probably mean a family of bounded adversarial generators **plus stable minimized counterexample artifacts**, not merely throwing random prompts/inputs at agents.

Potential V4 requirement:

`failure -> stable failure class -> minimized reproducer -> exact affected artifact/revision -> durable FOSSIL event`

This would make model-generated edge cases useful even when a human later needs to inspect the failure.

## Finding 2 — explicit authority ceilings are a major architectural pattern

LFP-042 implements `LogicAuthorityAudit@1` as a side-effect-free authority evaluator.

The code explicitly states that the audit:

- never installs tools;
- never probes the host;
- never opens the network;
- never upgrades a claim.

Its closed rules include:

- confidence / `is_valid` / similarity do not prove parse correctness;
- generic `ok` / `success` / `passed` does not become theorem/proof authority;
- quota exhaustion, timeout, or unavailability does not become logic evidence;
- only official proof-kernel success with pinned imports and matching environment identity establishes kernel/theorem authority.

The implementation classifies actors into roles such as advisor, candidate, hammer, solver, ATP, model checker, protocol tool, monitor, kernel, and support, and it associates those roles with different authority ceilings.

### V4 implication

This is highly relevant beyond theorem proving.

V4 should consider a typed **assurance authority ceiling** for every evidence producer/verifier.

Candidate vocabulary:

- `advisory`
- `candidate`
- `source_supported`
- `satisfiability`
- `bounded_model_check`
- `finite_trace`
- `protocol_symbolic`
- `kernel_proof`
- `external_observation`
- `human_decision`

The exact vocabulary is not frozen. The invariant is more important:

> A component cannot promote a claim above the maximum authority that its mechanism can actually establish.

A frontier model saying "tests passed" is not the same authority as test-runner output. A test-runner success is not the same authority as a real production observation. An SMT result is not automatically a proof of the user's natural-language requirement.

## Finding 3 — differential disagreement is typed inconclusive, not majority-voted

LFP-043 joins differential, metamorphic, translation-preservation, reconstruction, and end-to-end evidence.

The checked-in conformance cases explicitly include pairs such as:

- Z3 vs CVC5;
- TLC vs Apalache;
- ProVerif vs Tamarin;
- Vampire vs E prover;
- runtime monitor vs shadow monitor.

When paired mechanisms disagree, the examples set the joined verdict to `inconclusive` and explicitly note that disagreement is **never majority-voted**.

The E2E evidence also attaches authority ceilings to different mechanisms:

- kernel;
- exact/satisfiability;
- bounded;
- protocol symbolic;
- finite trace;
- candidate/reconstruction.

### V4 implication

This maps closely to the desired cross-vendor frontier-model behavior.

A useful default join policy is:

- agreement -> record agreement, but retain the minimum/appropriate authority of the underlying evidence;
- disagreement -> `inconclusive` / open a discriminating test;
- partial unavailability -> record availability failure, not epistemic evidence;
- higher-authority external oracle -> may resolve lower-authority model disagreement within its stated scope.

This gives V4 a principled alternative to voting.

## Finding 4 — preservation and metamorphic checks attack translation itself

LFP-043 includes positive and negative preservation fixtures. Negative fixtures attempt silent drops and are expected to fail.

This matters because V4 frequently translates:

`user language -> task contract -> model prompt -> intermediate claim -> test/formal statement -> outcome`

Every translation can lose a condition while still looking plausible.

### V4 implication

V4 should test **semantic preservation across control-plane transformations**, not only the final artifact.

Candidate metamorphic checks include:

- reordering equivalent evidence should not reverse a judge verdict;
- changing temporary agent aliases should not change epistemic state;
- normalizing prose into typed claim IR should preserve required assumptions;
- translating a user acceptance criterion into a formal property should round-trip to an equivalent human-readable constraint;
- narrowing a task must not silently remove a required acceptance criterion;
- a repaired implementation must be rechecked against the original frozen contract, not only the newly generated tests.

## Finding 5 — bounded objective refill is surprisingly close to the V4 hypothesis frontier

LFP-046 implements `LogicGapRefill@1` / `ObjectiveRefillFixedPoint@1`.

The code describes a transaction between owner-scoped evidence gaps and an append-only derived task ledger.

Observed rules include:

- admitted tasks are content-addressed over gap identity, owner paths, source/config/corpus identities, and validation command;
- per-epoch goal/task limits;
- bounds for open tasks, attempts, refinement depth, and cooldown;
- protected control artifacts cannot be rewritten;
- duplicate and broad unscoped tasks are rejected;
- generated tasks/gaps are **evidence only**, never completion or mutation authority;
- two consecutive scans over identical source/config/corpus identities that produce no new admissible tasks constitute a fixed point.

It also defines explicit dispositions such as:

- admitted;
- duplicate;
- depth rejected;
- retry exhausted;
- cooldown;
- unscoped rejected;
- protected rejected;
- authority rejected;
- fixed-point skip.

### V4 implication

This is a concrete production-pattern candidate for controlling hypothesis explosion.

V4 could adapt the same shape:

`EvidenceGap -> CandidateHypothesis/Test -> AdmissionPolicy -> ContentAddressed WorkOrder -> Attempt Ledger -> FixedPoint/Escalation`

Potential stopping rule:

1. freeze source/config/corpus/artifact revision identities;
2. scan unresolved evidence gaps;
3. admit only novel, scoped hypotheses/tests;
4. enforce branch/depth/attempt/cooldown/open-work ceilings;
5. require new evidence or a materially changed hypothesis after a failure;
6. if two consecutive scans produce no new admissible work, declare a **search fixed point**;
7. fixed point means `no_more_admissible_machine_work`, not `truth` or `success`;
8. unresolved high-risk claims then escalate.

This is more precise than an unrestricted "keep summoning agents until they agree" loop.

## Finding 6 — the Endomorphosis stack separates operational success from epistemic authority

Across these commits, a repeated design pattern appears:

- `success=true` is not automatically proof;
- availability failure is not evidence;
- candidate generators cannot self-promote;
- bounded mechanisms only claim bounded authority;
- kernel authority requires the official kernel and pinned environment/import identity;
- generated follow-up tasks do not own completion authority.

### V4 implication

V4 should separate at least:

- **operation status** — did the tool/model call run?
- **artifact validity** — is the returned artifact syntactically/schema valid?
- **evidence authority** — what can this mechanism actually establish?
- **claim state** — proposed/open/supported/disputed/etc.
- **promotion authority** — is policy allowed to change durable truth/production state?

Conflating these is a likely source of agent-harness false confidence.

## Critiques / limits of transferring these patterns

1. Formal-logic tools have much clearer authority semantics than open-ended coding, architecture, research, or product work.
2. A proof kernel can check a formal theorem; there is no equivalent universal kernel for "this is the right architecture."
3. The correctness of natural-language-to-formal translation remains a separate hard problem.
4. The inspected evidence is source/commit-level; these commits have not yet been reproduced in our environment.
5. A large test/conformance suite can itself encode wrong assumptions.
6. Fixed-point detection can stop too early if the gap detector is weak or biased.
7. Content-addressing prevents accidental duplicate work but does not prove two differently worded hypotheses are semantically distinct.
8. Cross-tool differential checks can still share a common faulty translation or input representation.

## Proposed V4 experiments derived from this audit

### Experiment A — authority-ceiling enforcement

Create intentionally misleading receipts:

- model says `passed`;
- test runner timed out;
- SMT solver returns `sat`;
- TLC finds no violation in a bounded state space;
- human reviewer marks approved.

Verify V4 cannot collapse these into one generic `verified=true`.

### Experiment B — differential disagreement

Give two independent verifier mechanisms contradictory results.

Expected:

`disagreement -> inconclusive -> discriminating test/escalation`

not majority vote.

### Experiment C — minimized failure artifacts

Have frontier models generate adversarial operation sequences; when one breaks a V4 invariant, automatically minimize the sequence while preserving the failure.

Store both original and minimized reproducer with provenance.

### Experiment D — hypothesis fixed point

Run a bounded hypothesis/search frontier with:

- max branch width;
- max depth;
- max attempts per mechanism;
- cooldown;
- duplicate/content-address checks;
- novelty/new-evidence requirement.

Test whether two empty rescans correctly stop machine work without incorrectly marking the task successful.

### Experiment E — translation-preservation metamorphics

Mutate:

- author alias;
- order of peer responses;
- prose rendering;
- evidence ordering;
- model/vendor metadata visibility.

The epistemic verdict should remain stable unless the mutated information is legitimately relevant.

## Current adjudication

These Endomorphosis implementation patterns materially strengthen the case that V4 is missing **typed verifier authority, differential/inconclusive joins, metamorphic translation checks, minimized counterexamples, and a bounded fixed-point policy for generated work**.

They do **not** prove that the full Cortex cross-vendor hypothesis-search methodology is correct.

Next evidence should come from:

1. snapshotting the exact relevant Endomorphosis source files/commits into FOSSIL;
2. locally reproducing a bounded subset of their tests/receipts;
3. implementing small V4 analogues behind feature flags;
4. comparing them against simpler controls under the planned A/B/C/D/E benchmark.
