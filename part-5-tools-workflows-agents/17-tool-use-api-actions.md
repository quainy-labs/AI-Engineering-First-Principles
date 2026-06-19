# Chapter 17: Tool Use and API Actions

## Real Problem

A support assistant can answer questions from docs. Now users ask:

> Can you create a replacement order?

Answering is no longer enough. System must act.

Action changes risk. A wrong answer is bad. A wrong tool call can modify data, charge money, send messages, expose private records, or trigger irreversible workflow.

Tool use turns language model into decision interface for software. That interface must be bounded.

## Bad Default Solution

Bad solution:

- give model many API tools;
- let model decide arguments;
- execute immediately;
- rely on prompt: "only do safe actions";
- store minimal logs.

This fails because:

- model may choose wrong tool;
- arguments may be invalid;
- user may lack permission;
- action may not be idempotent;
- tool result may be misread;
- audit trail missing;
- approval missing for risky steps.

Tool use needs engineering, not trust.

## First Principles

A tool is external capability exposed to model.

Tool call has:

- name;
- description;
- input schema;
- permissions;
- side effects;
- validation;
- execution result;
- error behavior;
- audit log.

Safe tool use separates:

```text
model proposes -> code validates -> policy checks -> human approves if needed -> tool executes -> result logged
```

Model should propose action. System decides if action is allowed.

## System Decision

Classify tools by risk:

| Tool type | Example | Approval |
|---|---|---|
| Read-only | search order | usually no |
| Draft-only | draft email | human before send |
| Reversible write | add note | maybe no |
| External send | email customer | yes |
| Financial action | refund | yes |
| Permission change | grant access | yes |
| Destructive action | delete record | strict approval or disallow |

Start with read-only tools. Add write tools only after logs, permissions, and approval exist.

## Minimal Build

Create `tool-interface-spec.md`:

```text
Tool name:
Purpose:
User intent it supports:
Inputs:
Output:
Side effects:
Risk level:
Permissions:
Validation:
Idempotency:
Approval required:
Failure behavior:
Audit log fields:
```

Example:

```text
Tool name: create_replacement_order
Inputs: order_id, sku, reason
Side effects: creates new shipment
Risk level: high
Permissions: support_lead
Validation: original order exists, sku eligible, reason enum
Approval required: yes
Audit log: user, ticket_id, old_order_id, new_order_id, approver, timestamp
```

## Failure Modes

Tool use fails when:

- tool descriptions are vague;
- model chooses tool from name only;
- input schema too permissive;
- permission checked after execution;
- approval skipped;
- tool result not validated;
- retries duplicate side effects;
- logs do not capture who/why/what.

Idempotency matters. If retry repeats payment, system failed.

## Evaluation

Create tool-call eval:

- correct tool selection;
- correct arguments;
- invalid argument rejection;
- permission denial;
- approval routing;
- safe refusal;
- idempotent retry.

Use examples:

```text
User asks to check order status -> read_order
User asks to refund without eligibility -> deny/escalate
User asks to delete customer data -> refuse or compliance flow
User asks to send angry reply -> draft only, no send
```

Metrics:

- tool selection accuracy;
- argument validity;
- unsafe execution rate;
- approval accuracy;
- duplicate-action rate.

## Improvement

Improve tool use by:

- narrowing tool scope;
- adding schema enums;
- improving descriptions;
- splitting read/write tools;
- adding dry-run mode;
- adding approval gates;
- adding audit log;
- adding idempotency keys;
- denying ambiguous requests.

## Proof Artifact

Create `tool-interface-spec.md`.

Include:

- tool list;
- schemas;
- risk classification;
- permissions;
- approval rules;
- audit log fields;
- tool-call eval cases.

## Decision Checklist

- [ ] Tool side effects are known.
- [ ] Input schema is strict.
- [ ] Permission check happens before execution.
- [ ] Approval rules exist.
- [ ] Idempotency considered.
- [ ] Tool result is logged.
- [ ] Unsafe execution eval exists.

## Active Recall

1. Why is tool use riskier than answering?
2. What should model propose versus what code should decide?
3. Why classify tools by risk?
4. What is idempotency?
5. What should audit log capture?

## Next

Next chapter connects tools into workflows. Tool calls alone are not enough; real work needs steps, retries, state, and handoffs.
