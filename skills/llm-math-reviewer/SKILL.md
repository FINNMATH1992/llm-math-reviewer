---
name: llm-math-reviewer
description: Evidence-driven verification workflow for mathematical papers in repositories with LaTeX sources, PDFs, bibliography files, and review tooling. Use when Codex needs to review a theorem paper, extract claims and dependencies, pressure-test proofs, run symbolic or formal checks, maintain an explicit obligation ledger, and produce a strict verdict such as verified, contradicted by counterexample, proof failure established, or verification incomplete. Default to a paper-as-written review that ignores errata and later material unless the user explicitly asks for a post-publication comparison.
---

# LLM Math Reviewer

Use this skill to review a mathematical paper as a verification task, not as a plausibility summary. Push each core claim toward a justified terminal status while preserving enough artifacts for a human or another agent to audit the work.

## Quick Start

1. Inspect the repository for the paper source, bibliography, appendices, macro files, and `review_config.yaml`.
2. Before writing any artifacts, create a paper-specific review root `review/papers/<paper_slug>/`, where `<paper_slug>` is derived from the reviewed paper filename or source directory name. If that slug is ambiguous, append a stable disambiguator such as the year or a short hash. Never write paper-specific outputs directly into a shared `review/outputs/`, `review/temp/`, or `review/bak/`.
3. Prefer existing `review/scripts/` helpers when they fit; create or adapt small reusable scripts when they do not.
4. Run the stages in order: intake, parsing, claim extraction, dependency analysis, risk scan, domain classification, target selection, obligation ledger, executable checks, verdict closure, machine-readable artifact, human report.
5. Revisit the obligation ledger after each major checking pass. Do not treat one forward pass as complete.
6. Treat any "stuck" state as a workflow failure, not a reason to stop. Convert it immediately into a smaller next action, an alternate verification route, or an explicit blocking leaf obligation.
7. End only when every core claim has a terminal status or an explicit blocking leaf obligation recorded as the reason for `verification_incomplete`.

Read `references/math-review-sop.md` when you need the full stage-by-stage contract, exact schemas, approved labels, report wording rules, or escalation triggers.

## Treat the Task Correctly

- Treat the paper's core claims as the unit of completion.
- Tie every finding to an exact source location, claim ID, evidence, and confidence.
- Distinguish what is proven, what is computationally supported, what is contradicted, and what remains unresolved.
- Treat absence of a discovered error as non-evidence. It does not justify a positive verdict.
- Treat tool failure as a workflow limitation unless the failure itself establishes a mathematical inconsistency.
- Treat "I do not yet know how to proceed" as a trigger to decompose the current obligation, switch verification mode, or move to another unresolved bottleneck. It is not a valid stopping point.
- Preserve scripts, commands, witnesses, raw outputs, and interpretation for decisive checks.
- Do not search for errata, retractions, corrigenda, commentary, or later discussion of the paper unless the user explicitly asks for a post-publication comparison.
- Do not use information published after the paper itself in the default review mode. If later information is retrieved accidentally, ignore it and continue the review using only contemporaneous paper materials and allowed repository inputs unless the user has explicitly requested a reconciliation pass.
- If the user explicitly asks to compare against errata, revised versions, or later author clarifications, keep that work in a separate reconciliation layer and do not overwrite the original paper-as-written verdict.

## Use These Terminal Outcomes

For core claims, finish only with:

- `verified`
- `contradicted_by_counterexample`
- `proof_failure_established`
- `verification_incomplete`

For the global paper verdict, finish only with:

- `verified_correct_in_scope`
- `contradicted_by_counterexample`
- `proof_failure_established`
- `verification_incomplete`

Do not stop at labels such as `reviewed_textually`, `looks_plausible`, `standard argument`, or `no contradiction found`.

## Maintain Forward Motion

- Never stop the review merely because the current path is inconclusive. If one route stalls, immediately choose another concrete route in the same session.
- After every failed, ambiguous, or non-decisive attempt, record what was learned and name the very next action for the same claim or its nearest unresolved dependency.
- When a proof is too large or vague to check directly, shrink the task: isolate a subclaim, a witness, a hidden assumption, a missing bound, or a cited lemma, then verify that smaller unit next.
- If direct proof checking stalls, switch modes instead of pausing: try adversarial search, symbolic computation, finite-instance testing, notation auditing, dependency tracing, or citation validation.
- If a tool fails, either repair the tooling path, replace it with another executable path, or record a precise blocked obligation and continue with a different unresolved obligation.
- Keep at least one active next step for every nonterminal core claim. A claim without a next step is a workflow bug that must be fixed before ending.
- Only close the review when no feasible next verification action remains for any nonterminal claim and the remaining blockers are explicitly recorded.

## Follow This Workflow

### 1. Establish Inputs and Working Layout

- Locate the main source file, bibliography, appendices, and macros.
- Read `review_config.yaml` if present.
- Use or create a review layout that can hold scripts, outputs, temporary formalization files, and paper-specific helpers.
- Put all paper-specific artifacts under `review/papers/<paper_slug>/`. Reserve `review/scripts/` for reusable helpers shared across papers.
- Fall back to PDF or text extraction only if TeX parsing fails, and record the degraded mode explicitly.
- Restrict sources to the paper, its repository materials, and sources that predate or are contemporaneous with the paper's own cited foundations when those are available in the repository or otherwise intentionally provided.

### 2. Parse Structure and Extract Claims

- Build a normalized text representation.
- Extract theorem-like objects, proofs, definitions, assumptions, remarks, and examples.
- Link each theorem-like item to its proof when present.
- Canonicalize each theorem-like statement into a claim record with hypotheses, objects, conclusion, proof link, and source span.
- Split conjunctions or exact downstream consequences into subclaims when that improves verifiability.

### 3. Build Dependencies and Risk Profile

- Build a directed dependency graph over claims and definitions.
- Distinguish explicit citations, implicit usage, notation dependencies, and hidden-assumption candidates.
- Run a risk scan before expensive checks.
- Flag compressed steps, undefined symbols, scope drift, omitted witnesses, circularity, case splits, and citation vagueness.
- Choose a domain profile that matches the paper instead of applying one generic checking strategy.

### 4. Select Targets and Maintain the Obligation Ledger

- Prioritize core theorem bottlenecks, high-severity risks, reused lemmas, computational subarguments, and claims with plausible counterexamples.
- Create an obligation ledger for every core claim and each unresolved bottleneck on its dependency chain.
- Split large obligations into leaf obligations until each blocker is specific and actionable.
- For every active leaf obligation, record the next attempted action, not only the problem statement.
- Update the ledger after symbolic checks, counterexample search, formalization attempts, and citation validation.
- Use the ledger, not intuition, to decide whether the review is actually closed.

### 5. Attempt Direct and Adversarial Verification

- Attempt at least one direct verification path for every core claim.
- Attempt at least one adversarial path for universal or easily falsifiable claims when feasible.
- If the first path is inconclusive and a feasible second path exists, take it before downgrading the claim to unresolved.
- Use executable tools whenever the mathematics admits direct checking.
- If a needed dependency or package is missing, check whether it is already installed and attempt installation when reasonably possible.
- Record installation attempts, versions when available, and whether the dependency was later used.
- Treat sampled computation as supportive only; keep the obligation open unless exhaustiveness is independently justified.

### 5A. Recover From Stalls Explicitly

- When progress stalls, do not summarize and stop. Run a recovery loop:
  1. Name the exact obstacle.
  2. Re-express it as a smaller leaf obligation.
  3. Choose the cheapest credible next route.
  4. Execute that route or record the precise blocker.
  5. Move immediately to the next unresolved obligation if the current one remains blocked.
- Valid recovery routes include:
  - replacing a global theorem check with a lemma-level check
  - extracting a candidate hidden assumption and testing whether the proof uses it
  - searching for a small counterexample before attempting full proof reconstruction
  - validating a cited dependency locally when the main proof compresses over it
  - converting prose mathematics into a bounded symbolic or finite computation
- Do not end a session while a claim is merely "partially analyzed" or "needs more checking" if a concrete next action is still available.

### 6. Close the Verdict Carefully

- Re-run target selection against the current obligation ledger, not only the initial risk report.
- Audit every unresolved leaf obligation before assigning the final verdict.
- If any unresolved leaf obligation still has a feasible next action, perform that action before closing.
- Assign a positive global verdict only if every core claim is verified within an explicit scope.
- Assign a negative verdict only when a verified counterexample, explicit contradiction, or precise proof failure is established.
- Otherwise assign `verification_incomplete`.

### 7. Optional Post-Publication Reconciliation Mode

Use this mode only if the user explicitly asks to compare the review against errata, corrigenda, revised versions, or later author clarifications.

- Keep the original paper-as-written verdict unchanged.
- Treat later materials as a separate reconciliation layer, not as replacement evidence for the original review.
- For each prior finding, classify its relation to later materials with labels such as:
  - `confirmed_by_erratum`
  - `partially_confirmed_by_erratum`
  - `not_confirmed_by_erratum`
  - `contradicted_by_erratum`
  - `superseded_by_erratum`
- If an official erratum directly states that a lemma or theorem is false, incorrect, or incomplete, prefer the strongest supported phrasing instead of a weaker local description.
- If a local as-written defect appears decisive but an official erratum explicitly states that the downstream result is unaffected, do not silently keep the stronger global claim. Reclassify the local issue as a transcription defect, notation defect, or unresolved inconsistency unless you independently resolve the contradiction from source-level evidence.
- Record whether each later source strengthens, weakens, or leaves unchanged the original finding.

## Produce These Core Outputs

At minimum, produce or maintain these artifacts when the repository layout supports them.
All paths below are relative to the paper-specific review root `review/papers/<paper_slug>/`.

- `outputs/intake_summary.json`
- `outputs/normalized_text.txt`
- `outputs/parsed_structure.json`
- `outputs/claims.json`
- `outputs/dependency_graph.json`
- `outputs/risk_report.json`
- `outputs/domain_profile.json`
- `outputs/check_targets.json`
- `outputs/obligation_ledger.json`
- `outputs/tool_checks_symbolic.json`
- `outputs/counterexample_search.json`
- `outputs/formalization_attempts.json`
- `outputs/citation_validation.json`
- `outputs/verdict_closure.json`
- `outputs/final_report.json`
- `outputs/final_report.md`

If the user explicitly requests post-publication reconciliation, also produce:

- `outputs/post_publication_reconciliation.json`
- `outputs/post_publication_reconciliation.md`

Read `references/math-review-sop.md` for the exact required fields, stage order, and report template.

## Route Tools Deliberately

- Use Python enumerators, SAT, or Sage for finite combinatorial cases.
- Use SymPy or Sage for symbolic identities.
- Use Python plus structured audits for estimate chains and notation consistency.
- Use Lean, Coq, Isabelle, or HOL Light only when the domain and tooling make that route realistic.
- If no suitable tool exists, record the path as blocked and keep the claim open.

## Preserve Honesty in Reporting

- Use cautious language that matches the evidence.
- State when computational checks cover only tested instances.
- State exactly which span required charitable reconstruction, what was reconstructed, and whether the issue remained substantive.
- Do not claim that the paper or theorem is correct unless the positive threshold is met.
- Do not say "everything checks out" or "no major issues found" while any core claim remains nonterminal.

## Escalate When the Review Finds Something Serious

Escalate for human review when you find a verified counterexample, a suspicious upstream lemma in the core chain, multiple hidden assumptions, a likely citation misapplication, a material mismatch between the written statement and the proof target, a likely invalid inference isolated by formalization, or a notation change that alters a central object's meaning.

## Reference

- Read `references/math-review-sop.md` for the full SOP, exact labels, detailed stage contracts, closure rules, and final report template.
