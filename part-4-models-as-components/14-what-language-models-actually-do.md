# Chapter 14: What Language Models Actually Do

Part 3 made knowledge usable: documents, schemas, lineage, and search.

Now we decide what a model should do with that knowledge. A language model is not the system. It is one component inside the system.

## Real Problem

A team wants a model to read customer emails and decide:

- what the customer wants;
- what policy applies;
- what action should happen;
- what reply should be sent.

This sounds like one task. It is four different tasks.

Good model tasks:

- understand messy language;
- summarize issue;
- classify intent;
- extract fields;
- draft reply from supplied facts.

Bad model-owned tasks:

- check account truth;
- approve refund;
- enforce permissions;
- decide policy alone;
- send final message.

## Bad Default Solution

Bad solution:

```text
You are a support agent. Read this email, decide refund eligibility, and write reply.
```

Problems:

- model may invent account facts;
- model may use stale policy;
- model may sound confident while wrong;
- model may produce a different answer on retry;
- system has no clear decision boundary.

This is not AI engineering. This is outsourcing system design to the model.

## First Principles

A language model maps input tokens to output tokens.

It has learned patterns from training data. Given context, it predicts useful continuations. Modern models can follow instructions, transform text, produce structured outputs, call tools, and draft natural language.

But a model is not:

- database;
- policy engine;
- permission system;
- calculator;
- audit log;
- source of truth;
- product owner.

Model strength:

- language understanding;
- summarization;
- classification;
- extraction;
- rewriting;
- translation between formats;
- drafting;
- comparing text;
- using provided context.

Model weakness:

- exact private facts not in context;
- current state unless fetched;
- strict guarantees;
- hidden policy enforcement;
- precise arithmetic under pressure;
- stable behavior across every edge case;
- knowing when it does not know unless system forces it.

## System Decision

For every model use, define boundary:

```text
Task:
Input:
Trusted context:
Allowed output:
Not allowed:
Validation:
Fallback:
Human review:
Owner:
```

Example:

```text
Task: Draft support reply
Input: ticket text + retrieved policy snippets + account facts
Trusted context: approved docs and live account API
Allowed output: draft reply with citations
Not allowed: final send, refund approval, policy invention
Validation: citations must map to retrieved snippets
Fallback: "needs human review"
Human review: required before sending
Owner: support workflow service
```

This turns model from uncontrolled actor into bounded component.

## Minimal Build

Create `model-capability-boundary.md`:

```text
Workflow:
Model task:

Good model uses:
-

Bad model uses:
-

Input:
Required context:
Output:
Validation:
Fallback:
Human review:

What deterministic code owns:
-

What database/API owns:
-

What human owns:
-
```

## Failure Modes

Language model use fails when:

- model is asked to know private facts;
- model decides policy;
- model chooses permissions;
- task is broad with no boundary;
- required context is missing;
- output is not validated;
- uncertainty is hidden;
- draft is treated as final action.

## Evaluation

Build capability tests:

| Example | Expected model role | Should refuse? | Human review? |
|---|---|---:|---:|
| Customer asks refund policy | retrieve + draft | no | yes |
| Customer asks account-specific refund | needs account lookup | no | yes |
| Customer asks for admin password | refuse | yes | no |
| Customer asks ambiguous policy edge | escalate | no | yes |

Measure:

- correct task classification;
- refusal accuracy;
- escalation accuracy;
- unsupported claim rate;
- output format validity.

## Improvement

If model overreaches:

- reduce task scope;
- add required context;
- add schema;
- add refusal rule;
- move decision into code;
- add approval gate.

If model underperforms:

- improve examples;
- improve context;
- simplify output;
- split task into extraction, deterministic decision, and draft.

## Proof Artifact

Create `model-capability-boundary.md`.

Include:

- good model tasks;
- bad model tasks;
- input, context, and output;
- validation;
- fallback;
- ownership split.

## Decision Checklist

- [ ] Model task is narrow.
- [ ] Required context is named.
- [ ] Output is constrained.
- [ ] Unsupported facts are forbidden.
- [ ] Deterministic code ownership is clear.
- [ ] Human ownership is clear.
- [ ] Failure cases are evaluated.

## Active Recall

1. Why is a model not a source of truth?
2. Which tasks are language models good at?
3. Which tasks should deterministic code own?
4. What is model capability boundary?
5. Why should model draft not equal final action?

## Next

Next chapter explains tokens, context, attention, and generation. Not for theory worship. For better system design.
