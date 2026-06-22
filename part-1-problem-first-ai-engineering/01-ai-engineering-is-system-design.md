# Chapter 1: AI Engineering Is System Design

Most people begin AI engineering with this question:

```text
Which model should we use?
```

That is usually the wrong first question.

AI engineering begins with:

```text
What work should improve, for whom, and how will we know?
```

This chapter sets the foundation for the whole book. Before models, prompts, RAG, agents, evals, or production, you need one durable idea:

> AI engineering is system design around uncertain components.

## Concept Overview

AI engineering is the practice of designing reliable systems that use AI components to improve real work.

An AI system is not just:

- a model;
- a prompt;
- a chatbot;
- a vector database;
- an agent;
- an API call.

An AI system includes:

- user;
- workflow;
- inputs;
- outputs;
- data sources;
- deterministic code;
- model calls;
- retrieval;
- tools;
- human review;
- evaluation;
- observability;
- security;
- feedback loops.

The model is one component. Sometimes it is the most visible component, but it is rarely the whole system.

This book follows that idea:

```text
problem
-> software
-> data
-> models
-> retrieval
-> multimodal interfaces
-> tools and agents
-> production
-> security and product reality
-> capstone
```

If the first step is vague, every later step becomes expensive guessing.

## Why It Exists

AI engineering exists because modern models are powerful but not automatically useful.

A model can:

- understand messy language;
- summarize;
- classify;
- extract;
- draft;
- reason over provided context;
- propose tool calls.

But real work also needs:

- permissions;
- source of truth;
- state;
- business rules;
- validation;
- approvals;
- logs;
- cost limits;
- latency limits;
- risk controls;
- user trust.

These are system concerns.

Example:

```text
Build an AI assistant for company knowledge.
```

This sounds like a requirement. It is not.

It could mean:

- search documents;
- answer employee questions;
- summarize PDFs;
- onboard new hires;
- draft support replies;
- cite policy sources;
- route questions to owners;
- trigger internal workflows.

Same phrase. Many possible systems.

If you start by choosing a model, you skip the real design work:

```text
Who is the user?
What work is broken?
What information is trusted?
What should AI do?
What should AI not do?
What proof will show improvement?
```

AI engineering exists to answer those questions before building.

## Mental Model

Think of an AI system as a workflow with several owners:

```text
real user
-> real workflow
-> input
-> system boundary
-> deterministic code
-> data/retrieval
-> model component
-> validation
-> human review when needed
-> output/action
-> evaluation and feedback
```

Each part has a job.

| Part | Job |
|---|---|
| User | has work to complete |
| Workflow | shows how work happens today |
| Input | what system receives |
| Deterministic code | handles rules, routing, validation, storage |
| Data/retrieval | provides trusted facts |
| Model | handles language, ambiguity, transformation, drafting |
| Human | owns judgment, approval, accountability |
| Evaluation | proves whether system improved work |
| Feedback | turns failures into improvements |

The core design question:

```text
Which part should own which decision?
```

Bad AI systems give too many decisions to the model.

Good AI systems split responsibility:

```text
code handles rules
data stores truth
retrieval provides evidence
model handles messy language
human approves risky decisions
eval measures quality
```

## Internal Mechanics

An AI system behaves differently from traditional deterministic software.

Traditional software often works like:

```text
same input + same code -> same output
```

AI components may behave like:

```text
similar input + same model -> possibly different output
```

This changes engineering.

### Inputs Need Shape

Models are sensitive to input quality.

Messy input should be normalized:

- clean text;
- structured fields;
- document metadata;
- user role;
- source labels;
- task instruction.

### Outputs Need Contracts

Do not accept vague output if the workflow needs structure.

Bad:

```text
This looks urgent.
```

Better:

```json
{
  "urgency": "high",
  "reason": "customer says finance closes books today",
  "needs_human_review": true
}
```

### Facts Need Sources

Model memory is not source of truth.

Trusted facts should come from:

- database;
- approved documents;
- API;
- user-provided context;
- verified records.

### Risk Needs Control

If output can affect money, permissions, customer messages, legal claims, or safety, the system needs stronger controls:

- validation;
- refusal;
- approval;
- audit log;
- fallback.

### Quality Needs Evaluation

You cannot judge an AI system by one good demo.

You need examples:

- normal cases;
- edge cases;
- missing-information cases;
- should-refuse cases;
- past failures.

This is why AI engineering is system design: uncertainty must be surrounded by structure.

## Real-World Example

A founder says:

```text
We need an AI assistant for support.
```

A weak builder hears:

```text
chatbot + latest model + docs
```

A strong AI engineer asks:

```text
Which support workflow?
Who uses it?
What happens today?
What is slow or broken?
What should the assistant produce?
What should it never do?
What proof will show value?
```

After observing support work, the actual workflow becomes clear:

```text
1. Support teammate reads ticket.
2. They search docs for policy.
3. They check account plan.
4. They draft a reply.
5. They ask senior teammate for risky cases.
6. They send final response.
```

Now the system can be designed:

```text
AI role:
- summarize ticket
- retrieve relevant docs
- draft reply with citations

Code role:
- fetch ticket
- check account plan
- validate citations
- save draft

Human role:
- approve final reply
- handle refund/policy exceptions

Not allowed:
- send final reply automatically
- approve refund
- invent policy
```

This is much clearer than:

```text
Build support chatbot.
```

The better requirement is:

```text
Build a support reply drafting system that reduces first-draft time from 12 minutes to under 5 minutes while keeping citation accuracy above 90%.
```

Now engineering can begin.

## Common Mistakes and Failure Modes

### Mistake 1: Starting With Model Selection

If you choose model before workflow, you optimize the wrong thing.

Model choice depends on:

- task;
- input size;
- output contract;
- latency;
- cost;
- privacy;
- risk.

### Mistake 2: Saying "Assistant" Instead of Naming the Job

Assistant is a broad interface word.

It does not define:

- user;
- workflow;
- output;
- source of truth;
- risk;
- success metric.

### Mistake 3: No Baseline

Without baseline, improvement is a story.

Baseline examples:

- current time per task;
- error rate;
- review time;
- support volume;
- missed SLA rate;
- manual search time.

### Mistake 4: Asking AI To Own Business Decisions

Models can draft, classify, extract, and propose.

They should not secretly own:

- refund approval;
- permission changes;
- compliance decisions;
- final customer communication;
- source-of-truth updates.

### Mistake 5: Treating Demo as Evaluation

Three good examples do not prove system quality.

Real users will bring:

- ambiguous requests;
- missing context;
- stale documents;
- adversarial text;
- edge cases;
- policy exceptions.

### Mistake 6: No Human Role

Even automated systems need accountability.

Someone owns:

- approval;
- correction;
- escalation;
- risk acceptance;
- feedback review.

## System Design Decisions

Before building any AI system, decide:

```text
What work should improve?
Who does that work today?
What input starts the workflow?
What output ends the workflow?
What makes output good?
What is the current baseline?
Where does trusted information live?
What should deterministic code own?
What should model own?
What should human own?
What failures are unacceptable?
How will quality be evaluated?
```

This is the first Quainy-aligned engineering move:

> Turn vague ambition into a bounded system.

Decision split:

| Decision | Better Owner |
|---|---|
| exact account status | database/API |
| policy source | approved documents/retrieval |
| language summary | model |
| structured extraction | model + validator |
| permission check | code |
| risky approval | human |
| final audit record | system log |

If you cannot fill this table, you are not ready to build.

## Build Artifact

Create `system-brief.md`.

Template:

```text
System name:
Primary user:
Current workflow:
Pain:
Baseline:

Input:
Output:
Trusted information sources:

AI role:
Non-AI/code role:
Human role:

Not allowed:
Failure modes:
Success metric:
Evaluation examples needed:
Proof artifact:
```

Worked example:

```text
System name: Support Reply Drafting System
Primary user: Customer support teammate
Current workflow: read ticket, search docs, draft reply, ask senior teammate if unsure
Pain: repeated questions take too long to draft
Baseline: median first-draft time = 12 minutes

Input: customer ticket + account plan + approved support docs
Output: draft reply with citations and needs-review flag
Trusted information sources: help center, policy docs, account API

AI role: summarize ticket, retrieve likely docs, draft reply
Non-AI/code role: fetch account plan, enforce schema, validate citations
Human role: approve before sending, handle exceptions

Not allowed: send final reply, approve refund, invent policy
Failure modes: wrong policy, missing citation, unsupported claim, risky case not escalated
Success metric: draft time < 5 minutes and citation accuracy > 90%
Evaluation examples needed: 20 past tickets with expected sources and review labels
Proof artifact: eval report comparing baseline vs AI-assisted draft workflow
```

Acceptance criteria:

- names a real user;
- describes current workflow;
- includes baseline;
- defines input and output;
- separates AI, code, and human roles;
- lists unacceptable failures;
- defines success metric;
- names proof artifact.

## Active Recall Questions

1. Why is AI engineering system design, not model selection?
2. What changes when a system includes probabilistic components?
3. Why is "build an assistant" not a complete requirement?
4. What should deterministic code own instead of a model?
5. Why does every AI system need baseline before building?
6. What is the difference between demo and evaluation?
7. What belongs in `system-brief.md`?

## Summary

AI engineering starts before prompts and models.

The durable idea:

```text
AI system = workflow + software + data + model + human judgment + evaluation + feedback
```

A model is only one component.

Good AI engineering begins by deciding:

- what work should improve;
- who the system serves;
- what information is trusted;
- what AI should and should not do;
- how quality will be measured;
- what failure looks like.

Do not build the model first.

Build the system brief first.

## Preview of Next Chapter

This chapter showed how to frame AI engineering as system design.

Next chapter asks the harder first-principles question:

> When should you refuse to use AI?
