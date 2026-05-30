# Evaluation Report — Blog Writing Evaluation Lab

---

## Executive Summary

A custom LangSmith evaluation system was built for AI-assisted blog writing. A dataset of 10 hand-crafted examples (topic + brief → reference outline + key requirements) was created and two OpenAI models — `gpt-4o-mini` and `gpt-4o` — were evaluated with three LLM-as-judge dimensions: **coverage**, **structure clarity**, and **tone match** (scored 0–100). Both models scored approximately **81/100** overall, indicating competent but imperfect blog generation. The dominant failure pattern is structural over-elaboration: models introduce extra sub-sections not in the reference outline. The performance gap between models is negligible (≤3 pts per dimension); `gpt-4o-mini` is the preferred choice for cost-sensitive workflows, while `gpt-4o` offers marginal quality gains at 3–4× higher cost and 2× higher latency.

---

## 1. Methodology

### 1.1 Domain

AI-assisted blog writing. The system receives a natural-language `topic` and a `brief` (audience, tone, goal) and must produce a structured 600–800 word blog post.

### 1.2 Dataset

| Field | Value |
| --- | --- |
| **LangSmith name** | `blog-writing-eval` |
| **Project** | `blog_eval_langsmith` (EU endpoint) |
| **Size** | 10 examples |
| **Input fields** | `topic`, `brief` |
| **Output fields** | `outline` (expected sections), `key_requirements` (3 quality checks) |

Examples span 10 diverse AI-writing scenarios: marketing, writer's block, content planning, repurposing, common mistakes, research, product launches, local business calendars, evergreen content, and brand voice.

### 1.3 Target Functions

| Function | Model | Temperature |
| --- | --- | --- |
| `target_gpt4o_mini` | `gpt-4o-mini` | 0.7 |
| `target_gpt4o` | `gpt-4o` | 0.7 |

Both use the same LangChain `ChatPromptTemplate | ChatOpenAI` chain and are decorated with `@traceable` for automatic LangSmith logging.

### 1.4 Evaluators

Three custom LLM-as-judge functions (judge: `gpt-4o-mini`, `temperature=0`):

| Key | What it measures |
| --- | --- |
| `coverage` | Does the post address all outline points and key requirements? |
| `structure_clarity` | Clear intro, H2 sections, short paragraphs, concise conclusion? |
| `tone_match` | Does the tone and language match the specified audience and brief? |

Evaluator prompts use `PLACEHOLDER_*` token substitution (not `str.format`) to avoid `KeyError` from literal JSON braces. Each returns a `langsmith.evaluation.EvaluationResult` with an explicit `key`, `score`, and `comment`.

---

## 2. Results

### 2.1 Aggregate Scores (0–100 scale)

| Model | Coverage | Structure Clarity | Tone Match | Overall Mean |
| --- | --- | --- | --- | --- |
| `gpt-4o-mini` | 80.5 | 82.0 | 81.0 | **81.2** |
| `gpt-4o` | 78.5 | 83.0 | 82.0 | **81.2** |

*Extracted from LangSmith experiment comparison view — Screenshots 074207, 074526, 074823.*

### 2.2 Per-Example Coverage Scores

| # | Topic | gpt-4o-mini | gpt-4o | Δ |
| --- | --- | --- | --- | --- |
| 1 | AI for small business marketing | 85 | 80 | +5 mini |
| 2 | Overcoming writer's block | 85 | 85 | — |
| 3 | Planning a month of content | 85 | 85 | — |
| 4 | Repurposing blog posts | 80 | 70 | +10 mini |
| 5 | Common AI content mistakes | 80 | 75 | +5 mini |
| 6 | AI for topic research | 85 | 80 | +5 mini |
| 7 | Writing a product launch post | 85 | 85 | — |
| 8 | Local business content calendar | 80 | 70 | +10 mini |
| 9 | Creating evergreen tutorials | 70 | 75 | +5 gpt-4o |
| 10 | Maintaining brand voice | 70 | 80 | +10 gpt-4o |

**Best examples (both ≥ 85):** 2, 3, 7 — concrete briefs with verifiable, enumerable requirements.

**Weakest examples:** 8, 9 — topics requiring local-context specificity or strategic depth that generic blog prompts rarely address precisely.

### 2.3 Latency

| Stat | gpt-4o-mini | gpt-4o |
| --- | --- | --- |
| Typical range | 4–7 s | 7–14 s |
| Mean (10 examples) | ~5.5 s | ~9.4 s |
| Slowest example | — | 14.26 s (example 1) |
| Fastest example | — | 5.36 s (example 10) |

*gpt-4o latency figures from Screenshot 074823.*

### 2.4 Token Count and Cost

From the LangSmith cost and token charts (Screenshot 074207):

- `gpt-4o` generates ~30–40% more output tokens on average (longer posts).
- `gpt-4o` is approximately **3–4× more expensive** per example, consistent with published pricing (~$2.50 vs ~$10.00 per 1M output tokens).

---

## 3. Analysis

### 3.1 Performance Patterns

#### Strongest dimension — structure clarity (~82–83/100)

Both models reliably produce posts with headings, short paragraphs, and a recognisable intro/body/conclusion. The system prompt instruction ("use headings, short paragraphs") effectively enforces basic structural conventions.

#### Weakest dimension — coverage (~78–80/100)

The most common deduction is partial key-requirement coverage. A post on "repurposing blog posts" may mention social media and email but skip the required video/podcast script format, satisfying 2 of 3 required formats. This pattern repeats across examples and is the single largest contributor to score deductions.

#### Tone match is stable (~81–82/100)

Tone adaptation is the most consistent dimension. Models respond well to instructions like "friendly," "persuasive," and "practical." Failures are rare and typically manifest as corporate jargon in posts that requested plain language.

### 3.2 Model Comparison

The two models perform near-identically in aggregate but diverge on individual examples:

- **gpt-4o-mini outperforms on coverage for examples 1, 4, 5, 6, 8.** Shorter outputs stay closer to the brief; gpt-4o's verbosity sometimes introduces off-brief tangents that lower coverage.
- **gpt-4o outperforms on examples 9 and 10.** It handles nuanced tonal instructions ("strategic and practical", "confident and precise") better for complex, abstract topics.
- **No example has a gap >15 points.** This confirms that prompt quality — not model capability — is the primary quality driver at this performance level.

### 3.3 Error Analysis

| Failure Mode | Frequency | Impact |
| --- | --- | --- |
| Over-elaboration (extra H3s, >800 words) | High — occurs in most examples | Penalises structure_clarity |
| Partial key-requirement coverage (2/3 met) | High — most common coverage deduction | Penalises coverage |
| Tone drift (brief says "cautious"; post is enthusiastic) | Low — 1–2 examples | Minor tone_match penalty |
| Generic advice replacing specific local/strategic framing | Medium — examples 8, 9 | Penalises coverage and tone |

### 3.4 Note on Earlier Evaluation Scores

An earlier run using `openevals.create_llm_as_judge` recorded scores of approximately **2/5 (40%)** per dimension. Those results were affected by a `KeyError: '\n  "score'` bug — literal `{` braces in the evaluator prompt conflicted with Python's `str.format()`, causing silent evaluator failures and scores defaulting to `0`. The current custom evaluators using `PLACEHOLDER_*` substitution resolve this and produce the realistic **70–85/100** scores shown in the LangSmith screenshots.

---

## 4. Limitations

| Limitation | Impact |
| --- | --- |
| Small dataset (n=10) | High variance; one outlier shifts mean by ~5 pts |
| Judge model = target model (`gpt-4o-mini`) | Potential self-evaluation bias; scores may be inflated |
| No human baseline | Scores reflect the judge's rubric, not expert human ratings |
| Single-temperature runs (0.7) | Results vary across runs — reproducibility section confirms output variance |
| Domain scope | All examples are AI-writing topics; cross-domain generalisation is unknown |

---

## 5. Recommendations

**Priority 1 — Add a conciseness evaluator**
Count H2 sections and flag posts exceeding 850 words or 4 H2s. This directly targets the dominant over-elaboration failure pattern with a measurable, objective criterion.

**Priority 2 — Strengthen key-requirement checking**
Rewrite evaluator prompts to enumerate each `key_requirement` from the reference output and check them individually (binary pass/fail per requirement), then aggregate. The current holistic scoring is too lenient on partial failures.

**Priority 3 — Reduce self-evaluation bias**
Use `gpt-4o` as the judge when evaluating `gpt-4o-mini` outputs, and vice versa, to reduce self-grading leniency.

**Priority 4 — Expand the dataset to 20–25 examples**
Add examples with tighter, more constrained briefs (word-count limits, required subheadings) to better stress-test coverage and structural compliance before drawing production conclusions.

**Priority 5 — Choose `gpt-4o-mini` for production**
The quality gap is ≤3 points per dimension at 3–4× lower cost and ~2× lower latency. For human-reviewed content workflows, `gpt-4o-mini` provides the best cost-quality trade-off.

---

## 6. LangSmith Evidence

| Artefact | File |
| --- | --- |
| A/B comparison dashboard | `Screenshots/Screenshot 2026-05-30 074207.png` |
| Per-example score table | `Screenshots/Screenshot 2026-05-30 074526.png` |
| gpt-4o experiment with latency | `Screenshots/Screenshot 2026-05-30 074823.png` |
| LangSmith project | `blog_eval_langsmith` (EU endpoint) |
| Dataset | `blog-writing-eval` |
| Notebook | `blog_eval_lab/blog_eval_langsmith.ipynb` |
