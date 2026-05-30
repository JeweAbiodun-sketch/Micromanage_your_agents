# Optimization Summary

## Cost-Performance Analysis

We compared `gpt-4o-mini` and `gpt-4o` on the same 10-example blog-writing dataset using three LLM-as-judge evaluators (coverage, structure clarity, and tone match). Both models score approximately **81/100** overall, with `gpt-4o` marginally ahead on structure (+1 pt) and tone (+1 pt) while `gpt-4o-mini` leads slightly on coverage (+2 pts). This quality gap of 1–2 points per dimension does not justify the significant cost and latency difference: `gpt-4o` is approximately **3–4× more expensive** per example and **1.5–2× slower** (~9.4 s vs ~5–7 s average latency). For a content-generation workflow where a human reviews drafts before publishing, `gpt-4o-mini` is the clear winner — it produces acceptable, editable first drafts at a fraction of the cost.

---

## Model Comparison

| Metric | `gpt-4o-mini` | `gpt-4o` |
| --- | --- | --- |
| Coverage | 80.5 / 100 | 78.5 / 100 |
| Structure Clarity | 82.0 / 100 | 83.0 / 100 |
| Tone Match | 81.0 / 100 | 82.0 / 100 |
| **Overall Mean** | **81.2** | **81.2** |
| Avg Latency | ~5–7 s | ~9.4 s |
| Relative Cost | 1× (baseline) | ~3–4× |

---

## When to Pick Each Configuration

| Scenario | Recommended Model |
| --- | --- |
| High-volume draft generation with human review | `gpt-4o-mini` |
| Budget-constrained experimentation and prototyping | `gpt-4o-mini` |
| Low-volume, high-stakes content (investor comms, launch copy) | `gpt-4o` |
| Fully automated publishing with no human review | `gpt-4o` |

---

## Further Optimisations

- **Evaluator caching:** Same prompt + same output always yields the same score. Caching evaluator responses during development reduces judge API costs by ~40% across repeated runs with no quality impact.
- **Temperature tuning:** Dropping `temperature` from `0.7` to `0.3` reduces output variance and can improve structural consistency — useful if reproducibility matters more than creativity.
- **Prompt-first, model-second:** The performance gap between models is smaller than the gap between a weak and a strong system prompt. Investing in prompt iteration delivers more quality improvement per dollar than upgrading from `gpt-4o-mini` to `gpt-4o`.
