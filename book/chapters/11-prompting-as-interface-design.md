# Chapter 11: Prompting as Interface Design

## Real Problem

A model gives inconsistent outputs.

Sometimes it returns JSON. Sometimes it explains. Sometimes it asks questions. Sometimes it guesses.

Team response:

> We need better prompt engineering.

Maybe. But better wordsmithing is not enough.

Prompt is interface between software and model. Interface must define task, context, constraints, output contract, and failure behavior.

## Bad Default Solution

Bad prompt:

```text
You are an expert assistant. Be accurate and helpful. Analyze this ticket and respond in JSON.
```

Problems:

- "expert" vague;
- "accurate" not testable;
- task underspecified;
- JSON schema missing;
- source rules missing;
- refusal behavior missing;
- tool use missing;
- human review missing.

This is not prompt engineering. This is weak interface design.

## First Principles

An interface defines how two components communicate.

Model interface has:

- input fields;
- instructions;
- context;
- output schema;
- constraints;
- tool affordances;
- error behavior;
- evaluation criteria.

Prompt is one part of this interface.

For production systems, model interface should be written like API contract:

```text
Task:
Inputs:
Allowed sources:
Disallowed behavior:
Output schema:
Validation:
Refusal:
Escalation:
Examples:
Evaluation:
```

## System Decision

Use prompt when:

- task is language-based;
- context can be supplied;
- output can be validated;
- errors are acceptable or reviewable.

Do not rely on prompt alone when:

- exact data needed;
- policy enforcement required;
- permissions needed;
- action is irreversible;
- output cannot be checked;
- stakes are high.

Prompt should not compensate for missing system design.

## Minimal Build

Create `model-interface-spec.md`:

```text
Model task:
Caller:
Input fields:
Context sources:
System instruction:
Task instruction:
Output schema:
Validation rules:
Refusal rules:
Escalation rules:
Examples:
Eval cases:
```

Example:

```text
Model task: classify support ticket
Input fields: ticket_text, customer_plan, product_area
Context sources: current ticket only
Output schema: issue_type, urgency, summary, needs_human_review
Refusal rules: if ticket asks for account secret, mark needs_human_review
Validation: issue_type must be enum
```

## Failure Modes

Prompt/interface fails when:

- task has multiple hidden objectives;
- prompt includes conflicting instructions;
- examples do not match real cases;
- schema is missing;
- model receives irrelevant context;
- refusal rule unclear;
- output validation absent;
- no eval set exists.

Prompt changes without eval are guesses.

## Evaluation

Evaluate model interface using real cases:

- valid normal cases;
- messy inputs;
- edge cases;
- should-refuse cases;
- malicious instruction cases;
- missing context cases.

Measure:

- output validity;
- task success;
- unsupported claims;
- refusal accuracy;
- escalation accuracy;
- cost/latency.

Track every prompt/interface change against same eval set.

## Improvement

Improve model interface in this order:

1. clarify task;
2. remove conflicting goals;
3. improve schema;
4. improve context;
5. add examples;
6. add refusal rules;
7. split task into smaller calls;
8. move logic into deterministic code.

Do not keep stretching one prompt to do five jobs.

## Proof Artifact

Create `model-interface-spec.md`.

Include:

- input contract;
- context sources;
- instructions;
- output schema;
- refusal and escalation rules;
- eval cases.

## Decision Checklist

- [ ] Model task is single-purpose.
- [ ] Inputs are explicit.
- [ ] Context sources are explicit.
- [ ] Output schema exists.
- [ ] Refusal rule exists.
- [ ] Validation exists.
- [ ] Eval set exists.

## Active Recall

1. Why is prompting interface design?
2. What belongs in model interface spec?
3. Why is "be accurate" weak instruction?
4. When should logic move from prompt to code?
5. Why evaluate prompt changes on same examples?

## Next

Next chapter studies failure. Once model interface exists, we need error analysis to improve it.
