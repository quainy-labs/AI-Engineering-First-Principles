# Chapter 11: Schemas, Contracts, and Structured Outputs

Chapter 10 showed how information enters system. This chapter defines how information moves between components reliably.

Software needs structure. AI systems need even more structure because model outputs are probabilistic and untrusted.

## Real Problem

You ask model:

> Extract customer name, issue type, urgency, and requested action from this support ticket.

It replies:

> The customer seems upset and probably wants refund quickly.

Useful to human. Not useful to software.

Software needs:

```json
{
  "customer_name": "Ravi Kumar",
  "issue_type": "refund_request",
  "urgency": "high",
  "requested_action": "refund_review",
  "needs_human_review": true
}
```

## Bad Default Solution

Bad solution:

- ask model for "JSON";
- parse response;
- hope it works;
- crash when model adds explanation;
- silently accept wrong fields.

Model output must be treated as untrusted input.

## First Principles

Schema defines shape of information.

Contract defines expectation between components.

Structured output is model output constrained into machine-readable form.

Contracts exist between:

- UI and API;
- API and worker;
- worker and database;
- retriever and context builder;
- model and validator;
- tool caller and tool executor.

## System Decision

Use schemas for:

- ingestion records;
- document metadata;
- extracted fields;
- eval examples;
- tool arguments;
- model outputs;
- trace logs.

Use free text only when output is for human reading and no downstream system depends on it.

## Minimal Build

Create `schemas.md`:

```text
Entity:
Purpose:
Consumer:

Fields:
- name:
  type:
  required:
  allowed values:
  validation:
  fallback:

Invalid examples:
1.
2.
3.
```

## Failure Modes

Schemas fail when:

- field meaning vague;
- enum values overlap;
- schema requires unavailable info;
- model infers database facts;
- validation missing;
- confidence treated as truth;
- downstream code trusts unvalidated output.

## Evaluation

For 30 examples, measure:

- valid JSON rate;
- required field completion;
- enum accuracy;
- validation failure rate;
- human-review accuracy.

## Improvement

Improve by:

- simplifying schema;
- reducing labels;
- adding validation;
- separating extraction from decision;
- routing uncertainty to human;
- moving exact logic to code.

## Proof Artifact

Create `schemas.md` with schemas for:

- document metadata;
- extracted record;
- eval example;
- trace event.

## Decision Checklist

- [ ] Every important entity has schema.
- [ ] Field types are clear.
- [ ] Validation exists.
- [ ] Invalid output path exists.
- [ ] Human review path exists.
- [ ] Model output is not trusted directly.

## Active Recall

1. What is difference between schema and contract?
2. Why is model output untrusted input?
3. What components need contracts?
4. When is free text acceptable?
5. Why separate extraction from decision?

## Next

Now that information has shape, next chapter keeps it reliable over time: quality, lineage, and versioning.

