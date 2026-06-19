# Chapter 9: What Language Models Actually Do

## Real Problem

A team wants a model to read customer emails and decide:

- what customer wants;
- what policy applies;
- what action should happen;
- what reply should be sent.

This sounds like one task. It is four different tasks.

Some are good model tasks:

- understand messy language;
- summarize issue;
- classify intent;
- draft reply.

Some need deterministic systems or humans:

- check policy source;
- verify account status;
- approve refund;
- send final message.

Language models are useful, but only when used as components inside system.

## Bad Default Solution

Bad solution:

> Ask model to handle whole workflow.

Prompt:

```text
You are a support agent. Read this email, decide refund eligibility, and write reply.
```

Problems:

- model may invent account facts;
- model may misapply policy;
- model may answer without retrieval;
- model may produce different answer on retry;
- model may sound confident while wrong;
- system has no decision boundary.

This is not AI engineering. This is outsourcing system design to model.

## First Principles

A language model maps input text to output text.

It has learned patterns from training data. Given context, it predicts useful continuation. Modern models can show reasoning-like behavior, follow instructions, transform text, use tools, and generate structured outputs.

But model is not:

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
- translation between formats;
- drafting;
- comparing text;
- generating alternatives;
- using provided context.

Model weakness:

- exact facts not in context;
- current private data;
- strict guarantees;
- hidden policy enforcement;
- arithmetic under pressure;
- consistent behavior across all edge cases;
- knowing when it does not know unless system forces it.

## System Decision

For every model use, define boundary:

```text
Input:
Context:
Allowed output:
Not allowed:
Validation:
Fallback:
Human review:
```

Example:

```text
Task: Draft support reply
Input: ticket text + retrieved policy snippets + account facts
Context: only approved docs and live account data
Allowed output: draft reply with citations
Not allowed: final send, refund approval, policy invention
Validation: citations must map to retrieved snippets
Fallback: "needs human review"
Human review: required before sending
```

This turns model from uncontrolled actor into bounded component.

## Minimal Build

Create `model-capability-boundary.md`:

```text
Workflow:
Model task:

Good uses:
- 

Bad uses:
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
- model is asked broad task with no boundary;
- model has no required context;
- model output is not validated;
- model uncertainty is hidden;
- model draft is treated as final answer.

## Evaluation

Build capability test:

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
- add approval gate;
- evaluate edge cases.

If model underperforms:

- improve examples;
- improve context;
- simplify output;
- split task into extraction + deterministic decision + draft.

## Proof Artifact

Create `model-capability-boundary.md`.

Include:

- good model tasks;
- bad model tasks;
- input/context/output;
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

1. Why is model not source of truth?
2. Which tasks are language models good at?
3. Which tasks should deterministic code own?
4. What is model capability boundary?
5. Why should model draft not equal final action?

## Next

Next chapter explains tokens, context, attention, and generation. Not for theory worship. For better system design.
