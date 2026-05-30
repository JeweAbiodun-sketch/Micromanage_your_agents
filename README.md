# Micromanage Your Agents

An evaluation and optimization project focused on understanding how agent behavior changes under different prompting and management strategies.

This repository brings together analysis, evaluation summaries, and optimization notes to document the process of improving agent performance in a structured way.

## Project Overview

The goal of this project is to:

- assess agent behavior using evaluation criteria
- identify strengths, weaknesses, and recurring patterns
- test optimization strategies
- compare results before and after changes

## Repository Contents

- `blog_eval_lab/blog_eval_langsmith.ipynb` - main notebook: dataset creation, tracing, evaluation, A/B model comparison
- `evaluation_report.md` - detailed evaluation findings with per-example scores and error analysis
- `evaluation_summary.md` - concise overview of methodology, results, and key findings
- `optimization_summary.md` - cost-performance analysis and model recommendations
- `Screenshots/` - LangSmith experiment screenshots used as evaluation evidence

## What You Will Find Here

- a clear evaluation framework
- observations about agent performance
- optimization ideas and improvements
- written summaries that support reporting or presentation

## How to Use This Repository

1. Open `blog_eval_lab/blog_eval_langsmith.ipynb` to run or review the full lab experiment.
2. Start with `evaluation_summary.md` for a quick understanding of the results.
3. Review `evaluation_report.md` for the full analysis with tables and error patterns.
4. Read `optimization_summary.md` to see cost-performance trade-offs and model recommendations.

### Running the Notebook

```bash
# Install dependencies
pip install langsmith langchain-openai openai python-dotenv

# Set environment variables in blog_eval_lab/.env
OPENAI_API_KEY=sk-...
LANGSMITH_API_KEY=ls__...
LANGSMITH_ENDPOINT=https://eu.api.smith.langchain.com
LANGSMITH_PROJECT=blog_eval_langsmith
```

Open `blog_eval_lab/blog_eval_langsmith.ipynb` using the **Micromanage Agents (.venv)** kernel and run all cells top to bottom.

## Suggested Workflow

If you are continuing work on this project, a practical workflow is:

1. evaluate the current agent behavior
2. document the findings
3. apply one optimization at a time
4. compare outcomes against the original baseline
5. update the summaries with the latest insights

## Notes

- This project is documentation-focused, so the markdown files are the primary source of truth.
- Keep the summaries short, consistent, and easy to scan.
- Update the README whenever the evaluation process or project scope changes.

## Author

Prepared as part of the Micromanage Your Agents project.
