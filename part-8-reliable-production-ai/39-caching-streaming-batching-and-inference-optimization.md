# Chapter 39: Caching, Streaming, Batching, and Inference Optimization

Chapter 38 made cost and latency visible.

This chapter reduces them.

AI systems often fail product reality not because quality is impossible, but because responses are too slow, too expensive, or too resource-heavy to operate.

Optimization matters, but it must preserve correctness, privacy, freshness, and safety.

## Concept Overview

Inference optimization improves speed, cost, throughput, and user experience.

Common levers:

- caching stable results;
- streaming output;
- batching offline work;
- precomputing embeddings;
- using smaller models for easy routes;
- reducing context;
- parallelizing independent steps;
- avoiding repeated tool calls;
- reusing retrieval results;
- async processing for slow tasks.

The first principle:

> Optimize the bottleneck, not the part that feels expensive.

If retrieval is slow, changing model may not fix latency. If repeated questions dominate cost, caching may help more than model switching. If long context is noisy, better retrieval may improve quality and cost together.

## Why It Exists

A RAG assistant works well in eval.

In production:

- p95 latency is 14 seconds;
- repeated questions cost money every time;
- long documents are reprocessed;
- users abandon responses;
- batch jobs take all night;
- provider rate limits cause queues;
- expensive model handles simple tasks.

Quality passed. System economics failed.

Bad optimization:

- switch to cheaper model everywhere;
- reduce context blindly;
- stream partial answer without fixing slow retrieval;
- cache unsafe personalized answers;
- batch requests that need realtime response;
- skip evals after optimization.

Optimization is not only cost cutting. It is product design under constraints.

## Mental Model

Think of an AI request as a pipeline with costs at each stage.

```text
request
-> retrieval
-> reranking
-> context assembly
-> model call
-> validation
-> tool call
-> response
```

Each stage has:

- latency;
- cost;
- quality impact;
- cacheability;
- privacy risk;
- freshness requirement.

Mental model:

```text
measure
-> find bottleneck
-> choose lever
-> preserve safety
-> rerun evals
-> monitor production
```

Optimization without measurement is guessing.

## Internal Mechanics

Different optimization levers fit different workloads.

### Caching

Caching stores a result for reuse.

Cache candidates:

- public documentation answers;
- retrieval results for common queries;
- embeddings;
- reranker results;
- stable classifications;
- tool reads that are safe and fresh enough;
- document parsing output.

Do not cache blindly.

Cache needs scope:

```text
global
tenant
workspace
user
session
resource
permission scope
```

Unsafe cache:

```text
cache personalized billing answer globally
```

Safe cache:

```text
cache public product doc answer by corpus version + query + language
```

Cache key should include behavior-shaping versions:

```text
query
user/tenant scope
permission scope
corpus version
index version
prompt version
model version
context policy version
```

Invalidation matters.

Invalidate when:

- source document changes;
- index rebuilds;
- permission changes;
- prompt/context policy changes;
- model changes;
- user-specific data changes.

### Streaming

Streaming sends output as it is generated.

Good for:

- long answers;
- drafting;
- summaries;
- user-perceived latency;
- conversational UX.

Streaming does not reduce total computation by itself. It improves perceived responsiveness.

Streaming risk:

- answer starts before validation;
- model begins unsupported claim;
- later refusal conflicts with earlier text;
- partial output visible before safety check.

Use streaming carefully for high-risk or citation-heavy answers.

Pattern:

```text
pre-validate context
stream draft
post-validate final
mark output as draft until complete
```

### Batching

Batching groups work to improve throughput.

Good for:

- document ingestion;
- embedding generation;
- eval runs;
- offline extraction;
- report generation;
- large back-office jobs.

Bad for:

- urgent user-facing response;
- realtime voice;
- approval-sensitive action;
- tasks needing immediate feedback.

Batch jobs need:

- job state;
- retry rules;
- progress;
- failure reporting;
- priority handling.

### Precompute

Precompute work before user asks.

Examples:

- embeddings for new documents;
- summaries for long documents;
- metadata extraction;
- common retrieval indexes;
- eval fixtures;
- cache warmup.

Precompute is useful when workload is predictable and source changes can trigger refresh.

### Model Routing

Use smaller or cheaper models for tasks that meet quality bar.

Examples:

- simple classification;
- format repair;
- low-risk extraction;
- draft rewriting.

Do not route risky tasks to weaker models only for cost.

Route based on evals and risk.

### Context Reduction

Reducing context can improve cost, latency, and quality.

Do not cut randomly.

Better:

- improve retrieval;
- remove duplicate chunks;
- filter stale sources;
- compress long evidence with rules;
- include only needed conversation history;
- use structured facts instead of long text.

Context reduction should preserve answer support.

### Parallelization

Run independent steps in parallel.

Example:

```text
retrieve docs
read account facts
check permission
```

Parallelize only when steps do not depend on each other and do not create unsafe side effects.

## Real-World Example

System:

```text
Support RAG assistant
```

Observed:

```text
p95 latency: 14s
average cost: high
retrieval: 1.2s
reranking: 2.5s
model: 8s
context tokens: too high
```

Bad fix:

```text
switch every route to cheaper model
```

Better plan:

```text
1. deduplicate retrieved chunks
2. reduce top-k from 12 to 6 after eval
3. cache retrieval results for common public doc queries by index version
4. stream long draft replies after context validation
5. route simple ticket classification to smaller model
6. precompute embeddings during document ingestion
7. rerun grounded eval and latency tests
```

Expected outcome:

- lower context tokens;
- faster model calls;
- fewer repeated retrieval costs;
- preserved grounding quality;
- no global cache for user-specific answers.

## Common Mistakes and Failure Modes

Mistake 1: Cheaper model everywhere

Cost drops but quality or safety may regress on hard cases.

Mistake 2: Cache without scope

Private answer can leak across users, tenants, or permission boundaries.

Mistake 3: Cache without invalidation

Stale policy or old index result keeps answering after source changes.

Mistake 4: Streaming unsafe final answers

Partial output may expose unsupported or unvalidated claims.

Mistake 5: Batch realtime tasks

Batching improves throughput but can harm user-facing latency.

Mistake 6: Reduce context blindly

Cutting evidence may reduce grounding and citation quality.

Mistake 7: Optimize without eval rerun

Speed improvement may hide quality regression.

Mistake 8: Ignore user-perceived latency

Streaming, progress states, and async status can improve experience even when total time remains.

## System Design Decisions

Define budgets:

```text
p50 latency
p95 latency
cost per task
throughput
max context tokens
max retries
```

Define cache policy:

```text
what can be cached
cache scope
cache key
TTL
invalidation triggers
privacy checks
freshness checks
```

Define streaming policy:

```text
which outputs can stream
which require pre-validation
how final validation works
how user sees draft vs final
```

Define batching policy:

```text
which jobs batch
priority levels
retry behavior
progress reporting
failure reporting
```

Define routing policy:

```text
which tasks can use smaller model
quality gate
risk exceptions
fallback route
```

Define optimization eval:

```text
quality pass rate
latency before/after
cost before/after
cache hit rate
stale answer rate
privacy leak tests
```

Every optimization should preserve product contract.

## Build Artifact

Create `inference-optimization-plan.md`.

Use this template:

```text
Inference Optimization Plan

System:
Main workflows:

Current Baseline:
- p50 latency:
- p95 latency:
- cost per task:
- top bottlenecks:

Budgets:
- latency:
- cost:
- quality:

Cacheable Items:
- item:
  scope:
  key:
  TTL:
  invalidation:
  safety check:

Non-Cacheable Items:
- item:
  reason:

Streaming:
- output:
  allowed:
  validation rule:

Batch Jobs:
- job:
  priority:
  retry:
  progress reporting:

Precompute:
- artifact:
  trigger:
  invalidation:

Model Routing:
- task:
  primary route:
  cheaper route:
  quality gate:

Eval Before/After:
- metric:
  baseline:
  target:
```

Example:

```text
Cacheable item:
retrieval results for public docs
scope: corpus_version + language + query
TTL: 24 hours
invalidation: index_version changes
safety check: no user-specific/private sources

Non-cacheable item:
billing eligibility answer
reason: user-specific live account data
```

Artifact complete when team can improve speed/cost and prove quality/safety stayed intact.

## Active Recall Questions

1. Why should optimization start with measurement?
2. What makes a cache unsafe?
3. What should be in a cache key for RAG answers?
4. What does streaming improve, and what does it not improve?
5. When is batching useful?
6. Why can context reduction improve quality?
7. Why rerun evals after optimization?
8. Why is cost per successful task better than raw token cost?

## Summary

Optimization improves latency, cost, throughput, and user experience through caching, streaming, batching, precompute, routing, context reduction, and parallelization.

Each lever has tradeoffs. Caches need scope and invalidation. Streaming needs validation. Batching fits offline work. Smaller models need eval gates. Context reduction must preserve evidence.

The most important rule:

> Speed and cost improvements are only wins if correctness, privacy, freshness, and safety remain intact.

## Preview of Next Chapter

Chapter 39 reduced runtime cost and latency.

Chapter 40 controls change through LLMOps.

Next, you will learn versioning for prompts, models, evals, data, indexes, tool schemas, routing rules, and release manifests so production behavior remains explainable and rollback is possible.
