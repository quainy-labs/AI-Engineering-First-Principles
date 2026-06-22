# Chapter 38: Observability, Traces, Cost, and Latency

Chapter 37 created release gates.

This chapter shows what happens in real use.

Evals test known cases. Observability shows actual requests, retrieved context, model calls, tool calls, failures, cost, latency, and feedback.

Without observability, production AI becomes rumor-driven debugging.

## Concept Overview

Observability answers:

> What happened inside the system, and why?

AI observability covers:

- user request;
- workflow state;
- retrieval;
- context assembly;
- model call;
- tool call;
- validation;
- approval;
- final response;
- cost;
- latency;
- errors;
- feedback.

The first principle:

> A final answer log is not an AI trace.

AI systems need step-level traces because failures can come from retrieval, prompt, context, model, tools, permissions, approvals, or routing.

## Why It Exists

Users complain:

```text
Assistant is slow and sometimes wrong.
```

You open logs. They show:

```text
POST /chat 200
```

This tells almost nothing.

You do not know:

- which model ran;
- which prompt version ran;
- what documents were retrieved;
- which sources entered context;
- how many tokens were used;
- whether validation failed;
- whether fallback ran;
- which tool was called;
- where latency happened;
- how much request cost;
- whether user gave bad feedback.

Bad logging:

- store final answer only;
- ignore retrieved sources;
- ignore prompt/context version;
- ignore tool calls;
- ignore token counts;
- ignore latency by step;
- ignore user role and permissions;
- ignore failure categories.

Without traces, debugging becomes archaeology.

## Mental Model

Think of observability as flight recorder for AI systems.

When something fails, you need timeline, inputs, decisions, system versions, and outputs.

Mental model:

```text
request
-> trace ID
-> step events
-> metrics
-> logs with redaction
-> feedback link
-> eval/regression case
```

The trace should explain:

```text
what system saw
what it retrieved
what it sent to model
what model returned
what tools were proposed/executed
what validations ran
why it stopped
how much it cost
how long it took
```

Observability closes the loop between production and evaluation.

## Internal Mechanics

AI traces should be structured.

### Trace Identity

Every request needs identifiers:

```text
request_id
trace_id
workflow_id
user_id or anonymized user
user_role
tenant/workspace
session_id
```

IDs let feedback, errors, costs, and eval cases connect.

### Version Fields

Store versions:

```text
system_version
prompt_version
model_version
retriever_version
index_version
corpus_version
tool_registry_version
workflow_version
context_policy_version
```

If answer changes, versions tell what changed.

### Retrieval Trace

Retrieval trace should show:

```text
query
filters
retrieved_sources
scores
reranked_sources
forbidden sources excluded
final evidence packet
```

Do not log raw sensitive content unless policy allows it. Store references or redacted snippets when needed.

### Model Trace

Model trace should include:

```text
model
input token count
output token count
context sections
temperature/settings if relevant
structured output validity
latency
cost
```

Raw prompts may contain sensitive data. Use redaction, sampling, or access controls.

### Tool Trace

Tool trace should include:

```text
tool_name
tool_version
arguments summary
permission check
risk level
approval status
result status
error
idempotency key
latency
```

For high-risk tools, trace must support audit.

### Validation and Safety Trace

Trace validation:

```text
schema validation
citation validation
permission check
policy check
refusal rule
human approval
fallback decision
```

This tells whether defenses worked.

### Cost Metrics

Track cost per request and per workflow.

Include:

- model input tokens;
- model output tokens;
- embedding calls;
- reranker calls;
- tool/API cost;
- storage or retrieval cost when meaningful;
- retries;
- fallback models;
- human review when measuring operations.

Cost per successful task is better than raw token cost.

### Latency Metrics

Measure step-level latency:

```text
retrieval
reranking
context assembly
model generation
tool calls
validation
approval wait
UI delivery
```

Total latency is not enough. Need know where time went.

Latency budget example:

```text
Total target: < 5s
Retrieval: < 500ms
Rerank: < 700ms
Model generation: < 3s
Validation: < 200ms
UI overhead: < 600ms
```

### Redaction and Access

Observability can leak data if careless.

Define:

```text
What is logged raw?
What is redacted?
Who can view traces?
How long are traces retained?
Can users request deletion?
Are prompts/outputs sensitive?
```

Trace access should match data sensitivity.

## Real-World Example

User reports:

```text
Assistant gave stale refund policy.
```

Bad logs:

```text
POST /chat 200
```

Useful trace:

```text
trace_id: tr_123
user_role: support_agent
prompt_version: refund_answer_v4
model_version: model_x
retriever_version: hybrid_v2
index_version: support_docs_2026_06_20
retrieved_sources:
- Refund Policy 2024 archived
- Refund Policy 2026 active
context_order:
- archived source first
- active source second
answer_citation: Refund Policy 2026
unsupported_claim: used archived 60-day rule
latency: 4.8s
cost: $0.04
feedback: stale answer
```

Diagnosis:

```text
context ordering problem + stale source not filtered
```

Fix:

- filter archived sources;
- add stale-source eval case;
- update context policy;
- rerun eval.

Observability made debugging concrete.

## Common Mistakes and Failure Modes

Mistake 1: Logging final answer only

Final answer cannot explain retrieval, context, validation, tool, or routing failure.

Mistake 2: Sensitive data logged raw

Prompts, retrieved context, screenshots, audio transcripts, and tool args may contain private data.

Mistake 3: No version fields

Without versions, behavior changes cannot be explained.

Mistake 4: Latency measured only total

Total latency does not show whether retrieval, model, tool, or UI is slow.

Mistake 5: Cost hidden until bill arrives

Cost should be tracked by task, route, model, and retry path.

Mistake 6: Feedback not linked to trace

Bad feedback without trace cannot guide fixes.

Mistake 7: Trace inaccessible to builders

Useful observability must be available to people fixing system, with right access controls.

Mistake 8: No failure categories

Uncategorized errors do not improve evals or product decisions.

## System Design Decisions

Define trace schema:

```text
request identifiers
user/workspace role
system versions
retrieval details
context details
model calls
tool calls
validation
approval
final status
cost
latency
feedback
```

Define redaction:

```text
PII
secrets
customer data
tool arguments
retrieved context
audio/image content
```

Define dashboards:

```text
p50/p95 latency
cost per task
failure rate by type
model call count
tool error rate
retrieval miss rate
fallback rate
user feedback rate
```

Define alerts:

```text
latency spike
cost spike
tool failure spike
invalid output spike
forbidden-source exposure
approval bypass
provider outage
```

Define retention and access:

```text
who can view traces
how long traces stay
what is sampled
what is redacted
what can be exported
```

Observability must support debugging without creating data leaks.

## Build Artifact

Create `observability-and-cost-plan.md`.

Use this template:

```text
Observability and Cost Plan

System:
Main workflows:

Trace Fields:
- field:
  purpose:
  sensitive: yes/no
  redaction:

Version Fields:
- field:
  source:

Step Latency:
- step:
  target:
  alert threshold:

Cost Tracking:
- cost item:
  how measured:
  owner:

Dashboards:
- metric:
  audience:
  decision supported:

Alerts:
- condition:
  severity:
  owner:

Redaction Rules:
- data type:
  rule:

Retention and Access:
- trace type:
  retention:
  viewer roles:

Feedback Link:
- feedback field:
  trace field:
```

Example:

```text
Trace field:
retrieved_source_ids
purpose: debug grounding failures
sensitive: no if IDs only
redaction: do not store full source text by default

Alert:
forbidden_source_rate > 0
severity: critical
owner: AI platform + security
```

Artifact complete when production issue can be traced from user feedback to root cause without exposing unnecessary sensitive data.

## Active Recall Questions

1. Why is final answer log insufficient?
2. What should an AI trace include?
3. Why store prompt, model, retriever, index, and tool registry versions?
4. Why track latency by step?
5. Why track cost per successful task?
6. How do observability and evals connect?
7. What sensitive data can traces expose?
8. What alerts matter for production AI?

## Summary

Observability makes production AI debuggable. It captures traces across retrieval, context, model calls, tools, validation, approvals, final output, cost, latency, errors, and feedback.

Good traces are structured, versioned, redacted, access-controlled, and linked to evals. Cost and latency are product metrics, not afterthoughts.

The most important rule:

> If you cannot trace it, you cannot reliably fix it.

## Preview of Next Chapter

Chapter 38 made cost and latency visible.

Chapter 39 reduces them through caching, streaming, batching, and inference optimization.

Next, you will learn how to improve speed and cost without breaking correctness, privacy, freshness, or safety.
