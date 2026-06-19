# Chapter 29: AI UX, Trust, Feedback, and Human Correction

Chapters 26-28 expanded inputs beyond text. This chapter designs the human interface around those AI outputs.

Before tools and agents, design the interface. If user cannot understand, correct, approve, or stop the system, automation becomes risk.

## Real Problem

A user asks:

```text
Handle this customer issue.
```

Assistant drafts a reply, suggests a refund, opens internal docs, and prepares an action.

The user needs to know:

- what system understood;
- what evidence it used;
- what it plans to do;
- what will happen if they approve;
- how to correct it;
- how to stop it.

AI interface is not just chat. It is control surface.

## Bad Default Solution

Bad interface:

- single chat box;
- hidden plan;
- hidden sources;
- hidden tool arguments;
- vague confidence;
- approve button without consequences;
- no edit path;
- no undo path;
- no trace.

This makes system feel magical until it fails.

## First Principles

An AI interface should expose:

- input;
- interpretation;
- evidence;
- proposed output;
- proposed action;
- risk;
- user controls;
- status;
- trace.

The interface should make uncertainty and authority visible.

For high-risk workflows, user should approve concrete action, not vague intent.

Bad approval:

```text
Approve assistant?
```

Good approval:

```text
Approve creating task "Follow up on API key access" for ticket T-104?
Tool: create_task
Arguments: ticket_id=T-104, title=...
Reason: customer still blocked
```

## System Decision

Choose interaction pattern:

| Pattern | Use When |
|---|---|
| autocomplete | user remains author |
| draft + edit | model proposes text |
| extract + correct | model structures messy input |
| answer + cite | model explains from sources |
| plan + approve | model prepares action |
| agent trace | multi-step task needs inspection |

Design controls:

- edit;
- retry;
- approve;
- deny;
- ask for evidence;
- escalate;
- stop;
- undo when possible.

## Minimal Build

Create `ai-interface-spec.md`:

```text
Workflow:
Primary user:
User intent:

System shows:
- interpretation:
- evidence:
- draft/output:
- proposed action:
- risk:
- trace/status:

User controls:
-

Approval copy:
-

Correction path:
-

Stop/undo path:
-
```

## Failure Modes

AI interfaces fail when:

- system hides uncertainty;
- user cannot inspect evidence;
- approvals are vague;
- correction does not improve future behavior;
- progress state is invisible;
- user cannot stop long-running work;
- final action differs from approved action;
- trace is unavailable after failure.

## Evaluation

Test interface with real tasks:

- normal request;
- ambiguous request;
- wrong extraction;
- missing evidence;
- risky action;
- long-running workflow;
- user denial;
- user correction.

Measure:

- user correction success;
- approval comprehension;
- unsafe approval rate;
- task completion;
- time to recover from wrong suggestion;
- trace usefulness.

## Improvement

Improve interface by:

- showing interpretation before action;
- making evidence inspectable;
- adding field-level correction;
- showing tool arguments;
- requiring explicit approval for writes;
- adding progress states;
- adding stop/retry controls;
- turning corrections into eval data.

## Proof Artifact

Create `ai-interface-spec.md`.

Include:

- interaction pattern;
- user controls;
- evidence display;
- approval design;
- correction flow;
- trace/status design;
- interface test cases.

## Decision Checklist

- [ ] User can inspect interpretation.
- [ ] User can inspect evidence.
- [ ] User can edit/correct output.
- [ ] Risky actions require concrete approval.
- [ ] Long-running work has status.
- [ ] Stop path exists.
- [ ] Corrections become eval data.

## Active Recall

1. Why is chat not always enough?
2. What should an approval show?
3. Why expose evidence in interface?
4. What controls should user have?
5. How can corrections improve system?

## Next

Part 7 connects interface to tools. Once user can inspect, correct, and approve, the system can safely call APIs and workflows.
