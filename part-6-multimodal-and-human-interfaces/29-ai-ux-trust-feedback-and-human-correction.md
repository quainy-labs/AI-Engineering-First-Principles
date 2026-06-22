# Chapter 29: AI UX, Trust, Feedback, and Human Correction

Chapters 26-28 expanded inputs beyond text:

```text
documents
-> images
-> audio
-> realtime interaction
```

This chapter brings those AI outputs back to the human user.

Before tools and agents, design the interface. If users cannot inspect, correct, approve, deny, or stop the system, automation becomes risk.

AI UX is not only a chat box. It is the control surface between uncertain AI output and human judgment.

## Concept Overview

AI UX defines how users understand and control AI-assisted work.

It answers:

```text
What did the system understand?
What evidence did it use?
What is it uncertain about?
What output did it produce?
What action does it propose?
What risk exists?
What can the user correct?
What can the user approve or deny?
Can the user stop or undo the work?
How does feedback improve the system?
```

Good AI UX makes uncertainty, evidence, and control visible.

The first principle:

> User should approve concrete outcomes, not vague AI intent.

Bad approval:

```text
Approve assistant?
```

Good approval:

```text
Approve creating task "Follow up on API key access" for ticket T-104?
Tool: create_task
Arguments: ticket_id=T-104, title="Follow up on API key access"
Reason: customer still blocked
```

The user should know what will happen before it happens.

## Why It Exists

A user asks:

```text
Handle this customer issue.
```

The assistant:

- reads the ticket;
- retrieves policy;
- drafts a reply;
- suggests a refund review;
- prepares a task;
- may call tools later.

The user needs to know:

- what the system understood;
- what evidence it used;
- what it plans to do;
- what will happen if they approve;
- how to correct it;
- how to stop it.

Bad interface:

- single chat box;
- hidden interpretation;
- hidden sources;
- hidden tool arguments;
- vague confidence;
- approve button without consequences;
- no edit path;
- no undo path;
- no trace.

This feels magical until it fails.

This chapter exists because AI systems become safer and more useful when users can guide them at the right moments.

## Mental Model

Think of AI UX as a dashboard for uncertainty and control.

The system may interpret, extract, retrieve, draft, classify, or propose. The user should see enough to decide whether the system is right.

The mental model:

```text
input
-> system interpretation
-> evidence
-> AI output
-> proposed action
-> user correction or approval
-> execution
-> trace and feedback
```

The interface should expose:

- input;
- interpretation;
- evidence;
- proposed output;
- proposed action;
- risk;
- user controls;
- status;
- trace.

The user does not need every internal detail. They need the right details for trust and control.

## Internal Mechanics

AI interfaces need interaction patterns.

### Autocomplete

Use when the user remains the author.

Examples:

- writing assistance;
- code completion;
- email phrase suggestions;
- search query suggestion.

Risk is lower because user chooses whether to accept the suggestion.

### Draft and Edit

Use when the model proposes text, but user owns final output.

Examples:

- support reply draft;
- report summary;
- customer update;
- meeting recap.

The interface should allow:

- edit;
- regenerate;
- inspect evidence;
- compare versions;
- mark wrong tone or missing fact.

### Extract and Correct

Use when model converts messy input into structured fields.

Examples:

- invoice extraction;
- call summary fields;
- screenshot triage;
- ticket classification.

The interface should show field-level evidence:

```text
field: invoice_total
value: 1296.00
evidence: page 1, total row
confidence: medium
user action: accept / edit / mark missing
```

Corrections should be stored and used for evals.

### Answer and Cite

Use when model answers from evidence.

The interface should show:

- answer;
- citations;
- source title;
- source freshness;
- limitation or refusal;
- report issue path.

User should be able to inspect source, not only read answer.

### Plan and Approve

Use when AI proposes action before execution.

Plan should show:

- steps;
- tools;
- arguments;
- side effects;
- risk;
- required approval;
- rollback or undo availability.

For high-risk workflows, approval must be specific.

### Trace and Status

Use when work has multiple steps or runs in background.

Show:

- current step;
- completed steps;
- waiting states;
- errors;
- retry state;
- human approval state;
- final result.

Long-running AI work without status feels broken even when it is working.

### Stop and Undo

Users need control.

Controls may include:

- stop;
- cancel;
- deny;
- retry;
- edit;
- escalate;
- undo when possible.

Some actions cannot be undone. For those, approval must be stronger before execution.

### Feedback

Feedback should be actionable.

Weak feedback:

```text
thumbs down
```

Better feedback:

```text
wrong extraction
bad source
missing evidence
unsafe action
wrong tone
should have refused
outdated information
```

Feedback should include trace ID, input, output, sources, model route, and user correction when possible.

## Real-World Example

Use case:

```text
Support issue assistant
```

User says:

```text
Handle this customer issue.
```

Good interface shows:

```text
Interpretation:
Customer reports API key access issue.

Evidence:
- Ticket T-104
- Enterprise Workspace API Key Management, updated 2026-05-10

Draft reply:
Explains how admin can rotate API key.

Proposed action:
Create internal follow-up task.
Tool: create_task
Arguments:
- ticket_id: T-104
- title: Follow up on API key access
- priority: normal

Risk:
Low. Creates internal task only. No customer message sent.

Controls:
Edit draft
Approve task
Deny task
Ask for more evidence
Escalate to human
```

Bad interface:

```text
Done.
```

The good interface creates trust because the user can inspect, correct, and approve concrete work.

## Common Mistakes and Failure Modes

Mistake 1: Chat-only interface for workflow tasks

Chat is weak when users need evidence, fields, actions, status, and approval.

Mistake 2: Hidden interpretation

If the system misunderstood the user, the user should see that before action happens.

Mistake 3: Hidden evidence

Users cannot trust or correct grounded answers without source visibility.

Mistake 4: Vague approvals

"Approve" should not hide tool name, arguments, side effects, or target resource.

Mistake 5: No field-level correction

Users need to fix specific extracted fields, not only reject the whole output.

Mistake 6: No progress state

Long-running jobs need pending, processing, waiting, failed, complete, and cancelled states.

Mistake 7: Final action differs from approved action

The executed action must match what user approved. If arguments change, approval should reset.

Mistake 8: Corrections do not improve system

User corrections should feed evals, review queues, and data improvement.

## System Design Decisions

Choose interaction pattern:

| Pattern | Use When |
|---|---|
| Autocomplete | User remains author |
| Draft + edit | Model proposes text |
| Extract + correct | Model structures messy input |
| Answer + cite | Model explains from sources |
| Plan + approve | Model prepares action |
| Trace/status | Multi-step work needs inspection |

Define what system shows:

```text
interpretation
evidence
draft/output
proposed action
tool arguments
risk
status
trace
```

Define controls:

```text
edit
retry
approve
deny
ask for evidence
escalate
stop
cancel
undo when possible
```

Define approval policy:

```text
Which actions need approval?
What exact text is shown?
What arguments are shown?
When does approval expire?
What happens if arguments change?
What actions cannot be undone?
```

Define correction policy:

```text
What can user correct?
Are corrections field-level?
Are corrections stored?
Do corrections update evals?
Who reviews repeated corrections?
```

Define trust signals:

```text
source title
last updated
evidence snippet
confidence or uncertainty
limitation
human review status
trace ID
```

The interface should make safe behavior easier than blind trust.

## Build Artifact

Create `ai-interface-spec.md` for one AI workflow.

Use this template:

```text
AI Interface Spec

Workflow:
Primary user:
User intent:
Risk level:

Interaction Pattern:
- pattern:
  reason:

System Shows:
- interpretation:
- evidence:
- draft/output:
- proposed action:
- risk:
- trace/status:

User Controls:
- control:
  available when:
  effect:

Approval Design:
- action:
  approval copy:
  tool:
  arguments shown:
  side effect:
  undo possible:
  approval expires:

Correction Path:
- field/output:
  correction method:
  stored where:
  becomes eval: yes/no

Stop/Undo Path:
- workflow state:
  stop behavior:
  undo behavior:

Feedback:
- category:
  review owner:
  next action:

Interface Test Cases:
- case:
  expected user-visible behavior:
```

Example:

```text
Workflow: Support task assistant

Interaction Pattern:
plan + approve

Approval Copy:
Approve creating task "Follow up on API key access" for ticket T-104?

Tool:
create_task

Arguments shown:
ticket_id=T-104
title=Follow up on API key access
priority=normal

Undo possible:
yes, task can be cancelled

Correction:
user can edit title and priority before approval
correction stored with trace and used in eval review
```

Artifact complete when designer and engineer can build user controls without guessing what system should reveal or hide.

## Active Recall Questions

1. Why is chat not always enough for AI workflows?
2. What should user see before approving an AI action?
3. Why expose system interpretation?
4. Why expose evidence and source metadata?
5. What makes approval concrete?
6. Why does field-level correction matter?
7. What status should long-running AI work show?
8. How can user corrections improve future system behavior?

## Summary

AI UX is the control surface between uncertain AI output and human judgment. Good interfaces expose interpretation, evidence, output, proposed action, risk, status, and trace. They give users controls to edit, retry, approve, deny, stop, undo, escalate, and correct.

Before tools and agents, users need to understand what system plans to do and what will happen if they approve.

The most important rule:

> Do not hide uncertainty, evidence, or side effects behind a chat box.

## Preview of Next Chapter

Part 6 expanded inputs and designed human control.

Part 7 connects AI systems to tools, workflows, and agents.

Next, Chapter 30 introduces tool use and API actions. This matters because once users can inspect, correct, and approve, AI systems can safely move from reading and drafting into proposing structured actions that software validates and executes.
