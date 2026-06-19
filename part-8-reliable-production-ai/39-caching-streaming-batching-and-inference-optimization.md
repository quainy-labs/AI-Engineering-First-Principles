# Chapter 39: Caching, Streaming, Batching, and Inference Optimization

Chapter 38 made cost and latency visible. This chapter reduces them.

AI systems often fail product reality not because quality is impossible, but because responses are too slow or too expensive.

## Real Problem

A RAG assistant works well in eval. In production:

- p95 latency is 14 seconds;
- repeated questions cost money every time;
- long documents are reprocessed;
- users abandon responses;
- batch jobs take all night.

Quality passed. System economics failed.

## Bad Default Solution

Bad optimization:

- switch to cheaper model everywhere;
- reduce context blindly;
- stream partial answer without fixing slow retrieval;
- cache unsafe personalized answers;
- batch requests that need realtime response.

Optimization must preserve correctness and safety.

## First Principles

Optimization levers:

- cache stable results;
- stream output for perceived latency;
- batch offline work;
- precompute embeddings;
- use smaller model for easy routes;
- reduce context;
- parallelize independent steps;
- avoid repeated tool calls.

Each lever has tradeoff.

## System Decision

Choose by workload:

| Workload | Optimization |
|---|---|
| repeated public answer | response/cache |
| repeated retrieval | query/result cache |
| document ingestion | batch + precompute |
| long generation | streaming |
| simple classification | smaller model |
| user-specific answer | careful scoped cache |

## Minimal Build

Create `inference-optimization-plan.md`:

```text
System:
Latency budget:
Cost budget:
Cacheable items:
Non-cacheable items:
Streaming use:
Batch jobs:
Precompute:
Model routing:
Invalidation rules:
Safety checks:
```

## Failure Modes

Optimization fails when:

- stale cache returns old policy;
- private answer leaks through cache;
- streaming hides later refusal;
- batching delays urgent work;
- smaller model handles risky cases;
- cost reduction lowers eval pass rate;
- cache invalidation is undefined.

## Evaluation

Measure before/after:

- quality pass rate;
- p50/p95 latency;
- cost per task;
- cache hit rate;
- stale answer rate;
- privacy leak rate;
- user abandonment.

## Proof Artifact

Create `inference-optimization-plan.md`.

## Next

Next chapter covers LLMOps: versioning prompts, models, evals, data, and indexes so changes are explainable.
