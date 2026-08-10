# Cortex V4 Hypothesis-Search and Assurance Harness — Working Synthesis — 2026-08-10

**State:** `working / proposed`  
**Promotion status:** **not promoted**  
**Purpose:** consolidate the current architecture hypothesis, strongest supporting theories, strongest critiques, and the experiments required to decide what V4 should mechanically enforce.

This document is derived from the 2026-08-10 conversation checkpoint and inspected project code. It is not independent external evidence.

## Working thesis

Cortex V4 should be designed as an **evidence-driven execution and assurance kernel that employs interchangeable models**, not as a multi-agent conversation or voting framework.

Its core search object is a graph/frontier of:

`task -> questions -> hypotheses -> claims -> assumptions -> evidence -> attacks -> tests -> observations -> state transitions`

Models generate and challenge candidates. External sources, executable tests, formal mechanisms, and calibrated human review determine what can be promoted.

## Proposed control-plane decomposition

### 1. Contract layer

Owns:

- user objective and non-goals;
- acceptance criteria;
- ambiguity/questions;
- risk and irreversibility;
- allowed tools/actions;
- evidence requirements;
- verification surfaces;
- budget/deadline;
- human-escalation conditions.

No implementation should silently redefine the contract.

### 2. Evidence/context layer

FOSSIL supplies:

- exact source snapshots;
- citations/spans;
- claim/relation lifecycle;
- disagreement;
- previous failures/KEDB knowledge;
- task-conditioned model/harness observations;
- retrieval context with provenance.

Retrieved context is evidence input, not authority.

### 3. Hypothesis/test frontier

The controller maintains competing theories rather than a single linear plan.

Each frontier item should be able to express:

- hypothesis/claim;
- predictions;
- assumptions;
- dependencies;
- current evidence;
- counterevidence;
- falsifiers;
- consequence if wrong;
- uncertainty/evidence gap;
- estimated experiment cost;
- stopping condition.

Scheduling should prioritize **expected risk reduction / information gain**, not agent seniority or majority preference.

### 4. Worker/attacker lanes

Models are assigned temporary roles such as:

- hypothesis generator;
- independent researcher;
- implementation worker;
- assumption auditor;
- counterexample generator;
- source/citation attacker;
- causal critic;
- runtime reproducer;
- test designer;
- formalization proposer;
- repair worker.

Persistent personas are unnecessary unless empirically useful.

### 5. Blind-review projection

The control plane retains true provenance but reviewers receive the minimum context required.

Default withheld fields:

- vendor/model identity;
- persistent author alias;
- prior peer verdicts;
- popularity/consensus;
- irrelevant provenance.

Default retained fields:

- claim;
- evidence/source references;
- assumptions;
- predictions;
- artifact revision;
- acceptance criteria;
- falsification target.

A typed intermediate representation and deterministic renderer should be preferred where it removes stylistic identity leakage without changing semantics.

### 6. Independent-first review

Attackers commit their first-pass critique before seeing other attacks.

After commit/reveal, a second phase may expose anonymous peer attacks for cross-examination.

This reduces information cascades and creates evidence about whether a conclusion was independently discovered.

### 7. Outcome oracles

Wherever possible, replace model adjudication with an external mechanism:

- tests;
- runtime probes;
- API/database state;
- compilers/typecheckers;
- static analyzers;
- theorem provers/SMT;
- model checkers;
- reproducible calculations;
- source verification;
- human-calibrated domain review.

Agent testimony is not an outcome oracle.

### 8. Evidence accounting / narrow adjudication

Avoid a single "super-judge" choosing the best story.

Instead maintain claim-level evidence accounting. A model adjudicator, when needed, should answer narrow questions such as:

- does this cited span actually support this claim?
- are these two claims contradictory under the frozen definitions?
- does the stated inference follow from the listed evidence?
- did this attack introduce a new falsifier or merely restate an objection?

The final state transition remains a control-plane policy action.

### 9. Formal/stateful assurance of V4 itself

Candidate stack:

- formal state-machine model (e.g. TLA+/TLC or equivalent);
- property/stateful tests against implementation;
- fault injection and fuzzing;
- trace-to-spec replay;
- idempotency and stale-revision checks.

Formal methods verify the control contract, not external truth.

### 10. Promotion/closeout

No `success=true` as the sole terminal state.

At minimum distinguish:

- `verified`
- `supported`
- `disputed`
- `unverified`
- `blocked`
- `failed`
- `escalated`

Closeout should include evidence, attacks, tests, unresolved risk, revisions, traces, and a human-verifiable artifact where required.

## Proposed invariants

The following are architecture hypotheses to formalize/test:

1. No mutating execution without the required task contract/preflight.
2. No claim promotion solely because models agree.
3. No promotion without the evidence class required by task/risk policy.
4. No verifier result can approve an artifact revision it did not inspect.
5. Independent-review status requires first-pass isolation.
6. Provider fallback does not count as epistemic independence.
7. Reviewer-visible author aliases are ephemeral.
8. FOSSIL retains exact model/provider provenance even when peer review is blind.
9. Search/retrieval failure cannot be converted into successful grounding.
10. Every retry has one owner and a bounded stopping condition.
11. Repeated attack without new evidence/falsifier/test does not count as progress.
12. Cancelled/losing workers cannot overwrite later state.
13. Every child task eventually verifies, fails, blocks, or escalates.
14. Closeout cannot succeed with required evidence missing.
15. Model agreement is metadata, not source evidence.
16. Source independence is assessed at the upstream-source level, not merely agent count.
17. Formal proof/model-check result must bind to a versioned statement/contract.
18. High-risk unresolved contradictions force escalation rather than silent synthesis.

## Supporting theory / precedent

### Multiple working hypotheses / strong inference

Strong conceptual support exists for keeping multiple candidate explanations alive and designing observations that discriminate them, rather than selecting an attractive hypothesis and collecting confirming evidence.

V4's contribution would be to make that behavior mechanical and bounded for software/research workflows.

### Independent verification and validation

The IV&V pattern supports separating creator and verifier authority, selecting independent techniques, and exercising adverse conditions.

V4 generalizes this into machine-scale independent lanes but must still prove that "different model" creates useful technical independence.

### Diverse implementations can find different failures

Heterogeneous model portfolios may expose distinct errors, supporting cross-vendor attack lanes.

However this support is conditional because nominally independent implementations/models can share common-mode faults.

### Formal/mechanical verification

The Endomorphosis logic stack demonstrates a useful authority separation: natural language can be parsed/formalized, while actual proof execution is delegated to Z3/CVC5/Lean/Coq with explicit result states and operational controls.

The proof establishes the formal statement, not that the formal statement faithfully represents the user's real-world intent.

## Strongest critiques / rival hypotheses

### Rival H1 — strong single model + tools may beat complex orchestration

A sufficiently capable frontier model with excellent retrieval, tests, and tool use may outperform a multi-agent harness on many tasks once orchestration overhead, latency, and failure modes are included.

**Required adjudication:** matched-budget benchmark against a single-model baseline.

### Rival H2 — diversity gains are mostly selector gains

The apparent benefit of many agents may come from producing more candidates while a strong selector does the real work.

**Risk:** V4 simply relocates the single point of failure into adjudication.

**Required adjudication:** test outcome-oracle/evidence accounting against learned/model selectors.

### Rival H3 — correlated errors dominate

Different vendors can share difficult failure surfaces, training-data patterns, benchmark artifacts, or the same poisoned context.

**Required adjudication:** measure task-conditioned error correlations and upstream evidence overlap.

### Rival H4 — adversarial agents create attack theater

Critics may increase false positives and developer cognitive load without discovering real defects.

**Required adjudication:** measure true defect discovery, false-attack rate, and human triage minutes.

### Rival H5 — hypothesis expansion causes search explosion

The harness may spend large token budgets generating plausible but low-value branches.

**Required adjudication:** compare scheduler policies by severe errors prevented per cost/human-minute.

### Rival H6 — blinding removes useful provenance

Model identity may sometimes be relevant to capability, tool semantics, context limits, known incidents, or provider-specific behavior.

**Required adjudication:** blind by default at the epistemic-review layer while allowing control-plane routing to use identity; benchmark whether review quality changes.

### Rival H7 — identity is still inferable

Style/code/output fingerprints can leak authorship despite hidden IDs.

**Required adjudication:** build a leakage classifier/test and measure post-normalization attribution accuracy.

### Rival H8 — formal methods validate the wrong thing

A theorem prover or model checker can establish a perfectly specified but irrelevant property.

**Required adjudication:** bind formal statement to source contract/citations and require translation review or round-trip/metamorphic checks.

### Rival H9 — FOSSIL itself becomes common-mode context

If all agents retrieve the same incorrect/stale/poisoned graph projection, model diversity does not help.

**Required adjudication:** provenance-aware retrieval attacks, alternative retrieval paths, raw-source probes, lifecycle checks, and poisoned-context tests.

### Rival H10 — automation relocates human cost

The harness can generate so many claims, attacks, receipts, and traces that the specialist spends more time understanding the assurance machinery than solving the task.

**Required adjudication:** human-attention minutes must be a first-class benchmark metric.

## Endomorphosis patterns currently worth borrowing

From source inspection, the most relevant ideas are not "legal AI" as a brand but specific mechanical patterns:

1. **Typed normative extraction before proof** — separate messy-language parsing from formal proof authority.
2. **Multiple proof backends** — Z3/CVC5/Lean/Coq as independent mechanical services where applicable.
3. **Explicit proof statuses** — unsupported/error is different from proved/disproved.
4. **Bounded proof execution** — timeout, validation, rate limiting, caching.
5. **Authority-boundary audits** — advisor/model/solver/proof-kernel roles should not be conflated.
6. **Differential + metamorphic + reconstruction + E2E evidence** — useful template for V4 gate design.
7. **Fuzzing / parser-bomb / Unicode / resource gates** — attack non-semantic implementation surfaces.
8. **Cross-product conformance suites** — test domain × view × family × provider combinations rather than one happy path.
9. **Bounded objective refill / fixed-point notion** — potentially relevant to stopping no-progress hypothesis/repair loops.
10. **Scoped autopatch + targeted validation** — changed scope maps to required tests.
11. **Retry suppression / ownership** — HACC explicitly disables nested model retry in an inspected routing path.
12. **Safe emit-vs-apply separation** — generated repair artifact can be reviewed before mutation.

These are **candidate patterns**, not yet adopted V4 requirements.

## Proposed benchmark campaign

Use representative hard tasks from debugging, architecture, research synthesis, long-running tool workflows, and ambiguous multi-source analysis.

### Arms

**A. Single-frontier baseline**
- one strongest model;
- normal FOSSIL/tool access;
- deterministic tests.

**B. Homogeneous ensemble**
- multiple independent attempts from same model/family;
- selector.

**C. Heterogeneous vote/panel**
- cross-vendor outputs;
- conventional judge/ranking.

**D. V4 hypothesis-search**
- competing hypotheses;
- independent attacks;
- evidence graph;
- discriminating tests;
- bounded repair.

**E. V4 full assurance**
- D plus provenance blinding;
- commit/reveal;
- leakage audit;
- outcome-oracle priority;
- formal/stateful control-plane checks.

### Metrics

Primary:
- severe defects prevented;
- final outcome correctness;
- unsupported-claim rate;
- citation/source correctness;
- human review minutes.

Secondary:
- distinct true failure classes found;
- false-positive attack count;
- test/experiment yield;
- residual uncertainty;
- reproducibility;
- tokens/cost;
- latency;
- number of escalations;
- stale/duplicate/retry violations.

### Critical comparison

The methodology should not be promoted because it sounds epistemically stronger.

It should be promoted only if it beats simpler baselines on a measured objective such as:

**severe errors prevented per unit of human attention under bounded cost and latency.**

## Current adjudication

**Promising architecture hypothesis; not proven production methodology.**

Strongest reasons to continue:

- aligns with FOSSIL's existing evidence/lineage model;
- turns model diversity into targeted failure search rather than consensus;
- allows deterministic mechanisms to own truth where available;
- explicitly treats human attention as a resource;
- has clear rival hypotheses and falsifiable benchmark design;
- formal/stateful methods can verify the orchestration layer itself.

Strongest reasons not to promote yet:

- no matched end-to-end benchmark against strong single-model/simple baselines;
- no measured task-conditioned model error-correlation matrix;
- no implemented information-gain scheduler;
- no implemented provenance-leakage benchmark;
- current V4 mechanical session enforcement has known runtime gaps;
- adjudication and evidence-authority policies are not yet fully mechanical;
- Endomorphosis source patterns have not yet been reproduced locally as executable receipts.

## Immediate next gates

1. Preserve the conversation checkpoint and this synthesis in FOSSIL.
2. Snapshot original external research sources for the highest-load-bearing claims.
3. Continue source/executable audit of Endomorphosis:
   - HACC adversarial runner and validation;
   - `ipfs_datasets_py` proof authority boundaries;
   - fuzzing/metamorphic/conformance suites;
   - bounded objective/fixed-point loop;
   - IPFS/content-addressed reliability where relevant.
4. Write V4 `AssuranceWorkOrder` / claim-hypothesis-test intermediate schema.
5. Design the matched A/B/C/D/E methodology benchmark before adding more orchestration complexity.
6. Fix V4's durable cross-process session state and fail-closed gates.
7. Add formal state-machine and stateful/fault-injection tests for the controller.
8. Only then consider promotion of new methodology invariants.

## Provenance note

This synthesis is a **derived local artifact** from the preserved conversation and inspected repositories. It deliberately retains criticisms and rival hypotheses. Later evidence should change claim states or supersede conclusions through new events rather than rewriting this document.
