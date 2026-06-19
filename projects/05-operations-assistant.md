# Project 5: Operations Action Assistant

## Build

Build an assistant that can perform bounded operational actions through tools, with validation, approval, state, and audit logs.

This project turns assistant from answer machine into controlled workflow system.

## User

Support or operations teammate who needs help completing repeated internal tasks.

## Product Outcome

User asks for action. System:

- understands request;
- retrieves context if needed;
- chooses allowed tool;
- validates tool arguments;
- asks approval for risky steps;
- executes safe tool call;
- logs trace;
- stops safely.

## Example Scenario

User:

```text
Create a follow-up task for ticket T-104 because customer still cannot access API keys.
```

Assistant:

1. reads ticket;
2. classifies task;
3. drafts task title/body;
4. asks confirmation;
5. creates task through mock API;
6. logs action.

## Mock Tools

Build local/mock tools:

- `read_ticket(ticket_id)`
- `search_docs(query)`
- `create_task(title, body, ticket_id)`
- `draft_customer_reply(ticket_id, message)`
- `add_internal_note(ticket_id, note)`

Do not include destructive tools in v1.

## Core Features

### 1. Tool Registry

Each tool has:

- name;
- description;
- input schema;
- risk level;
- permission rule;
- approval requirement.

### 2. Request Router

Classify request:

- read-only;
- draft-only;
- reversible write;
- risky action;
- unsupported.

### 3. Argument Validator

Reject:

- missing required fields;
- invalid IDs;
- unexpected arguments;
- unauthorized action.

### 4. Approval Gate

Before write action:

```text
Proposed action:
Tool:
Arguments:
Risk:
Evidence:
Approve / Deny
```

### 5. Workflow State

Track:

- current step;
- pending approval;
- completed action;
- failed action;
- final state.

### 6. Audit Log

Log:

- user request;
- selected tool;
- arguments;
- approval;
- execution result;
- timestamp.

## Architecture

```text
user request
-> intent classifier
-> planner/router
-> tool schema validator
-> approval gate
-> tool executor
-> state store
-> audit log
-> response
```

## Data Model

```json
{
  "trace_id": "TR-001",
  "request": "Create follow-up task for ticket T-104",
  "intent": "create_task",
  "tool_call": {
    "name": "create_task",
    "args": {
      "ticket_id": "T-104",
      "title": "Follow up on API key access"
    }
  },
  "approval_status": "approved",
  "execution_status": "success"
}
```

## Evaluation

Use 25 requests:

- 10 normal;
- 5 missing information;
- 5 risky action;
- 5 should-refuse.

Measure:

- correct tool selection;
- argument validity;
- approval accuracy;
- unsafe action rate;
- stop-condition accuracy;
- trace completeness.

Acceptance:

- zero unsafe actions;
- risky actions require approval;
- invalid arguments rejected;
- unsupported requests refused;
- every action traceable.

## What Learner Understands

- Tools are system interfaces, not magic.
- Model proposes, system validates.
- Approval is product design.
- Agent/workflow trace matters as much as final answer.

## Extension

Add:

- real API integration;
- user roles;
- dry-run mode;
- retry with idempotency key;
- workflow dashboard;
- trace viewer.
