# Chapter 22: Observability, Cost, and Latency

## Real Problem

Users complain:

> Assistant is slow and sometimes wrong.

You open logs. They show only:

```text
POST /chat 200
```

This tells almost nothing.

AI systems need observability across model calls, retrieval, tool use, cost, latency, errors, and user feedback.

## Bad Default Solution

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

## First Principles

Observability answers:

> What happened inside system, and why?

AI trace should show:

- user request;
- system version;
- model version;
- prompt version;
- retrieval query;
- retrieved sources;
- context size;
- model input/output;
- tool calls;
- validation result;
- approval state;
- final response;
- cost;
- latency;
- errors.

Cost and latency are not afterthoughts. They shape product viability.

## System Decision

Define observability plan:

```text
What do we log?
What do we redact?
Who can view logs?
How long retain?
What metrics go dashboard?
What alerts matter?
How do logs connect to evals?
```

Latency budget example:

```text
Total target: < 5s
Retrieval: < 500ms
Rerank: < 700ms
Model generation: < 3s
Validation: < 200ms
UI overhead: < 600ms
```

Cost budget example:

```text
Target: <$0.05 per support draft
Input tokens:
Output tokens:
Embedding cost:
Rerank cost:
Tool/API cost:
```

## Minimal Build

Create `observability-and-cost-plan.md`:

```text
System:

Trace fields:
- request_id
- user_id or anonymized user
- user_role
- system_version
- prompt_version
- model
- retrieval_query
- retrieved_sources
- context_tokens
- output_tokens
- tool_calls
- validation_result
- approval_status
- final_status
- cost
- latency

Redaction rules:
Dashboards:
Alerts:
Retention:
Access:
```

## Failure Modes

Observability fails when:

- sensitive data logged raw;
- logs omit intermediate steps;
- costs hidden until bill arrives;
- latency measured only total;
- traces cannot connect to user feedback;
- prompt versions not stored;
- failures not categorized;
- logs inaccessible to builders.

## Evaluation

Test observability with failures:

- retrieval miss;
- invalid output;
- tool error;
- permission denial;
- timeout;
- human rejection;
- high cost request;
- prompt injection attempt.

Pass if trace explains what happened without exposing unnecessary sensitive data.

Metrics:

- p50/p95 latency;
- average cost;
- failure rate by type;
- model call count;
- tool error rate;
- retrieval miss rate;
- user feedback rate.

## Improvement

Improve by:

- adding structured traces;
- sampling full payloads safely;
- redacting sensitive fields;
- adding step-level latency;
- adding cost per request;
- connecting feedback to trace;
- turning incidents into eval cases.

## Proof Artifact

Create `observability-and-cost-plan.md`.

Include:

- trace schema;
- redaction rules;
- dashboard metrics;
- alerts;
- latency budget;
- cost budget;
- access and retention rules.

## Decision Checklist

- [ ] Trace captures retrieval.
- [ ] Trace captures model versions.
- [ ] Trace captures tool calls.
- [ ] Cost tracked per request.
- [ ] Latency tracked per step.
- [ ] Sensitive data redacted.
- [ ] Feedback links to trace.

## Active Recall

1. Why is final answer log insufficient?
2. What should AI trace include?
3. Why track latency by step?
4. Why track cost per task?
5. How do observability and evals connect?

## Next

Next chapter covers security, privacy, and risk. Observability shows behavior; risk management controls what system is allowed to do.
