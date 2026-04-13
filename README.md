# LLM Math Reviewer

![LLM Math Reviewer logo](./assets/logo.svg)

`llm-math-reviewer` is a Codex plugin for evidence-driven verification of mathematical papers. It is an LLM-driven review workflow: the model is guided to treat paper review as structured claim verification rather than a plausibility summary or a loose reading pass.

In practice, this plugin turns a general-purpose large language model into a more disciplined math-paper reviewer by forcing explicit claims, dependency tracking, obligation ledgers, adversarial checks, and closure rules.

## Usage Note

For difficult papers, this plugin works best when the model is allowed to use the highest reasoning setting available. Lower reasoning settings are more likely to stop early, compress obligations too aggressively, or miss subtle dependency and citation issues.

## What It Does

- uses a large language model as the main reasoning engine, but constrains it with a verification-oriented workflow
- extracts theorem-like claims and proof structure
- builds dependency and risk views over the paper
- maintains an obligation ledger for unresolved proof steps
- keeps pushing when a verification path stalls by forcing alternate routes or smaller sub-obligations
- produces machine-readable outputs and a human-readable final report

## Why This Exists

General-purpose LLMs are often willing to stop too early, accept vague proof sketches, or settle for "looks plausible." This plugin exists to counter that failure mode. It gives the model a stricter review process so it keeps moving until claims are verified, contradicted, or explicitly left open with recorded blockers.

## Practical Note

We have tested this workflow on a small number of papers that were later associated with published corrections or errata. In those cases, the plugin was able to surface parts of the argument that merited further scrutiny and, in practice, pointed toward the sections that eventually required correction.

In limited testing, we have also seen cases where parts of a preprint highlighted by the workflow were later revised, clarified, or expanded in subsequent versions. We do not draw case-by-case conclusions from such observations, but they suggest that the workflow can sometimes identify sections of an argument that deserve closer review.

To avoid unnecessary controversy, this repository does not include case-by-case results or name specific papers.

## Disclaimer

- This plugin is a review aid, not an automated authority on mathematical correctness.
- Its outputs should be treated as structured review targets, not as final judgments, unless decisive evidence is explicitly recorded.
- A flagged step, unresolved obligation, or failed verification path does not by itself establish that a theorem or paper is false.
- Tool failure, parser failure, or formalization failure should be interpreted as workflow limitations unless they directly expose a mathematical contradiction.
- Human mathematical judgment remains necessary, especially for compressed arguments, domain-specific techniques, and borderline cases.
- This repository is intended for methodological experimentation and careful review support, not for making public accusations about specific papers or authors.

## Repository Layout

```text
.codex-plugin/plugin.json
skills/llm-math-reviewer/SKILL.md
skills/llm-math-reviewer/agents/openai.yaml
skills/llm-math-reviewer/references/math-review-sop.md
```

## Local Installation

There are two installation paths.

### Option 1: Let Codex Install It

1. Clone this repository to a local plugin path such as `~/plugins/llm-math-reviewer`.
2. Give Codex that path directly and ask Codex to install the plugin.

### Option 2: Install It Manually

1. Clone this repository.
2. Put the repository at `~/plugins/llm-math-reviewer` or another local plugins directory.
3. Make sure your Codex plugin marketplace includes an entry pointing to `./plugins/llm-math-reviewer`.
4. Reload Codex or reinstall the plugin so the updated manifest and skill files are picked up.

Example marketplace entry:

```json
{
  "name": "llm-math-reviewer",
  "source": {
    "source": "local",
    "path": "./plugins/llm-math-reviewer"
  },
  "policy": {
    "installation": "AVAILABLE",
    "authentication": "ON_INSTALL"
  },
  "category": "Productivity"
}
```

## Release Checklist

- replace `[TODO: ...]` fields in `.codex-plugin/plugin.json`
- update any remaining metadata fields after creating the GitHub repo
- tag a release after the first public push if you want a stable version reference

## License

MIT

## Development Notes

The main workflow lives in `skills/llm-math-reviewer/SKILL.md`. The plugin is intentionally instruction-heavy: it emphasizes explicit obligations, adversarial checks, and forward motion when a proof review gets stuck.
