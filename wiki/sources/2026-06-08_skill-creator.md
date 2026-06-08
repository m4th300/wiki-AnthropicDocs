---
type: source
created: 2026-06-08
updated: 2026-06-08
status: current
author: Anthropic
date_source: 2026-06-08
raw_file: raw/skill-creator/SKILL.md + agents/ + references/schemas.md
tags: [domain/claude-code, theme/extensions]
---

# Skill Creator — Skill for creating and iterating skills

## Résumé

`skill-creator` is a bundled Claude Code skill for creating, testing, evaluating, and optimizing other skills. It operationalizes a full skill development loop: draft → run evals → grade → compare → improve → optimize description. It ships with 3 specialized subagents (grader, comparator, analyzer) and a set of Python scripts for running benchmarks and the description optimization loop.

## Points clés

### Skill anatomy (from skill-creator)

```
skill-name/
├── SKILL.md              # required — frontmatter + instructions
├── evals/
│   └── evals.json        # test cases with prompts and assertions
├── scripts/              # helper scripts (run without loading into context)
├── references/           # docs loaded on demand
└── assets/               # output templates, icons
```

**3-level progressive disclosure:**
1. `name + description` — always in context (~100 words)
2. `SKILL.md` body — on activation (target <500 lines; if longer, add hierarchy + pointers to references)
3. Bundled resources (`scripts/`, `references/`, `assets/`) — on demand

### Skill creation workflow (5 stages)

**1. Capture intent**
- Extract from conversation history first (tools used, sequence, corrections)
- Answer: what should it do, when should it trigger, what's the output format, are test cases needed?

**2. Interview & research**
- Ask about edge cases, input/output formats, success criteria, dependencies
- Check available MCPs for useful research; parallelize with subagents if available

**3. Write SKILL.md**
- `name`: identifier
- `description`: **primary triggering mechanism** — include what + when. Be "pushy" to combat undertriggering.
  - Bad: `"How to build a dashboard"`
  - Good: `"How to build a dashboard. Use whenever the user mentions dashboards, data visualization, or wants to display any kind of data, even if they don't explicitly ask for a 'dashboard.'"`
- Keep under 500 lines; bundle heavy content in `scripts/` or `references/`
- Prefer explaining WHY over heavy-handed ALWAYS/NEVER imperatives
- Scripts that reappear across test cases → bundle them in `scripts/`

**4. Test cases** (`evals/evals.json`)
```json
{
  "skill_name": "my-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "realistic user prompt",
      "expected_output": "description of success",
      "files": [],
      "expectations": ["The output includes X", "The script Y was used"]
    }
  ]
}
```
Test prompts should be realistic, detailed, with backstory — not abstract. Good: specific file names, user context, typos, casual phrasing.

**5. Iterate**
- Run test cases (with-skill + baseline) as parallel subagents
- Grade with `agents/grader.md`
- Aggregate with `scripts/aggregate_benchmark.py`
- Analyze with `agents/analyzer.md`
- View with `eval-viewer/generate_review.py`
- Improve → re-run → repeat until happy

### The 3 specialized subagents

**Grader** (`agents/grader.md`)
- Evaluates each assertion against transcript + output files
- Also extracts and verifies implicit claims from outputs
- Critiques the evals themselves (flags trivially-satisfied or missing assertions)
- Output: `grading.json` with `{text, passed, evidence}` per assertion (exact field names required by viewer)

**Comparator** (`agents/comparator.md`)
- Blind A/B comparison: receives two outputs without knowing which skill produced them
- Scores on content rubric (correctness, completeness, accuracy) + structure rubric (organization, formatting, usability)
- Output: `comparison.json` with winner, reasoning, per-output scores

**Analyzer** (`agents/analyzer.md`)
Two modes:
1. *Post-hoc analysis*: after blind comparison, "unblinds" the result — reads both skills and transcripts to explain WHY the winner won, generates improvement suggestions per category (instructions, tools, examples, error_handling, structure, references)
2. *Benchmark analysis*: surfaces patterns aggregate stats hide — non-discriminating assertions, flaky evals, time/token outliers

### Description optimization loop

**Purpose**: the `description` field is the primary trigger mechanism. Claude tends to undertrigger. This loop optimizes it empirically.

**Process:**
1. Generate 20 eval queries: 8-10 should-trigger + 8-10 should-not-trigger (near-misses, not obviously irrelevant)
2. Review with user via `assets/eval_review.html` template
3. Run optimization: `python -m scripts.run_loop --eval-set ... --skill-path ... --max-iterations 5`
   - 60/40 train/test split (avoids overfitting to train)
   - Each iteration: evaluate current description (3 runs per query) → Claude proposes improvements → re-evaluate → repeat
   - Selects best by test score, not train score
4. Apply `best_description` to SKILL.md frontmatter

**When skills trigger**: Claude only consults skills for tasks it can't easily handle alone. Simple one-step queries rarely trigger skills. Test prompts must be substantive multi-step requests.

### Key JSON schemas

| File | Location | Purpose |
|---|---|---|
| `evals.json` | `evals/evals.json` | Test cases + assertions |
| `grading.json` | `<run-dir>/grading.json` | Grader output per run |
| `benchmark.json` | `<workspace>/iteration-N/` | Aggregated stats across runs |
| `timing.json` | `<run-dir>/timing.json` | Wall clock timing (from subagent notification — not persisted elsewhere) |
| `metrics.json` | `<run-dir>/outputs/` | Tool usage counts |
| `comparison.json` | `<grading-dir>/` | Blind comparator result |
| `analysis.json` | `<grading-dir>/` | Post-hoc analyzer result |

**Critical**: `grading.json` expectations must use `text`, `passed`, `evidence` (not `name`/`met`/`details`) — the eval viewer depends on these exact field names.

### Platform variations

| Platform | Key differences |
|---|---|
| Claude Code (default) | Subagents + browser + `claude -p` all available |
| Claude.ai | No subagents → serial test runs, no browser viewer, no description optimization |
| Cowork | Subagents yes, no browser → use `--static` for viewer HTML |

## Liens wiki

- [[features/skills]] — the feature being created and iterated with this skill
- [[features/subagents]] — the 3 subagents (grader, comparator, analyzer) use the subagent mechanism
- [[frameworks/agentic-patterns]] — skill-creator itself is a multi-agent workflow (fan-out test runs, parallel baseline/with-skill)

## Tensions

Aucune tension avec le contenu existant.

**Note interne**: Le `raw/skills.md` documentait déjà `skills` comme feature (portée, frontmatter, invocation). Le skill-creator ajoute la couche *méta* : comment créer, tester et optimiser des skills. Ces deux couches sont complémentaires.

## Questions soulevées

- Le timing.json est capturé depuis la notification de complétion de subagent (non persisté ailleurs). Est-ce aussi le cas dans l'Agent SDK, ou `ResultMessage` expose-t-il `duration_ms` ?
- Le seuil de 500 lignes pour SKILL.md est une recommandation — quelle est la limite dure (context budget) ?
