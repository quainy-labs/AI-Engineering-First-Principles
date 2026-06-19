# Chapter 2: When Not to Use AI

## Real Problem

A team wants AI to process refund requests.

Current process:

1. Customer submits request.
2. System checks order date.
3. System checks refund policy.
4. If eligible, refund is approved.
5. If unclear, human reviews.

Team asks for "AI refund agent."

But most of this workflow is rule-based. Order date, payment status, policy window, and refund amount are deterministic. A model should not decide what code can determine exactly.

Good AI engineering includes refusing AI.

## Bad Default Solution

Bad solution:

- Send full ticket to model.
- Ask if refund should be approved.
- Trust natural language answer.
- Add "be careful" to prompt.

This creates risk:

- inconsistent decisions;
- policy drift;
- legal exposure;
- hard-to-debug behavior;
- no audit trail;
- avoidable model cost.

The better system uses deterministic policy checks first. AI may summarize ticket, extract reason, or draft human-facing explanation. It should not own final eligibility when rules are clear.

## First Principles

AI is useful when input is messy and desired output requires interpretation.

AI is weak when task needs:

- exact arithmetic;
- strict policy enforcement;
- database truth;
- access control;
- repeatable decisions;
- regulatory audit;
- low-latency high-volume execution;
- simple rules.

Use software for certainty. Use AI for ambiguity.

AI should often sit between messy human input and deterministic systems:

```text
messy text -> AI extraction/classification -> validated schema -> deterministic workflow
```

Not:

```text
messy text -> AI decides everything
```

## System Decision

Use this ladder before adding AI:

1. Can simple rule solve it?
2. Can form or better UX solve it?
3. Can search solve it?
4. Can deterministic workflow solve it?
5. Can small classifier solve it?
6. Does language model add value?
7. Does tool-using workflow need model judgment?
8. Does risky step require human approval?

Prefer simplest system that meets need.

## Minimal Build

Create AI necessity table:

| Task | Best method | Why |
|---|---|---|
| Check refund window | Deterministic code | Exact policy |
| Detect customer reason | LLM extraction | Messy text |
| Calculate refund amount | Deterministic code | Exact arithmetic |
| Draft reply | LLM generation | Language output |
| Approve edge case | Human | Risk/ambiguity |

This table is one of most valuable artifacts in AI engineering. It keeps system honest.

## Failure Modes

AI is overused when:

- model makes policy decisions;
- model does arithmetic;
- model invents data instead of querying database;
- model chooses permissions;
- model replaces cheap deterministic checks;
- model is used because "AI" sounds better than form redesign.

Underuse also exists:

- forcing users into rigid forms when natural language would help;
- ignoring summarization for long documents;
- avoiding semantic search where keyword search fails;
- making humans repeatedly classify messy input.

Good engineering is not anti-AI. It is precise about where intelligence belongs.

## Evaluation

For each proposed AI feature, classify task:

- deterministic;
- retrieval;
- extraction;
- classification;
- generation;
- tool action;
- human judgment.

Pass if every AI use has reason tied to ambiguity, language, scale, or adaptation.

Fail if reason is "because AI can do it."

## Improvement

If system overuses AI, move certainty into code:

- policy rules;
- calculations;
- permission checks;
- formatting;
- validation;
- database lookup;
- routing logic.

Then use model for tasks around language, context, and uncertainty.

## Proof Artifact

Create `ai-necessity-table.md` for your workflow.

Include:

- task list;
- best method;
- reason;
- risk if AI used incorrectly;
- final system boundary.

## Decision Checklist

- [ ] I checked if rules solve task.
- [ ] I checked if better UX solves task.
- [ ] I separated exact decisions from ambiguous decisions.
- [ ] I kept permissions out of model judgment.
- [ ] I kept arithmetic out of model judgment.
- [ ] I can explain why AI belongs where used.

## Active Recall

1. When is AI weaker than deterministic code?
2. Why should models not own policy enforcement?
3. What does "use software for certainty, AI for ambiguity" mean?
4. How can AI support deterministic workflow without replacing it?
5. What is AI necessity table?

## Next

Next chapter turns chosen problem into system boundary.

