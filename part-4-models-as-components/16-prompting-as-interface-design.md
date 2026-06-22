# Chapter 16: Prompting as Interface Design

Chapter 15 showed that model behavior depends on context design and generation mechanics:

```text
tokens
-> context sections
-> attention
-> generation
-> output
```

This chapter turns the prompt into an engineering interface.

A prompt is not a spell. It is not a bag of magic words. It is the model-facing part of a contract between software and a probabilistic component.

Good prompting is interface design.

## Concept Overview

An interface defines how two components communicate.

For a model call, the interface includes:

- task definition;
- input fields;
- context sources;
- instructions;
- constraints;
- output schema;
- examples;
- refusal behavior;
- escalation behavior;
- validation rules;
- evaluation cases.

The prompt is one part of that interface. The rest lives in software around the prompt:

- context assembler;
- retriever;
- schema validator;
- tool executor;
- permission checker;
- fallback router;
- eval runner;
- trace logger.

The first principle:

> A production prompt should be designed like an API contract, not written like a motivational note.

Weak prompt:

```text
You are an expert assistant. Be accurate and helpful. Analyze this ticket and respond in JSON.
```

Strong interface:

```text
Task: classify support ticket
Inputs: ticket_text, customer_plan, product_area
Allowed sources: current ticket and supplied account facts
Output schema: issue_type, urgency, summary, missing_fields, needs_human_review
Rules: do not infer account status; mark missing fields explicitly
Validation: issue_type must be enum
Escalation: needs_human_review=true for low confidence or policy exception
```

## Why It Exists

Teams often experience inconsistent model behavior:

- sometimes it returns JSON;
- sometimes it adds explanation;
- sometimes it guesses missing facts;
- sometimes it asks a question;
- sometimes it refuses;
- sometimes it changes labels across similar inputs.

The usual response is:

> We need better prompt engineering.

Maybe. But "better wording" is not enough if the interface is unclear.

A model gives unstable output when the system fails to define:

- what job the model owns;
- what input fields mean;
- which sources are trusted;
- what output shape is required;
- what values are allowed;
- what to do when information is missing;
- what the model must not decide;
- what validation happens after output;
- when to escalate to a human.

This chapter exists so learners stop asking:

> What prompt should I use?

And start asking:

> What model interface does this workflow need?

## Mental Model

Think of a prompt as the function signature and docstring for a probabilistic function.

In normal software, a function has:

```text
name
inputs
types
constraints
return value
errors
tests
```

A model interface needs the same spirit:

```text
model task
input fields
context sources
allowed behavior
disallowed behavior
output contract
failure behavior
eval cases
```

The mental model:

```text
software prepares inputs
-> prompt defines the model interface
-> model produces output
-> validator checks contract
-> router handles failure or escalation
-> system continues workflow
```

Prompting is not isolated. It depends on everything before it:

- information map from Chapter 9;
- ingestion and metadata from Chapter 10;
- schemas from Chapter 11;
- lineage from Chapter 12;
- retrieval from Chapter 13;
- model boundary from Chapter 14;
- context budget from Chapter 15.

A prompt cannot compensate for missing system design. It can only express the interface clearly.

## Internal Mechanics

A strong model interface has several parts.

### Task Definition

The task should be narrow.

Weak:

```text
Analyze this customer issue and tell us what to do.
```

Stronger:

```text
Extract the customer's requested action, issue type, urgency, and missing information from the ticket text.
Do not decide eligibility or approve any action.
```

If one prompt has five hidden jobs, split it.

Better workflow:

```text
extract fields
-> retrieve policy
-> check database facts
-> apply deterministic rules
-> draft response
-> human review if needed
```

### Input Fields

Name the inputs clearly.

Example:

```text
ticket_text:
The raw customer message.

account_facts:
Trusted facts from the account API.

retrieved_policy:
Approved policy snippets with source titles and dates.

workflow_state:
Current approval or job status.
```

This prevents the model from treating every piece of text as equal.

### Source Rules

Tell the model what sources it may use and what sources it must not invent.

Example:

```text
Use account_facts for account-specific truth.
Use retrieved_policy for policy explanation.
Use ticket_text only for the customer's request and wording.
If a required fact is missing, mark it missing.
Do not infer billing status from the ticket text.
```

Source rules turn context into evidence.

### Output Contract

The output should match the consumer.

If software consumes it, use a structured output contract.

Example:

```json
{
  "issue_type": "refund_request | billing_question | login_problem | technical_bug | other",
  "urgency": "low | medium | high",
  "summary": "string",
  "missing_fields": ["string"],
  "needs_human_review": true,
  "evidence": ["string"]
}
```

The prompt should not merely say "respond in JSON." It should define fields, meanings, allowed values, and missing-information behavior.

### Constraints

Constraints define what the model must avoid.

Examples:

```text
Do not invent account facts.
Do not approve refunds.
Do not use documents marked draft.
Do not include internal-only policy in customer-facing output.
Do not call a tool directly.
Do not answer if required evidence is missing.
```

Good constraints are testable.

Weak:

```text
Be safe.
```

Stronger:

```text
If the user asks for account secrets, set needs_human_review=true and do not include the secret.
```

### Refusal and Escalation

A model interface should define what happens when the task cannot be completed safely.

Refusal is appropriate when:

- user asks for unauthorized information;
- required evidence is missing;
- request violates policy;
- requested output is outside the model's role.

Escalation is appropriate when:

- confidence is low;
- policy edge case exists;
- business impact is high;
- human approval is required;
- source conflict cannot be resolved.

Example:

```text
If purchase date is missing, do not decide eligibility.
Set missing_fields=["purchase_date"] and needs_human_review=true.
```

### Examples

Examples teach the interface through cases.

Useful examples include:

- normal valid case;
- missing information;
- ambiguous input;
- should-refuse case;
- should-escalate case;
- malicious instruction inside user text;
- source conflict case.

Examples should match real production inputs. Pretty examples that never occur in the product are weak tests.

### Validation

Validation sits outside the prompt.

The system should check:

- output is parseable;
- required fields exist;
- enum values are allowed;
- evidence maps to input;
- unsupported claims are absent;
- refusal/escalation rules are followed;
- sensitive information is not leaked;
- output is within length and format constraints.

If validation fails, the system should have a path:

```text
retry with validation error
repair
fallback model
ask clarification
human review
safe failure
```

### Evaluation

Prompt changes without eval are guesses.

Evaluate the interface on:

- normal cases;
- messy real inputs;
- edge cases;
- missing-context cases;
- source-conflict cases;
- should-refuse cases;
- should-escalate cases;
- adversarial instruction cases.

Track:

- output validity;
- task success;
- unsupported claim rate;
- refusal accuracy;
- escalation accuracy;
- cost;
- latency;
- regression by case type.

## Real-World Example

Imagine a support ticket classifier.

Weak prompt:

```text
You are an expert support assistant.
Analyze this ticket and return JSON.
Be accurate and helpful.
```

Why it fails:

- "expert" is vague;
- "accurate" is not testable;
- fields are undefined;
- enum labels are missing;
- source rules are missing;
- missing-information behavior is missing;
- validation behavior is missing.

Better model interface:

```text
Task:
Extract support ticket routing fields from ticket_text.

Inputs:
- ticket_text: raw customer message
- customer_plan: trusted account fact
- product_area: trusted account fact if available

Rules:
- Use ticket_text only to understand the customer's issue.
- Use customer_plan and product_area only as supplied facts.
- Do not infer account status, payment status, or refund eligibility.
- If information is missing, add it to missing_fields.

Output:
{
  "issue_type": "refund_request | billing_question | login_problem | technical_bug | account_access | other",
  "urgency": "low | medium | high",
  "summary": "string",
  "missing_fields": ["string"],
  "needs_human_review": true,
  "evidence": ["string"]
}

Escalation:
- needs_human_review=true if issue involves refund, legal, security, account secret, or missing required account facts.
```

Now the interface can be tested.

Example ticket:

```text
I was charged twice after upgrading. Please refund me today.
```

Expected output:

```json
{
  "issue_type": "refund_request",
  "urgency": "high",
  "summary": "Customer reports being charged twice after upgrading and requests a refund today.",
  "missing_fields": ["invoice_id", "duplicate_charge_status"],
  "needs_human_review": true,
  "evidence": ["charged twice", "refund me today"]
}
```

The model does not approve the refund. It extracts and routes the case.

## Common Mistakes and Failure Modes

Mistake 1: Prompt has multiple hidden tasks

One prompt tries to extract, retrieve, decide, draft, and act. Split the workflow.

Mistake 2: "Be accurate" replaces source rules

Accuracy requires trusted sources, evidence, validation, and refusal behavior.

Mistake 3: JSON without schema

JSON syntax does not define field meaning, allowed values, missing behavior, or validation.

Mistake 4: Examples conflict with instructions

The model may follow examples more strongly than abstract rules. Keep examples consistent with the interface.

Mistake 5: Refusal behavior is vague

"Do not answer unsafe requests" is weaker than naming exact conditions and output behavior.

Mistake 6: Prompt tries to enforce permissions

Permissions should be enforced in software before context is assembled and before tools execute.

Mistake 7: No validation outside the prompt

The prompt may ask for structure, but the system must verify structure.

Mistake 8: Prompt changes without regression tests

Every interface change can improve one case and break another. Use the same eval set.

## System Design Decisions

Decide whether prompting is appropriate.

Use a model interface when:

- the task is language-shaped;
- context can be supplied;
- output can be validated;
- errors can be caught or reviewed;
- uncertainty has a safe path.

Do not rely on prompt alone when:

- exact data is required;
- permissions must be enforced;
- policy enforcement is high-stakes;
- action is irreversible;
- output cannot be checked;
- failure would cause unacceptable harm.

Design the interface:

```text
Task:
Caller:
Input fields:
Context sources:
System instruction:
Task instruction:
Output schema:
Source rules:
Forbidden behavior:
Refusal rules:
Escalation rules:
Validation rules:
Examples:
Eval cases:
```

Decide what moves out of the prompt:

```text
Exact calculations -> code
Permission checks -> backend
Current facts -> database/API
Policy source selection -> retrieval/ranking
Irreversible actions -> workflow approval
Output checks -> validator
Model choice -> router
```

The prompt should express the model's job. It should not carry the entire architecture on its back.

## Build Artifact

Create `model-interface-spec.md` for one model call.

Choose a model call such as:

- support ticket classification;
- refund reply drafting;
- policy question answering;
- invoice field extraction;
- document summary;
- tool action proposal.

Use this template:

```text
Model Interface Spec

Workflow:
Model call name:
Caller:
Purpose:

Task:
- what the model should do:
- what the model should not do:

Input Fields:
- name:
  source:
  trusted: yes/no
  meaning:
  required: yes/no

Context Sources:
- source:
  allowed use:
  forbidden use:

Instructions:
- system instruction:
- task instruction:
- source rules:
- style rules:

Output Contract:
- format:
- fields:
- allowed values:
- missing-information behavior:

Refusal and Escalation:
- condition:
  required behavior:

Validation:
- check:
  failure path:

Examples:
- input:
  expected output:
  reason:

Eval Cases:
- case:
  expected behavior:
  metric:
```

Example:

```text
Model call name: TicketClassification
Caller: support intake service
Purpose: classify ticket for routing

Input field:
ticket_text
source: user-submitted ticket
trusted: no
meaning: raw customer issue
required: yes

Context source:
customer_plan
allowed use: routing priority
forbidden use: deciding payment/refund facts

Refusal and Escalation:
if ticket asks for account secret, set needs_human_review=true and do not include secret.

Validation:
issue_type must be one of allowed enum values.
invalid enum -> retry once with validation error, then human review.
```

The artifact is complete when another engineer can implement the model call, validator, and eval cases without guessing the model's role.

## Active Recall Questions

1. Why is prompting better understood as interface design?
2. What belongs in a model interface spec?
3. Why is "be accurate" a weak instruction by itself?
4. What is the difference between output format and output contract?
5. When should logic move from prompt to deterministic code?
6. Why should refusal and escalation behavior be explicit?
7. Why should validation happen outside the prompt?
8. Why should prompt changes be evaluated against the same examples?

## Summary

Prompting is the model-facing interface between software and a language model. A good interface defines task, inputs, context sources, source rules, output contract, forbidden behavior, refusal, escalation, validation, examples, and evaluation.

The prompt alone is not the system. It works with retrieval, context assembly, schemas, validators, permission checks, routers, tools, traces, and evals.

The most important rule:

> Do not ask for better wording when the real problem is an undefined interface.

## Preview of Next Chapter

Chapter 16 defined the model interface.

Chapter 17 asks which model should serve that interface.

Next, you will learn model selection, routing, and fallbacks. This matters because one model for every task is often too slow, expensive, brittle, or weak. Real systems choose models based on task difficulty, quality bar, context needs, latency, cost, privacy, fallback behavior, and human review paths.
