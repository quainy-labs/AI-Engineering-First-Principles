# Chapter 5: Data, Documents, and State

## Real Problem

A support teammate asks:

> Can we build an AI assistant that answers customer questions from our company knowledge?

Before model, ask: what knowledge?

Some knowledge lives in databases:

- customer plan;
- order status;
- account permissions;
- refund date;
- subscription state.

Some knowledge lives in documents:

- help center articles;
- policy pages;
- product docs;
- release notes;
- internal playbooks.

Some knowledge lives in state:

- current conversation;
- previous action;
- ticket status;
- approval history;
- user preference;
- unresolved task.

If these are mixed together carelessly, system answers wrong questions with confidence.

## Bad Default Solution

Bad solution:

1. Dump all documents into vector database.
2. Connect model.
3. Ask questions.
4. Hope retrieval finds truth.

This ignores:

- database facts;
- document freshness;
- conflicting sources;
- permissions;
- state transitions;
- auditability.

Example failure:

Customer asks:

> Can I get refund?

Policy document says refunds allowed within 30 days. Database says customer bought 45 days ago. If system only retrieves policy, answer may be wrong. If system only checks database, it may miss exception policy.

Real AI systems need information design.

## First Principles

An AI system uses three kinds of information.

### Data

Data is structured information.

It usually lives in:

- database tables;
- CRM records;
- product events;
- transactions;
- forms;
- spreadsheets.

Data is good for exact facts:

- user ID;
- order amount;
- date;
- status;
- count;
- permission;
- inventory.

Data should usually be queried by deterministic code, not guessed by model.

### Documents

Documents are human-readable knowledge.

They usually contain:

- explanations;
- policies;
- procedures;
- examples;
- guidelines;
- context.

Documents are good for meaning, but messy:

- same concept appears under different names;
- old documents conflict with new ones;
- important details hide in long text;
- headings matter;
- source authority matters.

### State

State is what system remembers about current or past interaction.

State answers:

- what happened already?
- what step are we in?
- what did user approve?
- what changed since last turn?
- what should system avoid repeating?

Without state, multi-step workflows break.

## System Decision

For every workflow, classify information:

| Need | Best source | Access method |
|---|---|---|
| Exact current fact | Database/API | Deterministic query |
| Policy or explanation | Document | Search/retrieval |
| Current task progress | State store | Workflow state |
| User approval | Audit log | Deterministic check |
| Ambiguous text meaning | Model | Extraction/classification |

Do not use model as database.

Do not use vector search as source of truth for exact facts.

Do not use chat history as audit log.

## Minimal Build

Create `information-map.md`:

```text
Workflow:
User:
Output:

Structured data needed:
- item:
  source:
  owner:
  freshness:
  access rule:

Documents needed:
- item:
  source:
  authority:
  freshness:
  conflicts:

State needed:
- item:
  lifespan:
  owner:
  reset rule:

Information not available:
- item:
  consequence:
  workaround:
```

Example:

```text
Workflow: Support reply assistant
Structured data needed:
- subscription plan
  source: billing API
  owner: finance systems
  freshness: live
  access rule: support role only

Documents needed:
- refund policy
  source: help center
  authority: public policy page
  freshness: last updated date required
  conflicts: internal exceptions doc may override

State needed:
- draft approval status
  lifespan: ticket lifecycle
  owner: support workflow DB
  reset rule: closes when ticket resolved
```

## Failure Modes

Information design fails when:

- stale document overrides live database;
- vector search retrieves outdated policy;
- model invents missing facts;
- private document leaks to unauthorized user;
- chat memory becomes hidden source of truth;
- conflicting documents have no authority order;
- state persists longer than user expects;
- system cannot explain where answer came from.

## Evaluation

Test information map with real questions:

1. questions answerable from database only;
2. questions answerable from documents only;
3. questions requiring both database and documents;
4. questions requiring current state;
5. questions system must refuse because info unavailable.

Pass if system knows source path for each question.

Fail if answer path is "send everything to model."

## Improvement

If information map is weak:

- name source of truth for each fact;
- add freshness rules;
- add authority order for conflicting docs;
- separate state from documents;
- restrict access by role;
- require citations for document answers;
- require API lookup for live facts.

## Proof Artifact

Create `information-map.md`.

Include:

- structured data;
- documents;
- state;
- source owners;
- freshness;
- access rules;
- unknown information.

## Decision Checklist

- [ ] Exact facts come from database/API.
- [ ] Policy/explanation comes from documents.
- [ ] Task progress lives in state.
- [ ] Source authority is clear.
- [ ] Freshness rules are clear.
- [ ] Permissions are clear.
- [ ] Unknown information is named.

## Active Recall

1. Why should model not act as database?
2. How are data, documents, and state different?
3. What happens when stale document beats live data?
4. Why does source authority matter?
5. What should live in state instead of prompt?

## Next

Next chapter turns information into contracts. Once system knows what information exists, it needs reliable structure for moving that information between components.
