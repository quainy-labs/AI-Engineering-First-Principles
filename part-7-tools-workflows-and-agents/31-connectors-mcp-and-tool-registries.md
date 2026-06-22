# Chapter 31: Connectors, MCP, and Tool Registries

Chapter 30 taught safe design for one tool:

```text
model proposes
-> system validates
-> human approves when needed
-> tool executes
-> trace records
```

That works for one API.

Real AI products rarely have one API. They connect to ticketing systems, CRMs, calendars, documents, billing systems, browsers, internal tools, and partner services.

This chapter asks:

> How does an AI system know which tools exist, which tools it may use, and how to call them safely?

## Concept Overview

A connector adapts an external system into safe AI-usable capabilities.

A tool registry is the catalog that tells the AI system which tools exist, who owns them, which workflows may use them, and what controls apply.

MCP-style tool interfaces make tool discovery and invocation more standard across clients and servers.

Together, they answer:

```text
What tools exist?
Where do they come from?
Who owns them?
Which workflow can use them?
Which user role can use them?
What schema do they require?
What risk level do they carry?
What version is active?
What approval is required?
What happens if connector fails?
```

The first principle:

> The model should not see every tool. It should see the right tools for the current workflow, user, and risk.

Tool discovery is not the same as safe execution.

## Why It Exists

Imagine an operations assistant for a support team.

A customer says:

```text
I still cannot access API keys. Please follow up today.
```

To help, the assistant may need to:

- read the ticket;
- search internal docs;
- check customer plan in CRM;
- create follow-up task;
- draft customer reply;
- add internal note;
- check billing status.

At first, the builder hardcodes every tool into one prompt.

It works in demo.

Then company adds:

- new CRM connector;
- new billing connector;
- calendar connector;
- deprecated ticket API;
- stricter permission rules;
- different tools for support agent and support lead.

Now tool use becomes messy.

Bad default:

```text
Here are all company tools. Choose the best one.
```

Problems:

- model sees irrelevant tools;
- high-risk tools appear in low-risk workflows;
- similar tools compete;
- permissions are unclear;
- old tool versions stay visible;
- no owner knows when schema changes;
- eval cannot tell whether failure came from model, registry, connector, or permission layer.

This chapter exists because many-tool systems need architecture, not prompt clutter.

## Mental Model

Think of connectors and registries as the tool supply chain.

The model should not wander through every capability in the company. The system should expose a small allowed tool set based on current workflow.

Mental model:

```text
user request
-> workflow identifies needed capability
-> registry returns allowed tools
-> connector handles external system details
-> model proposes tool call
-> system validates schema and permissions
-> tool executes
-> result is logged
```

Each layer has a job:

| Layer | Job |
|---|---|
| Workflow | Defines kind of work happening |
| Registry | Lists tools allowed for workflow/user/risk |
| Connector | Adapts external system into stable interface |
| Tool schema | Defines allowed inputs and outputs |
| Policy | Checks permission, risk, and approval |
| Trace | Records what happened |

If these layers are mixed together, tool systems become fragile.

## Internal Mechanics

Start with vocabulary.

### API

An API is a capability exposed by software.

Example:

```text
POST /tasks
```

It creates a task in a task system.

### Tool

A tool is the AI-facing wrapper around a capability.

Example:

```text
create_task(title, body, ticket_id)
```

The model sees tool name, description, and input schema. The system maps the tool call to real code.

### Connector

A connector is adapter code between your AI system and another product or service.

Examples:

```text
Ticketing connector -> read_ticket, add_internal_note
CRM connector -> read_customer_plan
Task connector -> create_task
Calendar connector -> schedule_meeting
Billing connector -> read_invoice_status
```

Connector owns:

- external auth;
- API details;
- retries;
- rate limits;
- timeout behavior;
- response normalization;
- external error mapping;
- provider-specific quirks.

The model should not know external API quirks. The connector hides them behind stable tools.

### Tool Registry

A tool registry is a catalog of tools.

It answers:

- what tools exist;
- who owns each tool;
- which connector provides it;
- which workflows may use it;
- which user roles may use it;
- what risk level it has;
- what schema version is active;
- what approval is required;
- whether tool is deprecated.

Registry is not just documentation. It controls exposure.

### MCP-Style Interfaces

MCP-style tool interfaces let tools be exposed and discovered through a standard pattern.

This helps with:

- interoperability;
- discovery;
- consistent tool descriptions;
- separation between client and tool server;
- reuse across environments.

But standard discovery does not solve safety by itself.

System still needs:

- workflow scoping;
- user permissions;
- schema validation;
- risk classification;
- approval gates;
- audit logs;
- eval cases.

MCP can help expose tools. It does not decide which action is safe for this user right now.

### Registry Entries

A useful registry entry includes:

```text
tool name
owner
connector
purpose
allowed workflows
forbidden workflows
input schema
output schema
permissions
risk level
approval rule
version
deprecation status
eval cases
```

Example:

```text
Tool name: create_task
Owner: support-platform
Connector: task-system
Purpose: create internal follow-up task linked to support ticket

Allowed workflows:
- create_follow_up_task
- escalate_support_ticket

Forbidden workflows:
- answer_policy_question
- refund_approval

Input schema:
- title: string, 5-100 chars
- body: string, 0-1000 chars
- ticket_id: string, pattern T-[0-9]+
- priority: enum low | normal | high

Output schema:
- task_id: string
- status: created | rejected
- link: string

Risk level: reversible_write
Approval: required when priority=high
Version: v1
```

### Tool Exposure

Tool exposure should be scoped.

Do not expose:

```text
all company tools
```

Expose:

```text
tools allowed for this workflow + user role + risk level
```

Example:

| Workflow | Allowed Tools | Hidden Tools |
|---|---|---|
| Answer policy question | search_docs | create_task, issue_refund |
| Create follow-up task | read_ticket, create_task | issue_refund, send_email |
| Draft customer reply | read_ticket, search_docs, draft_reply | send_email |
| Refund review | read_ticket, read_billing | issue_refund without approval |

Smaller tool sets reduce wrong-tool calls.

### Versioning and Deprecation

Tools change.

Registry should track:

```text
tool version
schema version
connector version
active/deprecated status
replacement tool
migration date
owner
```

Deprecated tools should not remain visible to model prompts.

### Connector Failure

Connectors fail.

Failure modes:

- API timeout;
- auth expired;
- rate limit;
- provider outage;
- schema mismatch;
- permission denied;
- partial write;
- duplicate retry risk.

Registry and connector should define failure behavior:

```text
retry?
fallback?
queue async job?
show user status?
route to human?
```

## Real-World Example

Use case:

```text
Support operations assistant
```

User role:

```text
support_agent
```

Workflow:

```text
create_follow_up_task
```

Allowed tools:

```text
read_ticket
search_docs
create_task
```

Forbidden tools:

```text
issue_refund
delete_customer
send_customer_email
grant_admin_access
```

User asks:

```text
Create a follow-up task for ticket T-104 because customer still cannot access API keys.
```

Good flow:

```text
workflow = create_follow_up_task
role = support_agent
risk = reversible_write
registry exposes read_ticket and create_task
model proposes create_task
system validates ticket_id
permission checks ticket access
approval shown if needed
connector calls task API
trace logs tool, args, user, result
```

Bad flow:

```text
all tools exposed
model sees send_customer_email and create_task
model sends customer email instead of drafting follow-up
```

Root cause is registry exposure, not only model behavior.

## Common Mistakes and Failure Modes

Mistake 1: Tools exposed globally

Global tool exposure increases wrong-tool selection and high-risk mistakes.

Mistake 2: Registry has no owner

No owner means stale schemas, unclear permissions, and broken tools stay active.

Mistake 3: Tool descriptions are ambiguous

`update_ticket` is vague. `add_internal_note` and `send_customer_reply` are clearer and safer.

Mistake 4: Similar tools exposed together without need

If workflow needs draft only, do not expose send.

Mistake 5: Deprecated tools stay visible

Old tools create inconsistent behavior and hidden failures.

Mistake 6: Connector auth hidden from policy

Tool may appear available even when external auth is expired or user lacks permission.

Mistake 7: MCP discovery treated as safety

Discovery helps find tools. Safety still requires policy, schema, approval, and trace.

Mistake 8: No wrong-tool evals

Eval must include distractor tools that model should not choose.

## System Design Decisions

Design registry governance:

```text
Who can register a tool?
Who owns each tool?
What schema review is required?
How are tools approved?
How are tools deprecated?
```

Design workflow scoping:

```text
Which workflows exist?
Which tools are allowed per workflow?
Which tools are forbidden per workflow?
Which roles can use each tool?
```

Design risk classification:

```text
read_only
draft_only
reversible_write
external_send
financial_action
permission_change
destructive_action
```

Design connector behavior:

```text
How is external auth handled?
What timeout applies?
What retry rule applies?
How are external errors normalized?
What happens during outage?
```

Design versioning:

```text
How are tool versions recorded?
How are schema changes rolled out?
How are deprecated tools hidden?
How are evals tied to versions?
```

Design observability:

```text
Which tools were exposed?
Why were they exposed?
Which tool was selected?
Which connector executed?
What args were used?
What permission check ran?
What result returned?
```

The registry should make tool availability explainable.

## Build Artifact

Create `tool-registry-spec.md` and `tool-selection-eval.md`.

Use this template for `tool-registry-spec.md`:

```text
Tool Registry Spec

Registry owner:
Who can add tools:
Schema review process:
Permission model:
Risk levels:
Approval policy:
Versioning policy:
Deprecation policy:

Connectors:
- connector:
  external system:
  owner:
  auth method:
  failure behavior:

Tool Entries:
- name:
  owner:
  connector:
  purpose:
  allowed workflows:
  forbidden workflows:
  input schema:
  output schema:
  permissions:
  risk level:
  approval:
  version:
  deprecation:
  eval cases:
```

Use this template for `tool-selection-eval.md`:

```text
Tool Selection Eval

Workflow:
User role:
Registry version:

Eval Cases:
- user request:
  workflow:
  user role:
  expected allowed tools:
  expected selected tool:
  forbidden tools:
  expected result:
```

Example eval:

```text
User request:
Tell the customer we are looking into this.

Workflow:
draft_customer_reply

Expected allowed tools:
read_ticket, search_docs, draft_customer_reply

Forbidden tools:
send_customer_email

Expected result:
model drafts reply only; no external send
```

Artifact complete when engineer can expose safe tool sets per workflow and prove model does not choose forbidden distractor tools.

## Active Recall Questions

1. What is difference between API, tool, connector, and registry?
2. Why should model not see every tool?
3. What does MCP-style discovery help with?
4. What does MCP-style discovery not solve?
5. Why should tools be scoped by workflow?
6. What belongs in a registry entry?
7. Why do deprecated tools need to be hidden?
8. Why should tool selection be evaluated separately from final answer?

## Summary

Connectors adapt external systems into stable AI-usable capabilities. Tool registries catalog those tools, scope them by workflow and role, track risk and versions, and control what the model can see.

MCP-style interfaces can standardize discovery and invocation, but safety still needs permission checks, schema validation, approval gates, workflow scoping, observability, and evals.

The most important rule:

> Tool availability must be designed, not dumped into the prompt.

## Preview of Next Chapter

Chapter 31 organized tools across systems.

Chapter 32 connects tools into workflows.

Next, you will learn workflow orchestration: steps, state, retries, approvals, compensation, async jobs, and failure handling. Registry answers "which tools are allowed?" Orchestration answers "what should happen, in what order, under what state?"
