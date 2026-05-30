# Evaluation Summary

## Domain & Dataset

**Domain:** AI-assisted blog writing. The system receives a `topic` and a `brief` (audience, tone, goal) and must produce a structured 600–800 word blog post with clear headings, short paragraphs, and a voice that matches the brief.

**Dataset:** `blog-writing-eval` — 10 hand-crafted examples covering diverse sub-topics: small-business marketing, writer's block, content planning, post repurposing, common AI mistakes, research, product launches, local business calendars, evergreen tutorials, and brand voice.

Each example has:

- **Inputs:** `topic` + `brief`
- **Reference outputs:** `outline` (expected sections) + `key_requirements` (3 specific quality checks)

---

## Methodology

**Target function:** `generate_blog` — a LangChain `ChatPromptTemplate | ChatOpenAI` chain with `@traceable`, tested at `gpt-4o-mini` and `gpt-4o` (both `temperature=0.7`).

**Evaluators:** Three custom LLM-as-judge functions (judge: `gpt-4o-mini`, `temperature=0`), each returning a score 0–100:

| Dimension | What it measures |
| --- | --- |
| `coverage` | Does the post address all outline points and key requirements? |
| `structure_clarity` | Clear intro, descriptive H2 sections, short paragraphs, concise conclusion? |
| `tone_match` | Does the tone and language match the specified audience and brief? |

**Infrastructure:** LangSmith EU endpoint, project `blog_eval_langsmith`, dataset `blog-writing-eval`, `max_concurrency=2`.

---

## Results

### Model Scores (0–100 scale)

| Model | Coverage | Structure Clarity | Tone Match | Overall Mean |
| --- | --- | --- | --- | --- |
| `gpt-4o-mini` | 80.5 | 82.0 | 81.0 | **81.2** |
| `gpt-4o` | 78.5 | 83.0 | 82.0 | **81.2** |

*Scores extracted from LangSmith experiment comparison (Screenshots 074207, 074526, 074823).*

### Per-Example Coverage Scores

| # | Topic | gpt-4o-mini | gpt-4o |
| --- | --- | --- | --- |
| 1 | AI for small business marketing | 85 | 80 |
| 2 | Overcoming writer's block | 85 | 85 |
| 3 | Planning a month of content | 85 | 85 |
| 4 | Repurposing blog posts | 80 | 70 |
| 5 | Common AI content mistakes | 80 | 75 |
| 6 | AI for topic research | 85 | 80 |
| 7 | Writing a product launch post | 85 | 85 |
| 8 | Local business content calendar | 80 | 70 |
| 9 | Creating evergreen tutorials | 70 | 75 |
| 10 | Maintaining brand voice | 70 | 80 |

---

## Key Findings

**Both models score ~81/100 overall** — competent but with clear room for improvement.

**Strongest dimension: structure_clarity (~82–83/100)**
Both models reliably produce posts with headings, paragraphs, intro, and conclusion. The system prompt instructions ("use headings, short paragraphs") are effective.

**Weakest dimension: coverage (~78–80/100)**
The most common failure is partial key-requirement coverage: posts typically satisfy 2 of 3 required use cases or formats, skipping one. Examples 8 and 9 (local calendar, evergreen tutorials) score lowest because their briefs require domain-specific framing that generic blog prompts rarely produce.

**Dominant failure pattern: over-elaboration**
Models introduce extra H3 sub-sections not in the reference outline and frequently exceed 800 words, which the structure judge penalises.

**Model gap is negligible**
gpt-4o-mini and gpt-4o differ by ≤3 points per dimension. Prompt quality, not model capability, is the primary quality driver at this performance level.

---

## Limitations

- **Small dataset (n=10):** One outlier shifts the mean by ~5 points; conclusions are directional, not statistically robust.
- **Self-evaluation bias:** The judge model (`gpt-4o-mini`) evaluates the same model's outputs, which may inflate scores.
- **No human baseline:** Scores reflect the judge's rubric interpretation, not expert human ratings.
- **Single domain:** All examples are AI-writing topics; generalisability to other domains is unknown.

---

## Recommendation

Add a **conciseness evaluator** that counts H2 sections and flags posts exceeding 850 words or 4 H2 sections. This directly targets the over-elaboration failure — the most frequent and consistent quality gap across both models and all 10 examples.
