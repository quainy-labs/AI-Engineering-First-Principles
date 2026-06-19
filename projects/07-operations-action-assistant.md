# Project 7: Operations Action Assistant

## Build

Build an assistant that can perform bounded operational actions through tools, with validation, approval, state, and audit logs.

This project turns assistant from answer machine into controlled workflow system.

This project belongs after Part 7. It should prove that learner can design the control surface, expose tools safely, orchestrate workflow, manage state, require approval, use traces, and evaluate agents.

## User

Support or operations teammate who needs help completing repeated internal tasks.

## Product Outcome

User asks for action. System:

- understands request;
- shows interpretation and evidence;
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

### 1. Action Interface

Show:

- interpreted intent;
- evidence used;
- proposed tool;
- proposed arguments;
- risk level;
- approve/deny/edit controls;
- workflow status.

### 2. Tool Registry

Each tool has:

- name;
- description;
- input schema;
- risk level;
- permission rule;
- approval requirement.

Include connector metadata:

- owner;
- version;
- allowed workflows;
- auth requirement;
- deprecation status.

### 3. Request Router

Classify request:

- read-only;
- draft-only;
- reversible write;
- risky action;
- unsupported.

### 4. Argument Validator

Reject:

- missing required fields;
- invalid IDs;
- unexpected arguments;
- unauthorized action.

### 5. Approval Gate

Before write action:

```text
Proposed action:
Tool:
Arguments:
Risk:
Evidence:
Approve / Deny
```

### 6. Workflow State

Track:

- current step;
- pending approval;
- completed action;
- failed action;
- final state.

### 7. Computer Use Sandbox

Optional but recommended:

- use browser automation against a mock admin page;
- allow read/fill actions;
- require approval before submit;
- save screenshot trace.

### 8. Agent Trace Review

Evaluate:

- tool sequence;
- arguments;
- approvals;
- stop reason;
- unnecessary steps;
- unsafe near-misses.

### 9. Audit Log

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
-> action interface
-> intent classifier
-> planner/router
-> tool schema validator
-> approval gate
-> tool executor
-> optional browser automation
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
  "risk": "reversible_write",
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
- trace completeness;
- user correction success.
- loop/extra-step rate.

Acceptance:

- zero unsafe actions;
- risky actions require approval;
- approval screen shows exact tool and arguments;
- invalid arguments rejected;
- unsupported requests refused;
- every action traceable.
- trace review catches unsafe near-miss.

## What Learner Understands

- Tools are system interfaces, not magic.
- AI interface is control surface.
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
