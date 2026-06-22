# Chapter 30: Tool Use and API Actions

Part 6 taught how users interact with AI systems: documents, images, audio, correction, trust, and human control.

Now the system crosses a boundary.

Until now, most AI components have read, transformed, extracted, retrieved, summarized, or answered. Tool use lets an AI-assisted system interact with software: read records, create tasks, draft messages, update tickets, schedule meetings, or trigger workflows.

Tool use is where AI stops being only language and starts touching real systems.

## Concept Overview

Tool use means the model can request an external capability through a structured interface.

Example:

```text
User: Create a follow-up task for ticket T-104.
Model proposes: create_task(ticket_id="T-104", title="Follow up on API key access")
System validates: ticket exists, user has permission, schema is valid
Tool executes: task is created
System logs: who, what, why, when
```

The important idea:

> The model proposes. The system decides and executes.

The model should not directly control your database, send emails, issue refunds, delete records, or change permissions. It should produce a structured proposal that code can validate.

## Why It Exists

A useful AI system eventually meets this user request:

```text
Can you do it for me?
```

A support assistant can answer:

```text
Here is the replacement order process.
```

But the user may need:

```text
Create the replacement order.
```

That requires action.

Tool use exists because language alone cannot complete many workflows. Real work often needs APIs:

- read ticket;
- search order;
- create task;
- draft email;
- add internal note;
- check account status;
- schedule meeting;
- start approval request.

But action changes the risk.

A wrong answer is bad.

A wrong tool call can:

- modify customer data;
- charge money;
- send a message;
- expose private records;
- create duplicate work;
- trigger irreversible workflow.

So tool use exists to connect AI to software, but it must be bounded by schema, permissions, validation, approval, and logs.

## Mental Model

Use this map:

```text
user intent
-> model interprets intent
-> model proposes tool call
-> code validates schema
-> policy checks permission and risk
-> human approves if needed
-> tool executes
-> result is returned
-> trace is logged
```

The tool call is not the action yet.

It is a proposal.

Safe tool use separates roles:

| Layer | Owns |
|---|---|
| User | intent and approval |
| Model | interpretation and proposed action |
| Code | schema validation and control flow |
| Policy | permission and risk rules |
| Tool/API | external action |
| Log/trace | accountability |

If the model owns all layers, the system is unsafe.

## Internal Mechanics

A tool has a contract.

At minimum:

```text
Tool name:
Purpose:
Input schema:
Output schema:
Side effects:
Permissions:
Risk level:
Validation:
Approval:
Failure behavior:
Audit log:
```

### Tool Name

Name should describe one capability.

Good:

```text
create_internal_task
draft_customer_reply
read_order_status
```

Risky:

```text
handle_customer
manage_order
do_action
```

Vague names cause vague behavior.

### Input Schema

Schema defines what arguments are allowed.

Example:

```text
ticket_id: string, required, pattern T-[0-9]+
title: string, required, 5-100 chars
priority: enum low | normal | high
```

Schema stops the model from inventing arbitrary fields.

### Permissions

Permission answers:

```text
Is this user allowed to do this action on this resource?
```

Permission must be checked before execution.

Do not ask the model:

```text
Is this user allowed?
```

Use code and policy.

### Side Effects

A side effect changes the world.

Examples:

- creating task;
- sending email;
- issuing refund;
- changing permission;
- deleting record.

Read-only tools are lower risk. Write tools need stronger controls.

### Idempotency

Idempotency means retrying the same request does not duplicate the action.

If the system retries `issue_refund`, it must not issue two refunds.

Use idempotency keys for actions that may be retried:

```text
idempotency_key = ticket_id + action_type + request_id
```

### Tool Result

Tool result should be structured.

Bad:

```text
Done.
```

Better:

```json
{
  "status": "created",
  "task_id": "TASK-482",
  "link": "/tasks/TASK-482"
}
```

Structured results make downstream workflow and logging reliable.

## Real-World Example

Scenario:

A support teammate says:

```text
Create a follow-up task for ticket T-104 because customer still cannot access API keys.
```

Bad system:

```text
Model decides action and directly calls task API.
```

Better system:

```text
1. Model classifies intent: create follow-up task
2. Model proposes tool call:
   create_internal_task(ticket_id="T-104", title="Follow up on API key access")
3. Code validates ticket_id format
4. System checks user can access T-104
5. Risk is reversible_write
6. Approval screen shows exact action
7. User approves
8. Tool creates task
9. Trace logs request, args, approver, result
```

The assistant feels useful because it reduces work.

The system remains safe because the model did not execute alone.

## Common Mistakes and Failure Modes

### Mistake 1: Giving Too Many Tools

If model sees 30 tools, it may choose similar but wrong tool.

Example:

```text
draft_customer_reply
send_customer_email
```

If workflow only needs draft, do not expose send.

### Mistake 2: Vague Tool Descriptions

Bad:

```text
update_ticket: updates ticket
```

Better:

```text
add_internal_note: adds a private support note to an existing ticket. Does not message customer.
```

### Mistake 3: Permissive Schemas

Bad:

```text
args: object
```

Better:

```text
ticket_id: required string
note: required string, max 1000 chars
visibility: enum internal
```

### Mistake 4: Trusting Model for Permission

The model may misunderstand user role or resource access.

Permission belongs in code.

### Mistake 5: Executing Before Approval

If action affects customer, money, permissions, or external communication, approval should be explicit.

### Mistake 6: No Idempotency

Retries can duplicate side effects.

### Mistake 7: Weak Logs

If logs only say:

```text
tool_call_success
```

you cannot audit who approved what.

## System Design Decisions

For each tool, decide:

```text
What user intent does this support?
Is it read-only or write?
What can go wrong?
Who can use it?
What input schema is strict enough?
What validation happens before execution?
Does it need approval?
Is it idempotent?
What should be logged?
What should happen on failure?
```

Risk classification helps:

| Tool Type | Example | Default Control |
|---|---|---|
| Read-only | read ticket | permission check |
| Draft-only | draft reply | user review |
| Reversible write | add internal note | log + maybe approval |
| External send | email customer | approval |
| Financial action | refund | strict approval |
| Permission change | grant access | strict approval |
| Destructive action | delete record | disallow or special flow |

Start with read-only and draft-only tools.

Add write tools only when:

- schema is strict;
- permission checks exist;
- approval path exists;
- logs are complete;
- retry behavior is safe;
- eval cases include unsafe requests.

## Build Artifact

Create `tool-interface-spec.md`.

Template:

```text
Tool name:
Purpose:
User intent it supports:

Input schema:
-

Output schema:
-

Side effects:
Risk level:
Permissions:
Validation before execution:
Idempotency rule:
Approval required:
Failure behavior:
Audit log fields:

Eval cases:
-
```

Worked example:

```text
Tool name: create_internal_task
Purpose: create internal follow-up task linked to support ticket
User intent it supports: "create a follow-up task"

Input schema:
- ticket_id: string, required, pattern T-[0-9]+
- title: string, required, 5-100 chars
- body: string, optional, max 1000 chars
- priority: enum low | normal | high

Output schema:
- status: created | rejected
- task_id: string
- link: string

Side effects: creates task in internal task system
Risk level: reversible_write
Permissions: user must have access to ticket_id
Validation before execution: ticket exists, title valid, priority enum valid
Idempotency rule: request_id + ticket_id + tool_name
Approval required: yes for priority=high, no for normal/low
Failure behavior: return rejected with reason; do not retry writes without idempotency key
Audit log fields: user_id, ticket_id, args, approval_status, task_id, timestamp

Eval cases:
- normal task creation
- missing ticket_id
- invalid ticket_id
- user lacks ticket access
- high priority requires approval
- duplicate retry does not create second task
```

Acceptance criteria:

- every input has type and constraint;
- side effects are named;
- permission check is explicit;
- approval rule is explicit;
- failure behavior is explicit;
- eval cases include unsafe and invalid requests.

## Active Recall Questions

1. Why is a tool call a proposal, not an action?
2. What should model own, and what should code own?
3. Why are write tools riskier than read tools?
4. What is idempotency, and why does retry make it important?
5. What fields belong in a tool interface spec?
6. When should human approval be required?
7. Why is a vague tool name dangerous?

## Summary

Tool use connects AI systems to software capabilities.

The durable idea:

```text
model proposes, system validates, human approves when needed, tool executes, trace records
```

Safe tool use requires:

- strict schema;
- permissions;
- side-effect awareness;
- approval gates;
- idempotency;
- structured results;
- audit logs;
- eval cases.

Do not make the model responsible for safety. Put safety in system design.

## Preview of Next Chapter

This chapter designed one tool safely.

Next chapter asks what happens when the system has many tools across many products.

That requires connectors, MCP-style interfaces, and tool registries.
