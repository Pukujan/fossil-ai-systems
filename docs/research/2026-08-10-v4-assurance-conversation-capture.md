# Cortex V4 Assurance-Harness Conversation Capture — 2026-08-10

**Status:** transcript-derived research checkpoint; **not a complete verbatim transcript**.

This checkpoint preserves the materially recoverable architecture discussion from the current ChatGPT conversation about Cortex V4, FOSSIL, cross-vendor frontier-model orchestration, adversarial verification, formal methods, and related production/research precedents.

Some earlier chat turns were compacted by the conversation runtime before this checkpoint was written. Their exact wording is therefore not reproduced. Where exact wording was not available, this document records only the recoverable substance and marks it as reconstruction rather than inventing a transcript.

## Why this checkpoint exists

The user explicitly asked to preserve the knowledge before continuing deeper research. The conversation itself had already identified chat/UI loss as a failure mode and FOSSIL's architecture already treats immutable evidence separately from derived claims.

This document is therefore an **evidence checkpoint**, not an architecture promotion decision.

## User objective recovered from the conversation

Cortex V4 is intended to become a standalone harness / agent-loop control plane for difficult work performed by many frontier models across vendors.

The intended system is not primarily a chat-room of agents and not a majority-vote ensemble. Its purpose is to spend abundant model inference to reduce scarce developer/specialist attention while requiring enough provenance, citations, falsification, testing, and independent challenge that agents cannot simply guess their way through a complex task.

The desired rough division of responsibility is:

- **V4** — task/control/assurance kernel: contract, decomposition, task routing, hypothesis/search frontier, gates, retries/deadlines, verification, promotion, closeout.
- **FOSSIL** — durable evidence, source snapshots, claims, disagreement, intellectual lineage, historical failure knowledge, and replaceable knowledge projections.
- **LiteLLM / provider layer** — model transport, provider routing, availability fallback, timeout provenance, and route health.
- **Frontier models** — replaceable workers, researchers, counterexample generators, critics, implementers, experiment designers, and narrowly scoped adjudicators.
- **External reality / executable mechanisms** — tests, probes, compilers, theorem provers, model checkers, runtime traces, repository state, APIs, databases, and human review where machine evidence is insufficient.

## Core methodology as recovered

The conversation converged on the following pattern.

1. Freeze a task contract: objective, non-goals, acceptance criteria, ambiguity, risk, evidence requirements, and verification surfaces.
2. Generate **multiple working hypotheses** rather than committing to the first plausible explanation.
3. Retrieve relevant FOSSIL history/evidence and current external facts with provenance.
4. Assign independent model lanes to:
   - expand distinct theories;
   - identify assumptions;
   - search prior art and known failures;
   - construct falsifying tests;
   - attack causal links and claims;
   - generate counterexamples and edge cases;
   - implement or reproduce where appropriate.
5. Convert disagreements into discriminating observations/tests whenever possible rather than extending verbal debate.
6. Keep a bounded hypothesis/test frontier and spend additional inference where another unit of compute is most likely to expose an expensive mistake.
7. Update claim/hypothesis state from evidence, not model agreement.
8. Run independent outcome verification on the real product/system surface.
9. Escalate only irreducible uncertainty, high consequence, subjectivity, or irreversible action to a human.
10. Preserve the full intellectual lineage: rejected theories, attacks, tests, observations, repairs, and reasons state changed.

The central object should therefore be **claims/hypotheses/tests/evidence**, not persistent agent personas.

## Explicit rejection of model voting as truth

The user repeatedly distinguished the intended system from multi-model voting.

The desired behavior is closer to:

`problem -> hypothesis graph -> adversarial expansion -> evidence acquisition -> counterexample search -> executable falsification -> bounded repair -> surviving claims`

rather than:

`many agents -> debate -> vote -> answer`

Model agreement may be stored as metadata. It is not external evidence and must not by itself promote a claim.

## Parallel theories and adversarial attack

The user emphasized that tasks should not be treated as a single path from A to B. The harness should deliberately expand the problem into alternative and supporting theories, including controversial or adversarial possibilities that one or several frontier models may miss.

Agents should be able to attack **individual claims** from other lanes. A useful distinction recovered from the conversation is:

- **objection** — plausible verbal concern;
- **counterhypothesis** — alternative causal explanation;
- **counterexample** — concrete state/input inconsistent with the claim;
- **test** — executable or observable discriminating procedure;
- **observation** — actual measured result;
- **falsification** — observation incompatible with a prediction.

The system should reward the latter categories more strongly than eloquent criticism.

## Model diversity: use complementarity, not vendor count

The conversation identified a key historical/research warning: independent implementations/models can still make correlated mistakes.

Therefore V4 should not encode a rule such as:

`OpenAI + Anthropic + Google = three independent votes`

Instead, FOSSIL should eventually learn task-conditioned failure profiles and pair models because their observed error modes are complementary.

A future scheduling representation might record, per task type:

- probability of catching a specific failure class;
- false-attack / false-positive rate;
- calibration;
- evidence-grounding performance;
- tool-use reliability;
- latency/cost;
- correlation of misses with other models.

The relevant resource is an **effective independent failure-detector portfolio**, not the raw number of models.

## Separate provider fallback from epistemic verification

A provider/model fallback chain and an independent verification panel are different mechanisms.

LiteLLM may answer:

> This route/model failed or is unavailable; what compatible route can complete the call?

V4 should separately answer:

> Which genuinely different reasoning/evidence channels should investigate or attack this task?

A fallback substitution must not automatically count as another independent verifier.

## Provenance blinding / gray-box review

The user proposed hiding the true model/vendor identity from peer reviewers to reduce self-preference, family preference, vendor bias, anchoring, and reputation effects.

The recovered design direction is:

- V4/FOSSIL privately retain exact model/provider/prompt/tool provenance.
- Reviewers receive a minimized **review projection** with temporary/random artifact aliases.
- Persistent aliases are avoided so reputation cannot accumulate accidentally.
- Peer verdicts remain sealed for the first pass.
- Model/vendor identities and prior verdicts are withheld unless required by the task.

However, the conversation also identified **identity leakage**: models may infer authorship from prose style, formatting, terminology, code patterns, or characteristic reasoning.

Proposed mitigations include:

1. Typed epistemic intermediate representation:
   - CLAIM
   - ASSUMPTION
   - EVIDENCE
   - SOURCE_SPAN
   - DEPENDENCY
   - COUNTEREXAMPLE
   - FALSIFICATION_TEST
   - OBSERVATION
   - RISK
   - UNRESOLVED_QUESTION
2. Deterministic rendering of that representation for blind review.
3. Normal formatters for code where useful.
4. A **provenance leakage audit** that attempts to infer author/vendor after normalization.
5. Counterfactual/metamorphic judge checks: reorder equivalent alternatives, change aliases, vary rendering, and ensure verdicts are stable.
6. Commit/reveal review: reviewers commit independent first-pass assessments before seeing peer attacks.

Perfect anonymity is not assumed.

## Human attention is the scarce resource

A recurring user requirement is economic rather than purely technical:

> model calls/token budget can be treated as comparatively abundant; developer and specialist attention is the expensive resource.

Therefore V4 should optimize not only answer quality but **human-attention reduction**.

A useful task-frontier priority concept discussed in the conversation was:

`priority ~ consequence_if_wrong * uncertainty * evidence_gap * historical_failure_likelihood * novelty / estimated_test_cost`

The exact formula is not frozen. The idea is to allocate inference and experiments where they are most likely to prevent an expensive human-visible failure.

Stopping should therefore not mean "agents agree." A better concept is:

`remaining uncertainty * consequence < escalation threshold`

with mandatory human review for risk classes where automation should not decide alone.

## Hypothesis explosion and the selection bottleneck

A major critique preserved from the conversation is that frontier models can cheaply produce hundreds of plausible theories and objections. More hypotheses do not automatically create more knowledge.

The missing central scheduler problem is therefore:

> Which unresolved claim/test should receive the next unit of compute?

The preferred direction is **information-gain / discriminating-test selection** rather than more debate. Given H1..Hn, V4 should seek an observation whose possible outcomes sharply separate those hypotheses.

This is closer to best-first scientific search than literal Dijkstra shortest-path search.

## Formal methods and finite/stateful adversarial exploration

The user asked whether something like TLA+/TLC or finite fuzzing is missing.

The recovered position is:

- use **TLA+/TLC or another model checker** to verify V4's own state-machine/control-plane invariants;
- use **stateful/property-based testing** to generate implementation operation sequences;
- use conventional fuzzing/fault injection for parsers, receipts, APIs, malformed artifacts, concurrency, and protocol boundaries;
- use frontier models to invent unusual scenarios and candidate counterexamples;
- use deterministic or formal mechanisms to judge properties where possible.

Candidate V4 invariants discussed include:

- no execution before required contract/preflight;
- no promotion without required evidence;
- no stale verification result can approve a newer artifact revision;
- retry count never exceeds policy;
- deadline/retry ownership is singular and bounded;
- cancelled/losing workers cannot commit late results over the winner;
- every child eventually verifies, fails, blocks, or escalates;
- duplicate durable events remain idempotent;
- closeout cannot succeed with orphaned children or missing required evidence.

Formal verification cannot prove that the real-world requirement/specification is correct; it only strengthens verification that V4 obeys the specified control contract.

## Current Cortex/FOSSIL repo evidence inspected during the conversation

### Cortex V4

The current V4 durable-control-plane contract says every run should follow a bounded lifecycle:

`contract -> questions -> competing research -> plan freeze -> implementation -> adversarial verification -> shadow/pilot -> human review -> promotion or escalation -> closeout`

It says model agreement alone is not validation, external facts require source-ranked preflight, retries/deadlines must have one owner, actual product surfaces must be verified, and closeout must be evidence-gated.

Earlier implementation inspection found important gaps that remain separate from this conceptual research:

- OpenCode plugin calls currently launch fresh Python controller processes while controller session state is in-memory, creating a cross-process state-loss bug.
- "Enforce" mode in the inspected hook logs denial rather than reliably blocking the action.
- corpus-search exceptions can be transformed into an apparently successful `searched=True` state.
- closeout can report success without sufficiently strong independent evidence.
- tests do not yet cover the real cross-process plugin sequence.
- GitHub CI/status checks were not running on the inspected V4 master.

These are implementation findings, not objections to the intended methodology.

### FOSSIL

FOSSIL's frozen architecture aligns strongly with the desired harness:

- immutable original evidence;
- append-only validated knowledge-changing events;
- stable corpus-owned identities;
- provenance and exact source references;
- disagreement and supersession as durable state;
- Graphiti/Neo4j as rebuildable projection, not deepest truth;
- proposal-before-commit;
- deterministic policy/risk gate owns truth-changing commits;
- model agreement is not evidence;
- explicit knowledge-pack boundaries;
- model/retrieval/verification services are replaceable adapters.

This conversation should therefore be ingested as source evidence and working hypotheses, not silently treated as promoted canonical architecture.

### LiteLLM / CKFF operations

The inspected operations contract separates provider, LiteLLM/proxy, client, and other deadline layers and says failures should report timeout provenance. This supports keeping provider availability/retry/fallback as a transport/runtime concern separate from epistemic verification.

## Research/theory brought into the discussion

The following were discussed as relevant external precedents or critiques. At capture time, these are **candidate external sources/research claims from the conversation** and should be snapshot-captured separately before being used to promote FOSSIL claims.

### Scientific / epistemic methodology

- T. C. Chamberlin — Method of Multiple Working Hypotheses.
- John R. Platt — Strong Inference.
- Popper-style falsification is conceptually related but does not by itself specify the orchestration system.
- information-gain / experimental-design approaches for selecting discriminating observations.

### Multi-agent / heterogeneous-model evidence

Discussed research themes included:

- heterogeneous model teams can provide complementary information compared with many homogeneous copies;
- multi-agent debate does not reliably outperform simpler baselines across all tasks;
- large models across vendors can still have correlated errors;
- selector/adjudicator quality can become a bottleneck;
- LLM judges exhibit self-preference, family preference, position/verbosity/bandwagon and other evaluation biases;
- model outputs can carry identifiable authorship/model-family fingerprints;
- panels of LLM judges may help in some evaluations but should not become external truth.

### Reliability / verification precedents

- N-version programming and the historical finding that nominally independent implementations can fail on the same difficult inputs.
- Independent Verification & Validation (IV&V): technical independence and adverse-condition testing.
- TLA+/TLC state-machine/model checking.
- stateful/property-based testing and conventional fuzzing.
- multi-agent-system failure taxonomies emphasizing orchestration, inter-agent alignment, and verification/termination failures.

These themes support the direction but also warn that diversity, debate, and judging are not sufficient by themselves.

## Endomorphosis OSS architecture — source-level findings captured so far

The user specifically requested deeper examination of Endomorphosis's OSS work because of its legal/reliability/formal-method orientation.

### `endomorphosis/HACC`

Source-level inspection found a real adversarial evidence workflow rather than only a README concept:

`collection -> corpus -> lexical/vector/hybrid/legal discovery -> grounding bundle -> evidence upload/support evaluation -> adversarial sessions -> scoring/ranking -> optional complaint synthesis/autopatch`

In `hacc_adversarial_runner.py`, observed implementation patterns include:

- explicit provider/model routing;
- Codex-centric defaults in the inspected path;
- provider mapping for several backends;
- scoped autopatch profiles down to target files/symbols;
- bounded timeout defaults by optimization scope;
- explicit `disable_model_retry=True` in the LLM router path;
- a narrowly defined Codex fallback for likely throttling;
- bounded repair attempts;
- targeted validation test selection based on changed files;
- safe/default distinction between emitting a patch and automatically applying it.

Relevance to V4:

- bounded repair and targeted validation are directly useful patterns;
- scope-to-test mapping is a useful mechanical gate;
- retry suppression/ownership is a strong pattern;
- the current HACC path should not be mistaken for a cross-vendor epistemic-independence implementation merely because several providers are addressable.

Pinned repository state observed during this capture:
`endomorphosis/HACC@6f3f42b4a4d2d7e6c021abe6eb055c8705d249f7`

### `endomorphosis/Clarity_Act_Deontic_Logic`

Source inspection of `src/clarity_act_parser.py` found:

- legal text/summary parsing;
- normative-clause identification;
- typed extraction into obligation, permission, and prohibition;
- agent/action/condition/temporal fields;
- translation path toward Lean, Coq, and SMT-LIB;
- a fallback mock implementation when the real `ipfs_datasets_py` logic integration cannot be loaded.

This is an important caution: generated theorem-prover-looking output and a successful real proof execution are different evidence states.

### `endomorphosis/ipfs_datasets_py` logic stack

The current repository history itself is notable. Recent commits include explicit work on:

- a cross-product conformance suite;
- fuzzing, parser-bomb, Unicode, and performance/resource gates;
- advisor/solver/Hammer/proof-kernel authority boundaries;
- differential, metamorphic, reconstruction, and end-to-end evidence;
- bounded objective refill to a fixed point;
- a sealed logic-parser release receipt.

Pinned repository state observed:
`endomorphosis/ipfs_datasets_py@fc49cbb3e0e96bf07b367859da32123187d706c1`

The inspected `ProofExecutionEngine` is not just a formatter. It detects and can invoke Z3, CVC5, Lean 4, and Coq; it includes timeouts, input validation, rate limiting, proof caching, explicit unsupported/error results, and translator boundaries.

This suggests a potentially important V4 design pattern:

> where a task produces constraints suitable for formalization, models can propose/translate the constraint, but a separate proof engine/proof kernel should own the mechanical proof result.

The validity of the translation from messy real-world language into the formal statement remains a separate verification problem.

## Critiques / failure hypotheses that must remain alive

The conversation identified at least the following reasons the V4 methodology could fail even with many frontier models:

1. **Hypothesis explosion** — too many plausible branches consume compute without adding discriminatory evidence.
2. **Branch starvation / scheduler bias** — the controller may prematurely underfund the correct minority hypothesis.
3. **Selection bottleneck** — a final selector can become the same single-point-of-failure problem moved downstream.
4. **Correlated errors** — different vendors may share training-data, benchmark, or conceptual blind spots.
5. **Attack theater** — critics generate eloquent objections without executable counterexamples or evidence.
6. **Judge bias** — evaluator preferences depend on position, style, verbosity, family, prior verdicts, or framing.
7. **Identity leakage** — blind reviewers infer model/vendor from output fingerprints.
8. **Evidence poisoning / authority laundering** — multiple agents can repeatedly cite the same bad upstream source and appear independent.
9. **Retrieval common mode** — all models receive the same faulty context or stale FOSSIL projection.
10. **Benchmark gaming** — scheduler/model roles optimize known harness tests while missing unmodeled failures.
11. **Formal-specification error** — a model checker proves the wrong specification perfectly.
12. **Translation error** — natural-language requirements are mistranslated into formal logic.
13. **Repair loops changing the question** — fixes can satisfy tests while silently violating the original user contract.
14. **Stale verification** — a verifier approves revision N while revision N+1 is promoted.
15. **Hidden retry/timeout stacking** — apparent autonomous work becomes unbounded repeated work.
16. **Human-attention displacement** — a complex harness can create more artifacts/uncertainty for the developer rather than reducing attention.

None of these critiques invalidates the core hypothesis; they define the next assurance targets.

## Proposed next benchmark: test the methodology against alternatives

Before promoting the full V4 methodology, compare matched task sets under several arms:

- **A — single best model** with normal tools/retrieval;
- **B — homogeneous multi-agent** with independent attempts;
- **C — heterogeneous cross-vendor vote/judge**;
- **D — V4 hypothesis/attack pipeline** with evidence graph and executable tests;
- **E — V4 + provenance blinding + commit/reveal + formal/stateful control-plane checks**.

Control or report:

- total tokens/model calls;
- model/provider mix;
- wall-clock latency;
- external tool/test budget;
- human review minutes;
- number and severity of true defects found;
- false-positive attacks;
- unsupported claims;
- citation/source correctness;
- reproducibility;
- branch coverage / distinct failure classes;
- residual uncertainty;
- outcome correctness on the real product surface.

The most important product-level metric is not "how many agents agreed." It is closer to:

> severe errors prevented per unit of human attention, under bounded cost/latency.

## Open research questions

1. What scheduler policy best allocates compute to the hypothesis/test frontier?
2. How should expected information gain be estimated when the hypotheses themselves are model-generated?
3. How can V4 measure error complementarity without overfitting to its own benchmark history?
4. Which task types benefit from multi-model attack versus a strong single-model + deterministic tool path?
5. What minimum evidence is required before a claim moves from proposed/open to supported?
6. How should source independence be computed when different agents cite the same upstream evidence?
7. How should blind-review leakage be measured and what score is sufficient?
8. Which judge tasks can be safely automated, and how are judges calibrated against humans/outcome oracles?
9. How should formal proof results be linked to the exact natural-language contract they purport to verify?
10. How should TLA+/model-checker traces be reconciled with real OTel execution traces?
11. How should V4 distinguish no-progress retries from genuinely new hypotheses?
12. How can the harness guarantee that it reduces, rather than merely relocates, human attention cost?

## Preservation rule

This checkpoint should remain immutable once ingested. Future research should add:

- original external source snapshots;
- exact code snapshots from Endomorphosis and other comparable systems;
- executable reproduction receipts;
- new claims/attacks;
- state transitions;
- supersession links;

rather than rewriting this checkpoint to make later conclusions appear inevitable.
