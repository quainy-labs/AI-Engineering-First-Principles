# Chapter 31: Connectors, MCP, and Tool Registries

Chapter 30 taught one tool call:

```text
model proposes -> code validates -> tool executes -> result logged
```

That is enough for one API.

Real products rarely have one API. They connect to ticketing systems, CRMs, calendars, documents, billing systems, browsers, internal tools, and partner services.

This chapter answers:

> How does an AI system know which tools exist, which tools it may use, and how to call them safely?

## What You Already Know

From Chapter 30:

- a tool wraps an external capability;
- tool inputs need schema;
- tool calls can have side effects;
- permissions and approval matter;
- model should propose, system should validate.

This chapter adds the missing layer:

```text
many tools -> organized catalog -> safe tool exposure -> controlled selection
```

## Real Problem

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

At first, the builder hardcodes all tools into one prompt.

It works in demo.

Then the company adds:

- new CRM tool;
- new billing connector;
- new calendar connector;
- deprecated ticket API;
- stricter permission rules;
- different tools for support agent and support lead.

Now tool use becomes messy.

The problem is no longer:

> Can the model call a tool?

The problem is:

> How does the system expose the right tools, to the right workflow, for the right user, with the right version and permissions?

## The Mental Model

Use this map:

```text
user request
-> workflow identifies needed capability
-> registry returns allowed tools
-> connector handles external system details
-> model proposes tool call
-> system validates permissions and schema
-> tool executes
-> result is logged
```

Each layer has a job:

| Layer | Job |
|---|---|
| Workflow | decides what kind of work is happening |
| Registry | lists tools available for that workflow/user |
| Connector | adapts external system into safe interface |
| Tool schema | defines allowed inputs and outputs |
| Policy | checks permission, risk, and approval |
| Trace | records what happened |

If these layers are mixed together, tool systems become fragile.

## Vocabulary Ladder

Learn terms in this order.

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

The model sees the tool name, description, and input schema. The system maps that tool call to real code.

### Connector

A connector is adapter code between your AI system and another product or service.

Example:

```text
Ticketing connector -> read_ticket, add_internal_note
CRM connector -> read_customer_plan
Task connector -> create_task
```

Connector owns:

- auth;
- API details;
- retries;
- rate limits;
- response normalization;
- external errors.

### Tool Registry

A tool registry is catalog of tools.

It answers:

- what tools exist?
- who owns them?
- what schema do they use?
- which workflows may use them?
- which users may use them?
- what risk level do they have?
- what version is active?

### MCP-Style Tool Interface

MCP-style integration means tools can be exposed through a standard interface so clients can discover and call them consistently.

This helps with interoperability.

But it does not solve safety by itself.

Even with MCP-style tools, system still needs:

- permission checks;
- workflow scoping;
- schema validation;
- tool risk classification;
- approval gates;
- trace logs;
- eval cases.

Standard discovery is not same as safe execution.

## Running Example

We will use support operations assistant.

Allowed workflow:

```text
Create follow-up task for existing support ticket.
```

User:

```text
support_agent
```

Available tools:

```text
read_ticket
search_docs
create_task
draft_customer_reply
add_internal_note
```

Forbidden tools:

```text
issue_refund
delete_customer
send_customer_email
grant_admin_access
```

The assistant should create a task, not send final email or modify billing.

## Bad Default Solution

Bad solution:

```text
Here are all company tools. Choose the best one.
```

Problems:

- model sees irrelevant tools;
- high-risk tools appear in low-risk workflows;
- tool descriptions compete with each other;
- permissions are unclear;
- old tool versions stay visible;
- no owner knows when schema changes;
- eval cannot tell whether failure came from model, registry, connector, or permission layer.

This is not tool architecture. This is prompt clutter.

## First Principles

Tool exposure should be scoped.

The model should not see every tool.

It should see only tools that are:

- relevant to workflow;
- allowed for user role;
- safe for current risk level;
- currently active;
- documented with strict schema;
- covered by eval cases.

Tool selection should happen inside system boundary:

```text
workflow + user role + risk -> allowed tool set
```

Then model chooses among allowed tools.

## System Decision

Design tool registry policy:

```text
Who can register a tool?
Who owns each tool?
What schema review is required?
Which workflows can use it?
Which roles can use it?
What risk level does it have?
Does it require approval?
How are versions handled?
How are tools deprecated?
What eval cases prove safe use?
```

Most important decision:

> Tools should be exposed by workflow, not globally.

Example:

| Workflow | Allowed Tools | Forbidden Tools |
|---|---|---|
| answer policy question | search_docs | create_task, issue_refund |
| create follow-up task | read_ticket, create_task | issue_refund, send_email |
| draft customer reply | read_ticket, search_docs, draft_reply | send_email |
| refund review | read_ticket, read_billing | issue_refund without approval |

## Worked Example

Create one registry entry:

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
Permission:
- support_agent can create task for assigned tickets
- support_lead can create task for any ticket in team

Approval:
- required when priority=high

Version: v1
Deprecation: none

Eval cases:
- normal task
- missing ticket_id
- invalid ticket_id
- high priority requires approval
- user lacks ticket access
```

This entry is more useful than a tool description in a prompt because it gives the system enough information to validate, route, evaluate, and debug.

## Minimal Build

Create `tool-registry-spec.md`:

```text
Registry owner:
Who can add tools:
Schema review process:
Permission model:
Risk levels:
Approval policy:
Versioning policy:
Deprecation policy:

Tool entries:
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
  eval cases:
```

Create `tool-selection-eval.md`:

```text
Eval cases:
1. user request:
   workflow:
   user role:
   expected allowed tools:
   expected selected tool:
   forbidden tools:
   expected result:
```

## Failure Story

A support assistant receives:

```text
Tell the customer we are looking into this.
```

Registry exposes both:

```text
draft_customer_reply
send_customer_email
```

The model chooses `send_customer_email`.

Email is sent without review.

Customer receives incomplete message.

Root cause is not only "model made mistake."

Root cause:

- workflow was draft-only;
- registry exposed send tool;
- send tool lacked approval gate;
- eval had no "draft vs send" case.

Correct fix:

- expose only `draft_customer_reply` in draft workflow;
- require approval for send;
- add eval case for accidental send.

## Failure Modes

Tool registries fail when:

- tools are exposed globally;
- registry has no owner;
- stale tool versions remain active;
- tool descriptions are ambiguous;
- high-risk tools appear in low-risk workflows;
- auth is checked after tool selection;
- output schema is missing;
- deprecated tools still appear in prompts;
- eval cases do not include wrong-tool distractors.

## Evaluation

Evaluate registry behavior, not only model behavior.

Test:

- correct tool discovery;
- wrong-tool distractors;
- unauthorized tool;
- deprecated tool;
- schema change;
- connector outage;
- risky action;
- approval-required action.

Metrics:

- allowed-tool accuracy;
- selected-tool accuracy;
- unauthorized exposure rate;
- schema-valid call rate;
- deprecated tool usage;
- connector failure handling;
- approval routing accuracy.

## Improvement

Improve tool registry by:

- reducing visible tools per workflow;
- adding better tool names;
- tightening schemas;
- adding owner and version metadata;
- adding role-based filtering;
- deprecating old tools;
- adding evals for similar tools;
- logging why each tool was exposed.

## Proof Artifact

Create:

- `tool-registry-spec.md`
- `tool-selection-eval.md`

Include:

- at least 5 tool entries;
- workflow-to-tool map;
- permission model;
- risk classification;
- versioning/deprecation policy;
- eval cases with wrong-tool distractors.

## Decision Checklist

- [ ] Tools are scoped by workflow.
- [ ] Tools are scoped by user role.
- [ ] Every tool has owner.
- [ ] Every tool has input and output schema.
- [ ] Every tool has risk level.
- [ ] High-risk tools require approval.
- [ ] Deprecated tools are hidden.
- [ ] Tool-selection eval includes distractors.

## Active Recall

1. What is difference between API, tool, connector, and registry?
2. Why should model not see every tool?
3. What does MCP-style discovery solve?
4. What does MCP-style discovery not solve?
5. Why should tool selection be evaluated separately from final answer?

## Next

Next chapter connects selected tools into workflows. Registry answers "what tools are allowed?" Workflow orchestration answers "what steps should happen, in what order, with what state and failure handling?"
