# Chapter 2: When Not to Use AI

Chapter 1 defined AI engineering as system design.

This chapter adds the first discipline of good system design:

> Do not use AI where simpler, safer software is enough.

Good AI engineers are not people who put AI everywhere. They are people who know where AI belongs and where it does not.

## Concept Overview

"When not to use AI" means separating work into the right owners:

```text
rules -> code
facts -> database or trusted source
search -> retrieval
messy language -> model
risky judgment -> human
```

AI is useful when the input is messy, ambiguous, language-heavy, or hard to express as simple rules.

AI is usually the wrong tool when the task needs:

- exact calculation;
- strict policy enforcement;
- database truth;
- permission decisions;
- repeatable deterministic output;
- low-cost high-volume routing;
- legal or regulatory audit;
- a simple form, filter, or rule.

The goal is not to avoid AI.

The goal is to use AI only where it improves the system.

## Why It Exists

Teams often reach for AI because the user input is messy.

But messy input does not mean every decision should be made by a model.

Example:

```text
Customer: I bought this 12 days ago and it arrived broken. Can I get a refund?
```

The message is messy language.

But refund eligibility may depend on exact rules:

- purchase date;
- delivery date;
- item category;
- refund window;
- payment status;
- prior refund history;
- policy exceptions.

These are not "model judgment" problems.

They are data and policy problems.

AI may help extract the customer's reason:

```json
{
  "reason": "item arrived broken",
  "requested_action": "refund"
}
```

But deterministic code should check:

```text
order_date <= refund_window
payment_status = paid
item_is_refundable = true
```

When AI is used in the wrong place, the system becomes:

- inconsistent;
- expensive;
- hard to audit;
- hard to debug;
- risky.

This chapter exists to build refusal muscle.

## Mental Model

Use the AI necessity ladder.

Before adding a model, climb from simplest to most flexible:

```text
1. Can a rule solve it?
2. Can better UX or a form solve it?
3. Can database lookup solve it?
4. Can search/retrieval solve it?
5. Can a small classifier or extractor solve it?
6. Does a language model add real value?
7. Does a tool-using workflow need model judgment?
8. Does a risky step require human approval?
```

Prefer the lowest reliable rung.

Another way to remember:

```text
certainty -> software
knowledge -> retrieval/database
language ambiguity -> model
risk/accountability -> human
```

Good systems often combine all four.

## Internal Mechanics

To decide whether AI belongs, inspect the task.

### Exact Tasks Belong to Code

Use deterministic code for:

- date comparison;
- arithmetic;
- policy windows;
- enum validation;
- permission checks;
- routing by known field;
- formatting;
- duplicate detection by ID.

Example:

```text
refund_window_days = 30
days_since_purchase = today - purchase_date
eligible = days_since_purchase <= refund_window_days
```

This should not be a model prompt.

### Truth Belongs to Sources

If answer depends on current data, fetch it.

Use:

- database;
- API;
- approved document;
- file store;
- audit log.

Do not ask model to remember:

- account status;
- order amount;
- current policy;
- user permissions;
- customer history.

### Messy Language Can Belong to Models

Models are useful when input is unstructured:

- customer email;
- support ticket;
- meeting notes;
- document paragraph;
- user request;
- messy explanation.

Useful model tasks:

- classify intent;
- extract fields;
- summarize;
- rewrite;
- draft;
- detect ambiguity;
- suggest next step.

### Risk Belongs to Human or Policy

If a decision can affect money, access, legal exposure, safety, or customer trust, the model should not silently decide.

Use:

- policy checks;
- approval gates;
- escalation;
- audit logs.

### AI Can Sit Between Messy Input and Deterministic Workflow

Good pattern:

```text
messy text
-> AI extraction/classification
-> validated schema
-> deterministic policy/code
-> human approval if risky
```

Bad pattern:

```text
messy text
-> AI decides everything
```

## Real-World Example

A team says:

```text
We need an AI refund agent.
```

Current workflow:

1. Customer submits refund request.
2. System checks order date.
3. System checks refund policy.
4. System checks payment status.
5. If eligible, refund can be approved.
6. If unclear, human reviews.

The request sounds like AI because the customer message is messy.

But most of the workflow is deterministic.

Break it down:

| Task | Best Owner | Why |
|---|---|---|
| Understand customer reason | Model | messy language |
| Check order date | Code/database | exact fact |
| Check refund window | Code/policy | exact rule |
| Calculate refund amount | Code | arithmetic |
| Detect policy edge case | Code + model signal | rule plus ambiguity |
| Approve unclear refund | Human | risk and judgment |
| Draft customer explanation | Model | language generation |

Better design:

```text
customer message
-> model extracts reason and requested action
-> code fetches order/payment facts
-> code applies refund policy
-> human reviews unclear/high-risk cases
-> model drafts explanation from decision
```

The model helps.

It does not own eligibility.

## Common Mistakes and Failure Modes

### Mistake 1: Using Model for Policy Enforcement

Bad:

```text
Ask model if refund is allowed.
```

Better:

```text
Code applies refund policy. Model drafts explanation.
```

### Mistake 2: Using Model for Arithmetic

Bad:

```text
Ask model to calculate refund amount from invoice.
```

Better:

```text
Extract fields if needed, then calculate in code.
```

### Mistake 3: Asking Model for Private Facts

Bad:

```text
Does this customer have Pro plan?
```

Better:

```text
Fetch plan from account API.
```

### Mistake 4: Replacing Bad UX With AI Too Early

If a simple form can solve the problem, use the form.

AI should not hide poor product design.

### Mistake 5: Using AI Because It Sounds Better

"AI-powered" is not a system requirement.

The requirement is improvement:

- faster;
- more accurate;
- safer;
- cheaper;
- easier to use.

### Mistake 6: Avoiding AI Where Language Actually Hurts Users

Underuse exists too.

Do not force users into rigid forms when natural language would reduce friction.

Do not make humans repeatedly classify messy tickets if a model can extract fields and route to review.

Good engineering is precise, not anti-AI.

## System Design Decisions

For each task in a workflow, decide:

```text
What is the input?
Is the input structured or messy?
Does the task require exact truth?
Does the task require policy enforcement?
Does the task require language understanding?
Can deterministic code solve it?
Would retrieval/search solve it?
Would a model reduce meaningful work?
What happens if model is wrong?
Who approves risky output?
```

Use this ownership map:

| Task Type | Prefer |
|---|---|
| exact rule | deterministic code |
| current fact | database/API |
| trusted knowledge | retrieval/search |
| messy text | model extraction/classification |
| language output | model draft/generation |
| risky decision | human approval |
| audit/accountability | system log |

If model is chosen, write why:

```text
We use AI here because:
- input is unstructured;
- rules cannot capture variation;
- output can be validated;
- failure can be reviewed or corrected.
```

If you cannot explain why AI belongs, do not use it yet.

## Build Artifact

Create `ai-necessity-table.md`.

Template:

```text
Workflow:
User:

Tasks:
- task:
  input:
  best owner: code | database | retrieval | model | human
  reason:
  risk if model owns it:
  validation:
  final decision:

System boundary:
- AI owns:
- Code owns:
- Data/retrieval owns:
- Human owns:
- Out of scope:
```

Worked example:

```text
Workflow: refund request handling
User: support teammate

Tasks:
- task: detect customer reason
  input: customer message
  best owner: model
  reason: messy natural language
  risk if model owns it: may misread reason
  validation: human can correct extracted reason
  final decision: use AI extraction

- task: check refund window
  input: order date, policy window
  best owner: code
  reason: exact date rule
  risk if model owns it: inconsistent eligibility
  validation: unit test policy rule
  final decision: deterministic code

- task: calculate refund amount
  input: payment record
  best owner: code
  reason: arithmetic and source-of-truth data
  risk if model owns it: wrong amount
  validation: compare against payment API
  final decision: deterministic code

- task: draft customer explanation
  input: policy result + customer reason
  best owner: model
  reason: language generation
  risk if model owns it: unsupported claim
  validation: cite decision reason, human review
  final decision: AI draft with review

System boundary:
- AI owns: reason extraction, reply draft
- Code owns: eligibility, amount, validation
- Data/retrieval owns: order facts, refund policy
- Human owns: unclear/high-risk approval
- Out of scope: automatic refund approval for edge cases
```

Acceptance criteria:

- every task has an owner;
- exact decisions are not assigned to model;
- AI use has clear reason;
- risky model output has review path;
- final boundary is explicit.

## Active Recall Questions

1. When is AI weaker than deterministic code?
2. Why should models not own policy enforcement?
3. What does "software for certainty, AI for ambiguity" mean?
4. How can AI support a deterministic workflow without replacing it?
5. What is an AI necessity table?
6. What are examples of AI overuse?
7. What are examples of AI underuse?

## Summary

Good AI engineering includes saying no.

The durable idea:

```text
Use software for certainty.
Use retrieval for trusted knowledge.
Use models for messy language and ambiguity.
Use humans for risky judgment and accountability.
```

AI should not replace deterministic rules, database truth, permission checks, arithmetic, or audit responsibility.

The goal is not less AI or more AI.

The goal is the right responsibility in the right component.

## Preview of Next Chapter

This chapter separated AI work from non-AI work.

Next chapter turns that separation into a system boundary:

> What belongs inside the system, what stays outside, and who owns each decision?
