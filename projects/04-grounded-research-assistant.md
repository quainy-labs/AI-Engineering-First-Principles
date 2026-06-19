# Project 4: Grounded Research Assistant

## Goal

Build assistant that answers from trusted documents with citations.

## Output

Create:

```text
projects/grounded-research-assistant/
  rag-design.md
  context-policy.md
  rag-eval-set.md
  knowledge-product-spec.md
```

## Required Behavior

Assistant must:

- retrieve relevant sources;
- answer only from provided context;
- cite sources;
- refuse when context is insufficient;
- expose uncertainty;
- log failures.

## Evaluation

Use 40 questions:

- 15 easy;
- 15 normal;
- 5 edge;
- 5 should-refuse.

Measure:

- retrieval accuracy;
- answer relevance;
- faithfulness;
- citation accuracy;
- refusal accuracy;
- latency;
- cost per answer.

## Final Artifact

Ship RAG evaluation report and demo.

