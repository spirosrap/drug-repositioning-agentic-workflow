# Drug Repositioning Agentic Workflow

## Project Overview

Build a multi-agent system that automates drug repurposing research. Given a target gene (e.g., TMEM97) and a disease (e.g., non-alcoholic steatohepatitis), your workflow will:

1. **Mine** biomedical databases for candidate compounds
2. **Route** candidates to appropriate enrichment strategies
3. **Enrich** candidates with literature evidence and safety data
4. **Score & Rank** candidates using iterative refinement
5. **Generate** a validation roadmap for top candidates

## Learning Objectives

This project demonstrates five agentic workflow patterns:

| Pattern | Agent | What You'll Implement |
|---------|-------|----------------------|
| Sequential Workflow | DataMiningAgent | ChEMBL → Open Targets → Merge pipeline |
| Prompt Chaining | LiteratureAgent | PubMed search → LLM assessment |
| LLM-based Routing | RoutingAgent | Dynamic strategy selection per candidate |
| Evaluator-Optimizer | EvaluationAgent | Score → Critique → Refine loop |
| Orchestration | Orchestrator | Coordinate all agents end-to-end |

## Setup

### Requirements

```bash
pip install -r requirements.txt
```

### Conda Environment (Optional)

```bash
conda create -n drug-repositioning python=3.11
conda activate drug-repositioning
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file:

```bash
# For Vocareum (if applicable)
UDACITY_OPENAI_API_KEY=your_key_here

# For local testing
OPENAI_API_KEY=your_key_here

# Optional
ENTREZ_EMAIL=your_email@example.com
```

## Running the Project

Run from the repo root and include `PYTHONPATH=.` so `ls_action_space` imports resolve.

### Mock Mode (Recommended for Development)

No API keys required. Uses deterministic mock data:

```bash
PYTHONPATH=. USE_MOCK_DATA=true python starter/scaffold.py
```

### Live Mode

Requires OpenAI API key. Queries real databases:

```bash
PYTHONPATH=. python starter/scaffold.py
```

### Custom Target/Disease

```bash
PYTHONPATH=. TARGET=SIGMAR1 DISEASE=depression USE_MOCK_DATA=true python starter/scaffold.py
```

## Project Structure

```
starter/scaffold.py          # Main file - implement TODOs here
ls_action_space/             # API helpers + mock/live data access
requirements.txt             # Python dependencies
```

## Your Tasks

Complete **10 TODOs** across 6 agent classes:

| TODO | Agent | Task |
|------|-------|------|
| 1 | ActionPlanningAgent | Write planning prompt |
| 2 | RoutingAgent | Write routing prompt |
| 3 | DataMiningAgent | Implement sequential workflow |
| 4 | LiteratureAgent | Implement prompt chaining |
| 5 | LiteratureAgent | Write assessment prompt |
| 6 | EvaluationAgent | Implement evaluator-optimizer loop |
| 7 | EvaluationAgent | Implement scoring logic |
| 8 | EvaluationAgent | Write critique prompt |
| 9 | EvaluationAgent | Write refinement prompt + apply adjustments |
| 10 | Orchestrator | Implement step execution |

Each TODO includes detailed instructions and hints in the code comments.

## Key Concepts

### Prompt Engineering for JSON Output

Several TODOs require prompts that return structured JSON. Tips:
- Explicitly specify the JSON format you expect
- Use `self._call_llm_json(prompt, fallback)` — it handles parsing and retries
- The `fallback` dict is returned if parsing fails

### Evaluator-Optimizer Pattern

The core insight: **adjustments must persist across iterations**.

```
Iteration 1: Initial scoring from data
Iteration 2: Recalculate totals from adjusted scores (don't rescore!)
Iteration 3: Continue accumulating adjustments...
```

If you call `_score_candidates()` every iteration, you wipe out previous adjustments.

### Routing Strategies

The RoutingAgent chooses strategies based on candidate characteristics:

| Literature Strategy | When to Use |
|--------------------|-------------|
| target_focused | Known MOA, look for target evidence |
| disease_focused | Unknown MOA, look for disease evidence |
| broad_search | Early stage, cast wide net |

| Safety Strategy | When to Use |
|-----------------|-------------|
| comprehensive | Phase 3-4 drugs (FDA labels + FAERS) |
| faers_only | Phase 1-2 drugs |
| basic | Preclinical compounds |

## Expected Output

Successful execution produces:

1. **Console output**: Progress logs, top candidates, validation roadmap
2. **CSV file**: `candidates_{target}_{disease}.csv` with ranked candidates
3. **JSON file**: `audit_{target}_{disease}.json` with full audit trail

### Sample Console Output (Mock Mode)

```
============================================================
DRUG REPURPOSING AGENTIC WORKFLOW
Target: TMEM97 | Disease: non-alcoholic steatohepatitis
Mock mode: True
============================================================


[Planning] Creating action plan...
  Objective: Find repurposable drugs for TMEM97 in non-alcoholic steatohepatitis
  Steps: ['data_mining', 'enrichment', 'scoring', 'roadmap']

[Step 1] Mine databases

[Step 2] Enrich candidates
  Enriching top 5 candidates...

[Step 3] Score and rank

[Step 4] Generate roadmap

============================================================
COMPLETE | LLM calls: 24 | Candidates: 5 (5 enriched)
============================================================

============================================================
TOP CANDIDATES
============================================================

1. PITOLISANT HYDROCHLORIDE (CHEMBL4164059)
   Score: 6.14 | Phase: 4 | pChEMBL: 8.15
   ...
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `EnvironmentError: Set either UDACITY_OPENAI_API_KEY...` | Use mock mode: `USE_MOCK_DATA=true` or set `OPENAI_API_KEY` |
| `ModuleNotFoundError: No module named 'ls_action_space'` | Run from the repo root with `PYTHONPATH=.` |
| Empty candidates list | Check TODO 3 — ensure all three steps execute |
| `candidates_enriched: 0` | Check TODO 10 — ensure `is_enriched = True` is set |
| Scores reset each iteration | Check TODO 6 — only call `_score_candidates` on iteration 1 |
| Loop never exits | Check TODO 8 — ensure approval phrase is detectable |

## Submission

Submit:
1. Completed `starter/scaffold.py`
2. Output files from a successful mock-mode run:
   - `candidates_tmem97_non_alcoholic_steatohepatitis.csv`
   - `audit_tmem97_non_alcoholic_steatohepatitis.json`
3. Screenshot or text capture of a live-mode run
