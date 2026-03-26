---
license: apache-2.0
task_categories:
- text-generation
language:
- en
tags:
- diversity
- routing
- llm-evaluation
pretty_name: Diversity Router Data
---

# Diversity Router Data

This dataset supports the paper **"No Single Best Model for Diversity: Learning a Router for Sample Diversity"**. It contains language model generation outputs and diversity metrics across four datasets to calculate the diversity coverage, used to study how different models produce diverse and high quality outputs and to train/evaluate a diversity router.

## Dataset Structure

```
diversity-router-data/
├── simple_questions/               # Questions with fixed answer sets
│   ├── model_data_list_all.json    # Ground-truth answer lists per model per question
│   └── {org}/{model}/
│       └── generations.jsonl       # Model outputs for each question
│
├── novelty_bench/                  # Open-ended Questions
│   └── {model}/
│       └── n_{N}/                  # N = number of API calls per prompt (1, 5, 100)
│           ├── data.jsonl
│           ├── scores.jsonl
│           └── summary.json
│
├── wildchat/                       # Open-ended Questions
│   └── {model}/
│       └── n_{N}/                  # N = 1, 5, 25
│           ├── data.jsonl
│           ├── scores.jsonl
│           └── summary.json
│
└── infinity_chat/                  # Open-ended Questions
    └── {model}/
        └── n_{N}/                  # N = 1, 5, 25
            ├── data.jsonl
            ├── scores.jsonl
            └── summary.json
```

## Benchmarks

| Benchmark | Prompts | Models | N values | Description |
|-----------|---------|--------|----------|-------------|
| `simple_questions` | 23 factual questions | 5 | — | List all valid items (occupations, countries, etc.) |
| `novelty_bench` | 92 curated open-ended questions | 18 | 1, 5, 100 | Enumerate as many open-ended answers as possible |
| `wildchat` | 1000 prompts from WildChat | 18 | 1, 5, 25 | Enumerate as many open-ended answers as possible |
| `infinity_chat` | 1000 samples from InfinityChat | 18 | 1, 5, 25 | Enumerate as many open-ended answers as possible |

**Models included** (novelty_bench / wildchat / infinity_chat):
`gemma-3-1b-it`, `gemma-3-4b-it`, `gemma-3-12b-it`, `gemma-3-27b-it`,
`llama-3.1-8b`, `llama-3.2-1b`, `llama-3.2-3b`, `llama-3.3-70b`,
`olmo-2-0325-32b`, `olmo-2-0425-1b`, `olmo-2-1124-7b`, `olmo-2-1124-13b`,
`qwen2.5-72b-instruct`, `qwen3-0.6b`, `qwen3-1.7b`, `qwen3-4b`, `qwen3-8b`, `qwen3-14b`

**What `n_{N}` means**: Each prompt was sent to the model `N` times independently (temperature=1.0). Larger `N` enables study of how diversity scales with more samples. Results for smaller N are derived from the same max-N run.

---

## File Descriptions

### `scores.jsonl` (primary file)

One JSON record per prompt. Contains the model's generated answers plus all diversity metrics.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique prompt identifier (e.g. `curated-0`, `WildChat-abc123`, `InfChat-abc123`) |
| `prompt` | string | Clean input prompt sent to the model (without format instructions) |
| `model` | string | Model identifier used for generation |
| `generations` | list[str] | Individual answers parsed and extracted from model outputs. Each element is one answer candidate. |
| `generation_indices` | list[int] | For each generation, the index of the raw API call it came from. Since one API call using the "list all" strategy can yield multiple parsed answers, this maps each parsed answer back to its source call. |
| `partition` | list[int] | Cluster ID for each generation. Two generations with the same partition ID are semantically equivalent (duplicates). |
| `distinct` | int | Number of unique partition clusters — the count of semantically distinct answers. |
| `generation_scores` | list[int] | Novelty score per generation: `0` if it is a duplicate of an earlier generation (same partition ID seen before), positive integer (reward model score) if novel. |
| `partition_scores` | list[int] | The score assigned to each generation's partition cluster. Duplicates carry the same cluster score as the first occurrence. |
| `raw_generation_scores` | list[int] | Scores for all answers regardless of duplication. |

### `summary.json`

Aggregate statistics across all prompts in the experiment.

| Field | Type | Description |
|-------|------|-------------|
| `mean_distinct` | float | Average number of distinct answers across all prompts |
| `mean_utility` | float | (metrics from noveltybench paper)Average utility score across all prompts |

### `data.jsonl`

Input dataset copy for the experiment.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique prompt identifier |
| `prompt` | string | Prompt with format instructions appended (used during inference) |



