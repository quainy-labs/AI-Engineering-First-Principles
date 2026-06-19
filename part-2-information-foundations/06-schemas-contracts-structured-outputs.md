# Chapter 6: Schemas, Contracts, and Structured Outputs

## Real Problem

You ask a model:

> Extract customer name, issue type, urgency, and requested action from this support ticket.

It replies:

> The customer seems upset and probably wants a refund quickly.

This may be useful to a human. It is not useful to software.

Software needs structure:

```json
{
  "customer_name": "Ravi Kumar",
  "issue_type": "refund_request",
  "urgency": "high",
  "requested_action": "refund",
  "confidence": 0.82
}
```

AI engineering often means turning messy language into reliable structure that other systems can validate, store, route, and act on.

## Bad Default Solution

Bad solution:

- ask model for "JSON";
- parse response;
- hope it works;
- crash when model adds explanation;
- silently accept wrong fields.

This creates brittle systems.

Model output must be treated as untrusted input. Even when model follows format most of the time, production system needs validation and fallback.

## First Principles

A schema defines shape of information.

A contract defines expectation between components.

Structured output is model output constrained into machine-readable form.

Example contract:

```text
Input: support ticket text
Output: classification JSON
Required fields:
- issue_type: one of refund_request, bug_report, account_access, billing_question, other
- urgency: one of low, medium, high
- summary: string, max 240 chars
- needs_human_review: boolean
```

Why this matters:

- downstream code can route ticket;
- validation can catch invalid output;
- evaluation can measure field accuracy;
- humans can review edge cases;
- logs become analyzable.

## System Decision

Use structured output when model result feeds another system.

Use free text when output is for human reading only.

Common structured output tasks:

- extraction;
- classification;
- routing;
- scoring;
- tool arguments;
- eval judgments;
- risk labels.

Common free-text tasks:

- explanation;
- draft reply;
- summary for human;
- teaching;
- brainstorming.

When free text leads to action, add structure around it.

## Minimal Build

Create `output-schema.md`:

```text
Task:
Input:
Output consumer:

Schema:
- field:
  type:
  required:
  allowed values:
  validation rule:
  fallback:

Invalid examples:
1.
2.
3.

Retry rule:
Human review rule:
```

Example:

```text
field: refund_eligible
type: boolean
required: true
allowed values: true, false
validation rule: must be based on policy_check result, not model guess
fallback: needs_human_review = true
```

## Failure Modes

Structured output fails when:

- schema is too vague;
- enum values overlap;
- field requires unavailable information;
- model is asked to infer exact database facts;
- invalid output is accepted;
- retries hide deeper ambiguity;
- confidence score is treated as truth;
- downstream code trusts model without validation.

## Evaluation

Build field-level eval.

For 30 examples, label expected output:

```json
{
  "issue_type": "billing_question",
  "urgency": "medium",
  "needs_human_review": false
}
```

Measure:

- valid JSON rate;
- required field completion;
- enum accuracy;
- human-review accuracy;
- unsafe action rate.

Pass only if structured output is valid and correct enough for downstream use.

## Improvement

If output fails:

- simplify schema;
- reduce number of labels;
- add examples;
- add validation;
- separate extraction from decision;
- route uncertain outputs to human;
- use deterministic code after extraction.

Do not fix every schema problem with longer prompt. Sometimes schema design is wrong.

## Proof Artifact

Create `output-schema.md` with:

- schema;
- validation rules;
- invalid examples;
- retry rule;
- human review rule;
- 30-example field-level eval plan.

## Decision Checklist

- [ ] Output consumer is known.
- [ ] Every field has type.
- [ ] Every enum has clear meaning.
- [ ] Validation rules exist.
- [ ] Invalid output path exists.
- [ ] Human review path exists.
- [ ] Field-level eval exists.

## Active Recall

1. What is difference between schema and contract?
2. Why is model output untrusted input?
3. When is free text acceptable?
4. Why should exact policy decisions not come from model-only schema?
5. What metrics measure structured output quality?

## Next

Next chapter moves from structured outputs to finding information. Before semantic retrieval, understand search, ranking, and information architecture.
