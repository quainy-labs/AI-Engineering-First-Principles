# Chapter 1: AI Engineering Is System Design

## Real Problem

A founder says:

> We need an AI assistant for our company knowledge base.

This sounds clear. It is not.

What does "assistant" mean?

- Answer employee questions?
- Search documents?
- Summarize long PDFs?
- Draft customer replies?
- Trigger actions in internal tools?
- Reduce support load?
- Help new employees onboard?
- Replace a manual workflow?

Same phrase, many systems. If builder starts with model choice, they have already skipped engineering.

AI engineering begins before model selection. It begins with system design.

This chapter starts the book's full path:

```text
problem -> software -> data -> models -> retrieval -> interfaces -> action -> production -> governance
```

You begin with problem because every later engineering choice depends on what work should improve. Software architecture, data pipelines, model selection, retrieval, tools, evals, and security are all downstream of this first decision.

## Bad Default Solution

Common path:

1. Pick latest model.
2. Connect documents.
3. Add chat UI.
4. Call it internal copilot.
5. Demo works on three examples.
6. Real users ask messy questions.
7. Answers are incomplete, stale, uncited, or unsafe.
8. Nobody trusts it.

Failure was not "bad prompting." Failure was weak system design.

No clear user. No baseline. No data ownership. No retrieval evaluation. No permissions. No failure handling. No feedback loop. No definition of success.

## First Principles

An intelligent system is still a system.

It has:

- users;
- goals;
- inputs;
- outputs;
- state;
- information sources;
- decisions;
- actions;
- constraints;
- failure modes;
- feedback loops.

AI adds probabilistic behavior. It does not remove engineering responsibility.

Traditional software is mostly deterministic:

> same input + same code -> same output

AI systems often include probabilistic components:

> similar input + same model -> possibly different output

This changes engineering:

- outputs need validation;
- quality needs evaluation sets;
- logs need more context;
- users need trust signals;
- risky actions need approval;
- system needs fallback behavior.

AI engineering is the discipline of deciding which parts should be deterministic, which parts should use learned models, which parts need retrieval, and which parts require human judgment.

That means AI engineering is not one skill. It is a chain of decisions:

- problem decision: should this be built?
- software decision: what system shape supports it?
- data decision: what information does it need?
- model decision: where does learned behavior help?
- retrieval decision: what knowledge must be grounded?
- interface decision: how do users and models communicate?
- action decision: what tools can system safely use?
- production decision: how is quality measured and maintained?
- governance decision: what risks, permissions, and responsibilities exist?

## System Decision

Before building, ask:

1. What work should improve?
2. Who does that work today?
3. What input do they start with?
4. What output do they need?
5. What makes output good?
6. Where does required information live?
7. What can be solved with normal software?
8. Where does language understanding or generation help?
9. What failure would be expensive, unsafe, or embarrassing?
10. How will we measure improvement?

If these are unanswered, do not build agent. Build understanding.

## Minimal Build

Create one-page system brief:

```text
System name:
User:
Current workflow:
Pain:
Baseline:
Input:
Output:
Information sources:
AI role:
Non-AI role:
Human role:
Success metric:
Failure modes:
Proof artifact:
```

Example:

```text
System name: Support Reply Assistant
User: Customer support teammate
Current workflow: Read ticket, search docs, draft reply, ask senior teammate if unsure
Pain: Slow replies for repeated questions
Baseline: Median first draft time = 12 minutes
Input: Customer ticket + product docs
Output: Draft reply with citations
Information sources: Help center, changelog, policy docs
AI role: Retrieve relevant docs, draft response
Non-AI role: Ticket routing, citation validation, template formatting
Human role: Approve before sending
Success metric: Draft time < 5 minutes, citation accuracy > 90%
Failure modes: Wrong policy, missing context, confident hallucination
Proof artifact: 20-ticket eval report
```

This is not glamorous. It prevents waste.

## Failure Modes

AI engineering fails early when:

- problem is vague;
- user is undefined;
- workflow is unknown;
- baseline is missing;
- success metric is subjective;
- system has no trusted data source;
- model is asked to do what deterministic code should do;
- assistant can take risky action without approval;
- demo examples replace evaluation.

## Evaluation

For Chapter 1, evaluate only system clarity.

Pass if system brief answers:

- who uses it;
- what work improves;
- what baseline exists;
- what output matters;
- what AI should and should not do;
- what failure looks like;
- what proof will show impact.

Fail if brief says only:

- "build chatbot";
- "use AI to automate";
- "make workflow faster";
- "improve productivity."

These are hopes, not engineering requirements.

## Improvement

If brief is vague, improve it by interviewing user or observing workflow.

Ask:

- Show me last 5 real examples.
- What made them hard?
- Where did you search?
- What did you copy?
- What did you decide?
- What did you check before sending?
- What mistakes matter?

Real examples beat imagined requirements.

## Proof Artifact

Create `system-brief.md` for one real workflow.

Include:

- current workflow;
- baseline;
- proposed AI role;
- proposed non-AI role;
- failure modes;
- success metric.

## Decision Checklist

- [ ] I can name real user.
- [ ] I can describe current workflow.
- [ ] I know baseline.
- [ ] I know what output must improve.
- [ ] I know what model should not do.
- [ ] I know how system can fail.
- [ ] I know what proof artifact will show impact.

## Active Recall

1. Why is AI engineering system design, not model selection?
2. What changes when system contains probabilistic component?
3. Why is baseline required before building?
4. What should be deterministic instead of AI-driven?
5. What makes "internal copilot" too vague as requirement?

## Next

This chapter defined AI engineering as system design. Next chapter asks harder question before any architecture: when should you refuse to use AI?
