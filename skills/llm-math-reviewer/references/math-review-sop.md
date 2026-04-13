# CODEX Math Review SOP

## Table of Contents

- [0. Purpose](#0-purpose)
- [1. Operator Summary](#1-operator-summary)
- [2. Non-Negotiable Rules](#2-non-negotiable-rules)
- [3. Inputs and Repository Layout](#3-inputs-and-repository-layout)
- [4. Script and Dependency Policy](#4-script-and-dependency-policy)
- [5. Auditability and Preservation](#5-auditability-and-preservation)
- [6. Closure Discipline](#6-closure-discipline)
- [7. Standard Labels](#7-standard-labels)
- [8. Stage Overview](#8-stage-overview)
- [9. Detailed Stages](#9-detailed-stages)
- [10. Tool Routing Policy](#10-tool-routing-policy)
- [11. Failure Handling](#11-failure-handling)
- [12. Human Escalation Triggers](#12-human-escalation-triggers)
- [13. Final Markdown Template](#13-final-markdown-template)
- [14. Recommended Execution Order](#14-recommended-execution-order)
- [15. Recommended First-Version Scope](#15-recommended-first-version-scope)
- [16. Reviewer Philosophy](#16-reviewer-philosophy)

## 0. Purpose

This SOP defines a strict, evidence-driven review workflow for mathematical papers.

The goal is not to produce a literary review or plausibility summary. The goal is to push each core claim as far as possible toward one justified terminal outcome:

1. `verified_correct_in_scope`
2. `contradicted_by_counterexample`
3. `proof_failure_established`
4. `verification_incomplete`

The agent must produce:

1. a structured claim inventory
2. a dependency graph
3. a risk report
4. an obligation ledger
5. executable or reproducible checks
6. a claim-by-claim verification ledger
7. a final human-readable report
8. a machine-readable final artifact

This SOP is written for Codex or a similar coding agent operating inside a repository.

---

## 1. Operator Summary

Read this section first. It is the shortest valid mental model of the SOP.

### 1.1 What the agent is trying to do

For every core claim:

1. identify the exact statement
2. identify what it depends on
3. find the proof bottlenecks
4. try direct verification
5. try adversarial pressure-testing when appropriate
6. keep splitting unresolved steps into smaller obligations
7. stop only when the claim has a terminal status or a precise blocker

### 1.2 What does not count as completion

The following are not completion criteria:

- "reviewed textually"
- "looks plausible"
- "standard argument"
- "no contradiction found"
- "too long"
- "too technical"
- "probably correct"

These are intermediate observations only.

### 1.3 When the review is allowed to end

The review may end only when both conditions hold:

1. every core claim has a terminal status
2. every unresolved core claim has an explicit blocking reason recorded in the obligation ledger

If even one core claim remains in a vague nonterminal state, the review is not finished.

---

## 2. Non-Negotiable Rules

The agent MUST follow these rules at all times.

1. Do not skip stages.
2. Do not confuse fluent prose with proof.
3. Every finding must be tied to exact source location, claim ID, evidence, and confidence.
4. Every core claim must finish with one terminal status:
   - `verified`
   - `contradicted`
   - `proof_failure_established`
   - `verification_incomplete`
5. A positive overall verdict is allowed only if all core claims are fully verified within an explicit scope.
6. A negative overall verdict is allowed only if a verified counterexample, explicit contradiction, or precise proof failure is established.
7. If neither threshold is met, the overall verdict MUST be `verification_incomplete`.
8. Absence of detected error is not evidence of correctness.
9. Partial tool support is not enough for a positive verdict.
10. Citation plausibility is not enough for a positive verdict.
11. `reviewed_textually`, `looks_plausible`, `standard`, and similar labels are never terminal statuses for core claims.
12. If a step cannot be checked, record why exactly:
   - parsing limitation
   - missing definitions
   - unsupported domain
   - missing citation source
   - formalization gap
   - computational infeasibility
   - environment or toolchain blocker
13. If a tool fails, do not automatically interpret that as a mathematical error.
14. Never silently discard ambiguity.
15. If a proof step is too compressed, split it into smaller sub-obligations.
16. The review must maintain a live unresolved-obligation list for the core proof chain.
17. The review may not end while unresolved core obligations remain unless each one is explicitly recorded as the reason for `verification_incomplete`.
18. For every core claim, attempt at least one direct verification avenue.
19. For every universal or easily falsifiable claim, attempt at least one adversarial avenue when feasible:
   - counterexample search
   - boundary-case probing
   - witness generation
   - small-model search
20. Never claim `verified` on the basis of charitable reconstruction alone.
21. Preserve enough artifacts so that another human or agent can understand and rerun the decisive parts of the review.
22. In the default mode, do not search for errata, corrigenda, commentary, blog posts, or other later discussion of the paper.
23. In the default mode, do not obtain or use information published after the paper. If such information is retrieved accidentally, ignore it and continue using only admissible contemporaneous sources.
24. If and only if the user explicitly asks for comparison against errata, revised versions, or later author clarifications, run that work in a separate reconciliation layer and keep the original paper-as-written verdict unchanged.
25. Later materials may refine how findings are classified, but they do not retroactively change what the original text said.
26. Treat a review target as a review target, not as a defect, unless terminal evidence is established.
27. Negative language stronger than `unresolved`, `blocked`, or `merits further scrutiny` requires one of:
   - a verified counterexample
   - an explicit contradiction
   - a precise proof failure tied to an exact source span
28. Tool failure, parser failure, formalization failure, or incomplete verification paths are workflow limitations unless they themselves establish a mathematical contradiction.
29. Charitable reconstruction alone cannot justify `verified` or `proof_failure_established`.
30. Sampled computation, heuristic search, or finite-instance testing may support suspicion, but they cannot justify exhaustive conclusions unless scope and exhaustiveness are independently established.
31. Do not strengthen conclusions when moving from the structured ledger to the final prose report.
32. Distinguish step-level findings, claim-level statuses, and the paper-level verdict; do not promote a local unresolved step into a global defect claim without the required evidence.
33. Perform blocker triage early: separate cheap local checks from deep dependency-driven blockers before attempting long proof reconstructions.
34. Use executable checks early for local algebra, parameter consistency, index consistency, and similar low-cost invariants when feasible.
35. Audit cited results for statement match as early as possible; verify that the cited statement actually supplies the hypotheses, scope, and parameter regime the proof uses.
36. Distinguish typo or transcription defects from substantive proof blockers, and record both when relevant without conflating them.
37. For each unresolved obligation, record whether it is local to the current argument or inherited through an external dependency chain.
38. Every active blocker must carry a highest-value next action; vague notes such as "continue verification" are not sufficient ledger entries.

---

## 3. Inputs and Repository Layout

### 3.1 Expected inputs

- `paper/main.tex` or equivalent LaTeX source
- bibliography files if present
- appendices if present
- macro files if present
- `review_config.yaml`

### 3.2 Optional inputs

- domain classification
- preferred proof assistant
- trusted citation sources
- custom parser rules
- reviewer-defined priority claims

### 3.3 Recommended layout

```text
.
├── paper/
│   ├── main.tex
│   ├── sections/
│   ├── appendix/
│   └── refs.bib
├── review/
│   ├── scripts/
│   ├── prompts/
│   └── papers/
│       ├── <paper_slug_a>/
│       │   ├── outputs/
│       │   ├── temp/
│       │   └── bak/
│       └── <paper_slug_b>/
│           ├── outputs/
│           ├── temp/
│           └── bak/
├── tools/
└── review_config.yaml
```

### 3.4 Repository policy

- `review/scripts/` stores reusable workflow code.
- Every reviewed paper MUST get its own review root at `review/papers/<paper_slug>/`.
- Derive `<paper_slug>` from the reviewed paper filename stem or source directory name. Normalize it into a filesystem-safe slug; if that slug is ambiguous, append a stable disambiguator such as the year or a short hash.
- `review/papers/<paper_slug>/bak/` stores paper-specific or experimental helpers.
- `review/papers/<paper_slug>/outputs/` and `review/papers/<paper_slug>/temp/` are paper-local and must never be shared across different papers.
- If custom logic is specific to one paper, keep only the reusable portion in `review/scripts/`.

---

## 4. Script and Dependency Policy

### 4.1 Script creation policy

The agent MAY and SHOULD create scripts when needed.

Rules:

1. If required scripts are missing or mismatched to the paper, create or adapt them before continuing.
2. Prefer small reusable scripts in `review/scripts/`.
3. Put paper-specific checks in `review/papers/<paper_slug>/bak/` unless generalized.
4. Do not block on the absence of a pre-existing tool if a lightweight script can be written safely.
5. Every generated script should state:
   - purpose
   - expected inputs
   - produced outputs
   - whether it is generic or paper-specific

### 4.2 Dependency installation policy

Missing packages are not an excuse to skip a high-value check.

Rules:

1. First check whether a dependency is already installed.
2. If a verification path requires a missing dependency, attempt installation when reasonably possible.
3. Use the smallest reasonable dependency footprint.
4. Record for every installation attempt:
   - tool or package name
   - install method
   - version if available
   - whether installation succeeded
   - whether it was later used
5. If installation fails, record the failure as a toolchain limitation, not a mathematical error.
6. If installation is blocked by permissions, networking, platform limits, or missing system packages, record that explicitly.
7. The final outputs must distinguish:
   - not attempted
   - attempted but blocked
   - installed and used

Examples of potentially relevant dependencies:

- `sympy`
- `sage`
- `numpy`
- `scipy`
- `z3-solver`
- Lean
- Coq
- Isabelle
- HOL Light

---

## 5. Auditability and Preservation

The review itself must be auditable.

For every decisive or high-value check, preserve:

1. claim ID or target ID
2. source file and source span
3. command, script, or backend invocation
4. dependency information
5. raw or minimally processed output
6. reviewer interpretation
7. limitations or blockers
8. path to generated artifacts needed for rerunning

Forbidden behavior:

- reporting a key result without preserving the command or script
- reporting a formalization outcome without preserving the formal file
- reporting a computational contradiction without preserving the witness
- relying on undocumented manual inspection for a decisive verdict
- ending because the paper is "too long" or "too technical" without an explicit unresolved-obligation ledger

---

## 6. Closure Discipline

This SOP is a closure loop, not a single forward pass.

### 6.1 Core rule

After the first pass through parsing and initial checking, the agent MUST revisit unresolved core claims until each one is either:

1. terminally resolved, or
2. blocked by a precise, documented leaf obligation

### 6.2 Required closure loop

1. Build an `open_obligations` list for the core proof chain.
2. For each open obligation, record:
   - obligation ID
   - linked claim ID
   - source span
   - why it is still open
   - next best verification action
   - attempts already made
   - whether it blocks a terminal verdict
3. Re-run target selection using the obligation list, not just the initial risk scan.
4. After each major checking stage, update the ledger:
   - close discharged obligations
   - split large obligations
   - add new leaf obligations if a check isolates a smaller blocker
5. Before writing the final verdict, audit the obligation ledger one last time.

### 6.3 Minimum closure standard

A core claim is sufficiently localized only when:

- the remaining uncertainty is stated as explicit leaf obligations
- each leaf obligation has at least one attempted verification action or one explicit blocked attempt
- the blocking reason is concrete rather than generic

Good blocker:

- "PRF-017-S09 uses a uniformity bound not stated in Lemma LEM-011; tried symbolic and citation validation; no supporting source found"

Bad blocker:

- "Section 7 is hard"

---

## 7. Standard Labels

Use only these labels unless the output schema requires a more specific subtype.

### 7.1 Claim statuses

- `not_reviewed`
- `reviewed_textually`
- `computationally_supported`
- `partially_formalized`
- `formally_checked`
- `verified`
- `suspicious`
- `notation_inconsistent`
- `proof_failure_established`
- `contradicted_by_counterexample`
- `tool_inconclusive`
- `verification_incomplete`

### 7.2 Terminal claim statuses

Only these count as terminal for core claims:

- `verified`
- `contradicted_by_counterexample`
- `proof_failure_established`
- `verification_incomplete`

### 7.3 Proof-step statuses

- `clear`
- `compressed`
- `ambiguous`
- `missing_justification`
- `computationally_checked`
- `formally_checked`
- `failed_formalization`
- `high_risk`

### 7.4 Tool result statuses

- `passed`
- `failed`
- `timeout`
- `unsupported`
- `partial`

### 7.5 Confidence labels

Confidence is about the quality of the review, not the truth of the theorem.

- `low`
- `medium`
- `high`

Interpretation:

- `low`: major parts of the core proof chain remain unresolved
- `medium`: substantial coverage, but at least one nontrivial unresolved gap remains
- `high`: the core claim set is fully resolved within the stated scope

Do not output numerical truth probabilities unless explicitly requested.

---

## 8. Stage Overview

Use this as the quick execution checklist.

Path convention: every `outputs/...`, `temp/...`, or `bak/...` path below is relative to the current paper-specific review root `review/papers/<paper_slug>/`, not to a shared top-level review directory.

| Stage | Name | Core question | Required output |
| --- | --- | --- | --- |
| A | Environment and intake | What paper are we reviewing? | `outputs/intake_summary.json`, `outputs/normalized_text.txt` |
| B | Structural parsing | What are the mathematical objects and theorem-like units? | `outputs/parsed_structure.json` |
| C | Claim extraction | What exact claims must be checked? | `outputs/claims.json` |
| D | Dependency graph | What depends on what? | `outputs/dependency_graph.json` |
| E | Risk scan | Where are the likely bottlenecks? | `outputs/risk_report.json` |
| F | Domain classification | What checking strategy fits this paper? | `outputs/domain_profile.json` |
| G | Target selection | What should be checked first? | `outputs/check_targets.json` |
| G.5 | Obligation ledger | What remains open, and why? | `outputs/obligation_ledger.json` |
| H | Symbolic and computational checks | What can be executed directly? | `outputs/tool_checks_symbolic.json` |
| I | Counterexample search | Can we break universal claims? | `outputs/counterexample_search.json` |
| J | Formalization bridge | What can be checked in a prover? | `outputs/formalization_attempts.json` |
| K | Citation validation | Are prior results used correctly? | `outputs/citation_validation.json` |
| L | Human-facing synthesis | What is the evidence-backed narrative? | `outputs/final_report.md` |
| M | Machine-readable artifact | What is the strict downstream summary? | `outputs/final_report.json` |
| N | Verdict closure | Does the verdict actually follow from the ledger? | `outputs/verdict_closure.json` |
| O | Optional post-publication reconciliation | How do later corrections compare with the paper-as-written findings? | `outputs/post_publication_reconciliation.json`, `outputs/post_publication_reconciliation.md` |

---

## 9. Detailed Stages

## Stage A. Environment and Intake

### Objective

Establish the paper context and prepare files for downstream analysis.

### Required actions

1. Locate the main source file.
2. Expand LaTeX includes if needed.
3. Resolve macros where possible.
4. Create the paper-specific review root `review/papers/<paper_slug>/` before writing artifacts.
5. Build normalized plaintext.
6. Record metadata:
   - title
   - authors
   - date
   - subject area
   - theorem environments used

### Outputs

- `outputs/intake_summary.json`
- `outputs/normalized_text.txt`

### Failure policy

- If LaTeX parsing fails, fall back to PDF or text extraction.
- Record degraded mode explicitly.

---

## Stage B. Structural Parsing

### Objective

Identify the mathematical scaffold of the paper.

### Extract at minimum

- definition
- notation
- assumption
- lemma
- proposition
- theorem
- corollary
- claim
- proof
- remark
- example

### For each extracted object record

- unique ID
- type
- title if present
- source span
- raw text
- normalized text
- local section context

### ID examples

- `DEF-001`
- `ASM-003`
- `LEM-012`
- `THM-002`
- `PRF-012`

### Output

- `outputs/parsed_structure.json`

### Rules

- Every theorem-like statement must be linked to its proof if present.
- If proof is omitted, mark `proof_status = omitted`.
- If proof is indirect, mark `proof_status = indirect_reference`.

---

## Stage C. Claim Extraction and Canonicalization

### Objective

Turn prose mathematics into checkable claims.

### For each theorem-like item

1. extract the formal statement in normalized prose
2. separate hypotheses, objects, and conclusion
3. create a claim record

### Also extract prose claims that materially affect correctness

- prose conclusions immediately after proofs
- displayed formulas asserting exact values
- "therefore", "hence", "we conclude" style standalone conclusions
- exact computations, asymptotics, extremal consequences, or application claims

### Minimum claim schema

```json
{
  "claim_id": "THM-002",
  "claim_type": "theorem",
  "source_location": {
    "file": "paper/main.tex",
    "start_line": 120,
    "end_line": 148
  },
  "dependencies_textual": [],
  "objects": [],
  "hypotheses": [],
  "conclusion": "",
  "domain_tags": [],
  "proof_id": "PRF-002",
  "check_priority": "unknown",
  "is_core_claim": true,
  "terminal_status": "unresolved",
  "blocking_obligations": []
}
```

### Rules

- Split conjunctions into subclaims when that improves verifiability.
- Preserve quantifiers carefully.
- Do not silently strengthen or weaken a statement.
- If a theorem is specialized later in prose to an exact consequence, extract that consequence as its own claim.

### Output

- `outputs/claims.json`

---

## Stage D. Dependency Graph Construction

### Objective

Determine what each major result depends on.

### Required actions

1. identify explicit references to earlier results
2. infer likely implicit dependencies
3. build a directed graph

### Node and edge meaning

- node: claim or definition
- edge: dependency

### Required edge labels

- `explicit_citation`
- `implicit_usage`
- `notation_dependency`
- `hidden_assumption_candidate`

### Rules

- Distinguish formal dependency from expository mention.
- Mark cycles for review.
- Mark later-theorem dependence as suspicious unless clearly deferred.
- Separate upstream theorem dependencies from downstream application dependencies.
- Do not propagate failure upstream without an explicit dependency edge.

### Outputs

- `outputs/dependency_graph.json`
- optional `outputs/dependency_graph.mmd`

---

## Stage E. Risk Scan

### Objective

Identify likely proof bottlenecks before expensive checks.

### Scan for

1. undefined symbols
2. assumption drift
3. notation overload
4. missing quantifier scope
5. unproved transitions
6. circular reasoning
7. hidden case splits
8. vague appeals to standard results
9. compression phrases such as:
   - clearly
   - obviously
   - similarly
   - one checks
   - routine
   - omitted
10. existential constructions with no witness
11. missing dimension or finiteness assumptions
12. ambient category or topology drift
13. equality versus isomorphism conflation
14. pointwise versus uniform versus almost-everywhere ambiguity
15. asymptotic notation without quantified regime
16. codomain or domain mismatch
17. scale or rescaling interface mismatch
18. recursive or telescoping index reversal risk
19. simultaneous refinement or dyadic pigeonholing with multiple preserved invariants
20. every-scale witness claims with no valid witness family

### Each risk finding must include

- `risk_id`
- `linked_claim_id`
- `source_span`
- `risk_type`
- `explanation`
- `severity`
- `confidence`
- `recommended_next_action`

### Severity scale

- `low`
- `medium`
- `high`
- `critical`

### Output

- `outputs/risk_report.json`

---

## Stage F. Domain Classification

### Objective

Choose a fitting checking strategy.

### Domain tags

- combinatorics
- graph theory
- elementary number theory
- algebra
- representation theory
- algebraic geometry
- topology
- PDE
- functional analysis
- probability
- logic
- category theory
- mixed
- unknown

### Checking modes

- discrete-heavy:
  - enumeration
  - counterexample search
  - symbolic simplification
- algebra-heavy:
  - symbolic algebra
  - identity checking
  - selected formalization
- analysis-heavy:
  - assumption consistency
  - estimate-chain auditing
  - local formalization only
- geometry-heavy:
  - definition coherence
  - object consistency
  - local lemma checks

### Output

- `outputs/domain_profile.json`

---

## Stage G. Target Selection

### Objective

Choose what to check first under a completeness-oriented budget.

### Priority order

1. core theorem bottlenecks
2. high-severity risk findings
3. lemmas with many downstream dependencies
4. computational subarguments
5. claims with possible small counterexamples
6. claims with omitted proof details
7. all remaining unverified steps in the main proof chain

### Selection heuristics

- Prefer steps reused by many downstream claims.
- Prefer transitions changing scale, normalization, and referenced lemmas at once.
- Prefer witness-producing "for every scale" or "for every parameter" steps.
- Deprioritize already-classified notation-only defects unless they still block the core chain.

### For each target record

- `target_id`
- `linked_claim_id`
- `target_type`
- `reason_for_selection`
- `intended_tool`
- `expected_check_type`

### Output

- `outputs/check_targets.json`

---

## Stage G.5. Obligation Ledger and Closure Planning

### Objective

Turn unresolved proof burden into an explicit worklist.

### For every core claim, create obligation entries for

- the claim statement itself
- each unresolved bottleneck on its dependency chain
- each critical external result whose applicability is still unresolved

### Required fields

- `obligation_id`
- `linked_claim_id`
- `source_span`
- `obligation_type`
- `parent_obligation_id`
- `blocking_reason_current`
- `next_verification_action`
- `attempts_made`
- `status`
- `is_leaf`
- `blocks_terminal_verdict`

### Rules

- If an obligation says "too large", split it.
- If an obligation says "depends on prior result", link the upstream obligation explicitly.
- If an obligation remains open after a check, update the blocker with new detail.
- A core claim may be terminal only when all blocking leaf obligations are either resolved or explicitly cited as the reason for `verification_incomplete`.
- Regenerate the ledger after Stages H, I, J, and K.

### Output

- `outputs/obligation_ledger.json`

---

## Stage H. Symbolic and Computational Checks

### Objective

Use executable tools whenever the mathematics admits direct checking.

### Typical tools

- SymPy
- SageMath
- NumPy or SciPy
- custom Python
- SAT or SMT solvers
- combinatorial enumerators
- notation consistency checkers
- textual reconstruction audits

### Use this stage for

- algebraic identities
- finite case validation
- recurrence checks
- determinant, rank, or eigenvalue checks
- modular arithmetic examples
- witness generation
- local consistency checks
- suppressed proof-skeleton reconstruction

### Rules

1. Computation cannot prove a universal theorem unless the problem is genuinely finite.
2. Negative computational evidence is strong and must be preserved.
3. Positive sample checks support plausibility but do not prove general correctness.
4. If a universal claim reduces to a finite family, the reduction itself must be verified.
5. A textual reconstruction audit may support `verification_incomplete`, but not `verified`, unless every hidden step has been independently discharged.
6. If a computational check only probes samples, the linked obligation stays open unless exhaustiveness is separately established.
7. If a needed dependency is missing, attempt installation before downgrading the check to unavailable.

### For each tool run record

- `tool_name`
- `command_or_script`
- `input`
- `output`
- `interpretation`
- `limitations`
- `dependencies_used`
- `dependencies_installed`
- `installation_attempts`

### Output

- `outputs/tool_checks_symbolic.json`

---

## Stage I. Counterexample Search

### Objective

Pressure-test universal claims.

### Search targets

- universal statements
- uniqueness statements
- monotonicity claims
- minimality claims
- closure claims
- edge-case parameter claims

### Search methods

- brute force
- random search
- constrained search
- small-model search
- adversarial parameter selection

### If a candidate is found

1. save the object
2. verify it independently
3. attach it to the claim
4. immediately downgrade dependent claims as justified by the dependency graph

### Status labels

- `none_found`
- `candidate_found`
- `verified_counterexample`
- `search_inconclusive`

### Output

- `outputs/counterexample_search.json`

---

## Stage J. Formalization Bridge

### Objective

Translate high-value proof fragments into proof-assistant-friendly form and actually run the checker when feasible.

### Supported backends

- Lean
- Coq
- Isabelle
- HOL Light
- optional DSL intermediary

### Procedure

1. Start with the highest-value fragments in the main theorem chain.
2. Formalize definitions, assumptions, and a target lemma.
3. Ensure the backend and libraries are installed.
4. Attempt installation when missing and reasonably possible.
5. Run the checker.
6. Record exact commands, versions, and errors.
7. Classify the result precisely.

### Failure classes

- `syntax_error`
- `missing_library_support`
- `missing_backend`
- `installation_failed`
- `under_specified_paper_step`
- `translation_ambiguity`
- `likely_invalid_step`
- `timeout`
- `unsupported_domain`

### Rules

1. Never equate formalization failure with theorem falsity.
2. Always preserve the original natural-language step next to the formal attempt.
3. Extend the formalized frontier until either:
   - the core theorem chain is covered, or
   - a precise blocking gap is isolated
4. If a formalization isolates one exact blocked premise, convert it into a new leaf obligation.
5. A positive formalization result counts only if the checker actually runs successfully.

### Lean-specific guidance

- Lean is the default first-choice backend when the domain is compatible.
- If Lean tooling is missing, attempt installation, typically via `elan`.
- If mathlib is needed, create or reuse a minimal project and fetch it before declaring the path blocked.
- Prefer small isolated files for reused lemmas or interfaces.
- A file using `sorry`, `axiom`, or skipped obligations does not count as verified.

### Outputs

- `outputs/formalization_attempts.json`
- backend-specific files under `temp/formal/`

---

## Stage K. Citation and Prior Result Validation

### Objective

Check whether cited prior facts are used correctly.

### For each critical cited result

1. identify the source
2. compare prerequisites with the current use
3. check for hidden strengthening
4. check for version mismatch
5. check whether the cited theorem proves what is being claimed

### Flags

- `citation_too_weak`
- `citation_imprecise`
- `citation_misapplied`
- `citation_unverifiable`

### Output

- `outputs/citation_validation.json`

---

## Stage L. Human-Facing Review Synthesis

### Objective

Produce an evidence-backed narrative report.

### Required sections in `final_report.md`

1. Executive summary
2. Scope of review
3. Paper structure summary
4. Core claim dependency summary
5. Claim-by-claim verification ledger
6. High-risk findings
7. Tool-based checks performed
8. Formalization attempts
9. Counterexample search results
10. Reproducibility and preserved artifacts
11. Unverified critical steps
12. Open obligations that forced the stopping point
13. Final verification verdict
14. Confidence statement
15. Limitations of this review

### Required reporting rule

If charitable reconstruction was used, the report must state:

- exact reconstructed span
- minimal reconstruction attempted
- whether it was notation-only or structure-changing
- whether it removed the issue, reduced it, or still left a substantive gap

### Approved summary language

- "The core theorem has been verified within the stated scope."
- "No concrete contradiction was found in the checked fragments."
- "Several high-risk transitions remain unverified."
- "The strongest concern is concentrated in Claim THM-002, Step PRF-002-S07."
- "Computational checks support tested instances but do not establish the general case."
- "A verified counterexample was found for the statement as written."
- "Verification remains incomplete because Claim THM-002 depends on unresolved Step PRF-002-S07."
- "The statement as written contains a repairable transcription defect, but the argument still lacks independent verification."
- "The review stopped only after the remaining unresolved burden was localized to the listed leaf obligations."

### Forbidden summary language

- "The paper is definitely correct." unless the positive threshold is met
- "The proof is valid." unless the positive threshold is met
- "Everything checks out." unless every core claim is verified
- "This theorem is true." without matching terminal evidence
- "The main theorem was reviewed." if it still has open blocking obligations
- "No major issues found." if any core claim remains nonterminal

### Output

- `outputs/final_report.md`

---

## Stage M. Machine-Readable Final Artifact

### Objective

Produce a strict machine-readable summary for downstream systems.

### Minimum top-level schema

```json
{
  "paper_metadata": {},
  "domain_profile": {},
  "claims": [],
  "prose_claims": [],
  "dependency_summary": {},
  "verification_ledger": [],
  "obligation_ledger": [],
  "risk_findings": [],
  "tool_checks": [],
  "counterexample_results": [],
  "formalization_results": [],
  "citation_validation": [],
  "preserved_artifacts": [],
  "critical_unverified_steps": [],
  "review_summary": {
    "overall_status": "verification_incomplete",
    "highest_risk_claims": [],
    "human_attention_priorities": [],
    "final_verdict": "",
    "confidence_statement": "",
    "limitations": []
  },
  "reconstruction_log": []
}
```

### Recommended fields for each `verification_ledger` entry

- `claim_id`
- `is_core_claim`
- `terminal_status`
- `direct_verification_attempts`
- `adversarial_verification_attempts`
- `blocking_leaf_obligations`
- `final_basis_for_status`
- `evidence_paths`

### Recommended fields for each `preserved_artifacts` entry

- `artifact_id`
- `linked_claim_id`
- `artifact_type`
- `path`
- `producer`
- `command`
- `dependencies`
- `replay_instructions`
- `notes`

### Recommended fields for each `reconstruction_log` entry

- `reconstruction_id`
- `claim_id`
- `source_span`
- `issue_class`
- `minimal_reconstruction`
- `reconstruction_scope`
- `introduces_new_assumptions`
- `post_reconstruction_status`
- `notes`

### Output

- `outputs/final_report.json`

---

## Stage N. Verdict Closure

### Objective

Determine whether the global verdict actually follows from the ledger.

### Required actions

1. identify the core claim set
2. inspect whether each core claim's dependency chain is terminally resolved
3. distinguish inconclusive tool failure from genuine proof failure
4. assign one global verdict:
   - `verified_correct_in_scope`
   - `contradicted_by_counterexample`
   - `proof_failure_established`
   - `verification_incomplete`
5. write a concise verdict justification citing exact claims and evidence

### Rules

- If any core claim is still unresolved, the verdict cannot be positive.
- If any unresolved dependency blocks a core claim, the verdict must be `verification_incomplete` unless a stronger negative verdict is justified.
- A positive verdict must state the verification scope explicitly.
- A downstream application failure does not automatically force a negative verdict for the whole paper.
- A proof failure for the statement as written may coexist with a typo hypothesis; record both, but classify the written text.
- A plausibly repairable transcription defect alone does not justify a negative global verdict.
- A substantive omitted argument that remains unresolved after charitable reconstruction blocks a positive verdict.
- If charitable reconstruction requires a new nontrivial lemma, new bookkeeping, or a new uniformity assumption, treat the original step as unresolved unless that added material is independently checked.

### Output

- `outputs/verdict_closure.json`

---

## Stage O. Optional Post-Publication Reconciliation

Run this stage only if the user explicitly requests comparison with errata, corrigenda, revised versions, or later author clarifications.

### Objective

Compare the paper-as-written findings against later materials without overwriting the original review.

### Required actions

1. preserve the original paper-as-written verdict
2. identify each finding or claim that later material bears on
3. classify the relation using labels such as:
   - `confirmed_by_erratum`
   - `partially_confirmed_by_erratum`
   - `not_confirmed_by_erratum`
   - `contradicted_by_erratum`
   - `superseded_by_erratum`
4. record whether the later source strengthens, weakens, or leaves unchanged the original finding
5. if an official erratum states that a lemma or theorem is false, incorrect, or incomplete, upgrade the finding language to match that stronger evidence
6. if later material says a downstream result is unaffected, but the local paper-as-written review found a stronger defect, mark that conflict explicitly and do not silently keep the stronger global claim unless it is independently re-established

### Rules

- Keep the reconciliation verdict separate from the original verdict.
- Do not rewrite the paper-as-written ledger to make it look as if the original text was better than it was.
- Distinguish a local transcription or notation defect from a mathematically acknowledged false statement.
- If later material and local review disagree, say so explicitly and classify the disagreement rather than collapsing it.

### Outputs

- `outputs/post_publication_reconciliation.json`
- `outputs/post_publication_reconciliation.md`

---

## 10. Tool Routing Policy

Use the following routing defaults.

| Situation | Preferred tool |
| --- | --- |
| Finite combinatorial cases | Python enumerator, SAT, Sage |
| Symbolic identities | SymPy, Sage |
| Modular arithmetic examples | Sage, PARI, Python |
| Local formal proof attempt | Lean, Coq |
| Estimate-chain sanity audit | Python plus structured audit |
| Citation prerequisite mismatch | citation validator plus manual compare |
| Notation consistency | parser plus LLM scan |

If no suitable tool exists:

1. mark the path as blocked or unavailable
2. keep the issue in the obligation ledger
3. do not pretend the claim has been covered

---

## 11. Failure Handling

If any stage fails:

1. record the stage
2. record the exact reason
3. continue in degraded mode only where safe
4. do not fabricate outputs

Examples:

- If dependency extraction is partial, continue but mark the graph incomplete.
- If the formalization backend is unavailable, keep the target in the unresolved queue.
- If symbolic computation times out, reduce scope and note the limitation.
- If a decisive check was performed but its artifacts were not preserved, the review remains incomplete until artifacts are reconstructed or the check is downgraded.

---

## 12. Human Escalation Triggers

Escalate for human review when any of the following occurs:

1. a verified counterexample is found
2. a core theorem depends on a suspicious lemma
3. multiple hidden assumptions are detected in the main proof chain
4. a cited theorem appears misapplied in a critical step
5. the statement as written differs materially from the proof target
6. a formalization attempt indicates likely invalid inference
7. notation changes alter the meaning of a central object

### Escalation output

- `outputs/human_escalation.md`

---

## 13. Final Markdown Template

Use this structure in `final_report.md`.

```md
# Review Report

## 1. Executive Summary

## 2. Scope of This Review

## 3. Structural Summary

## 4. Core Dependency Chain

## 5. Claim-by-Claim Verification Ledger

## 6. High-Risk Findings

## 7. Tool-Based Checks Performed

## 8. Formalization Attempts

## 9. Counterexample Search Results

## 10. Reproducibility and Preserved Artifacts

## 11. Unverified Critical Steps

## 12. Open Obligations That Forced the Stopping Point

## 13. Final Verification Verdict

## 14. Confidence Statement

## 15. Limitations of This Review
```

---

## 14. Recommended Execution Order

Run stages in this order unless a paper-specific blocker forces a temporary detour:

```text
A -> B -> C -> D -> E -> F -> G -> G.5 -> H -> I -> J -> K -> G.5 -> N -> M -> L
```

Notes:

- `G.5` is not optional. It is the closure mechanism.
- After H, I, J, and K, revisit `G.5`.
- Do not write the final report before verdict closure is complete.

---

## 15. Recommended First-Version Scope

For MVP deployments, prefer papers where the workflow is most likely to close cleanly:

- combinatorics
- graph theory
- elementary number theory
- algebra with explicit constructions

Use extra caution before offering strong support for:

- PDE
- geometric analysis
- advanced algebraic geometry
- highly implicit categorical arguments

---

## 16. Reviewer Philosophy

This system is a verification engine with explicit honesty constraints.

It should:

1. certify correctness when justified
2. detect incorrectness when justified
3. otherwise say `verification_incomplete`

It does not replace expert mathematics, but it should push much closer to an actual correctness determination than an ordinary narrative review.
