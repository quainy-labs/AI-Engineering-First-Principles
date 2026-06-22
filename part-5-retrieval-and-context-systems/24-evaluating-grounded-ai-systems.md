# Chapter 24: Evaluating Grounded AI Systems

Chapter 23 defined grounding and context policy:

```text
allowed sources
-> authority order
-> context packet
-> citation rule
-> refusal rule
```

This chapter measures whether the system actually follows those rules.

A fluent answer is not enough. A grounded AI system must prove where the answer came from, whether the source was allowed, whether the citation supports the claim, and when the system should not answer.

Grounded systems need evaluation at two layers:

```text
retrieval quality
answer quality
```

If the answer is bad, inspect retrieval first.

## Concept Overview

Evaluating a grounded AI system means testing whether the system finds the right evidence and answers only from that evidence.

It answers:

```text
Did retrieval find the expected source?
Did retrieval avoid forbidden sources?
Did retrieval avoid stale sources?
Did the answer use the retrieved evidence?
Did citations support the claims?
Did the system refuse when evidence was missing?
Did the system respect permissions?
How much did it cost?
How long did it take?
```

Grounded evaluation is different from judging whether an answer "sounds good."

The first principle:

> A grounded answer must be useful and supported.

Useful means it addresses the user question.

Supported means the answer can be traced to allowed evidence.

## Why It Exists

RAG demos often look good on five hand-picked questions.

Then real users ask:

- questions with no answer in the docs;
- questions requiring two documents;
- questions where old docs conflict with new docs;
- questions with exact policy nuance;
- questions outside their access scope;
- questions with misleading wording;
- questions where the right answer is refusal.

Trust breaks when evaluation only checks easy cases.

Bad evaluation:

- ask a few easy questions;
- judge answer by vibes;
- ignore retrieved sources;
- ignore citations;
- ignore refusals;
- ignore stale documents;
- ignore forbidden sources;
- ignore cost and latency.

This hides real failures.

This chapter exists because grounded AI must be measured across the full evidence chain, not only the final text.

## Mental Model

Think of grounded evaluation as two inspections.

First inspection:

> Did the system bring the right evidence into the room?

Second inspection:

> Did the answer use that evidence correctly?

The mental model:

```text
question
-> retrieved sources
-> context packet
-> generated answer
-> citations
-> validation
-> eval result
```

If the correct source was never retrieved, the generation layer did not get a fair chance.

If the correct source was retrieved but the answer is wrong, inspect context ordering, source labels, instructions, citation policy, and model behavior.

If the answer is unsupported but fluent, the grounding policy failed.

## Internal Mechanics

Build the eval set before optimizing.

### Eval Set Design

A useful grounded eval set includes multiple question types.

Start with 40 examples:

```text
easy direct answer: 10
normal question: 10
multi-source synthesis: 5
conflicting-source trap: 5
stale-source trap: 5
should-refuse: 5
```

Add permission-boundary cases if the product has access control.

For each eval item, record:

```text
question
user role
expected source
acceptable alternate source
forbidden source
expected answer
forbidden answer
must cite
should refuse
risk level
notes
```

The expected source matters because answer quality cannot be separated from evidence quality.

### Retrieval Metrics

Retrieval metrics ask whether the right evidence was found.

Useful metrics:

```text
top-1 accuracy
top-3 recall
top-5 recall
expected-source recall
forbidden-source rate
stale-source rate
duplicate-source rate
no-result rate
permission violation rate
```

Example:

```text
Question: How do I request production log access?
Expected source: Security Access Policy > Production Logs
Forbidden source: Old Onboarding Guide > Log Access
```

If retrieval returns the old onboarding guide, generation should not be blamed first.

### Answer Metrics

Answer metrics ask whether the generated response used evidence correctly.

Useful metrics:

```text
answer relevance
faithfulness
citation accuracy
unsupported claim rate
refusal accuracy
over-refusal rate
under-refusal rate
source-conflict handling
```

Faithfulness means:

> The answer's claims are supported by the provided evidence.

An answer can be relevant but unfaithful.

Example:

```text
Question: Can contractors access production logs?
Answer: Contractors can access logs with manager approval.
Evidence: Only employees can request access.
```

The answer addresses the question, but it is not faithful.

### Citation Accuracy

Citation accuracy checks whether citations support the claims they are attached to.

A citation is wrong when:

- the cited source was not retrieved;
- the cited source is irrelevant;
- the cited source says something different;
- the answer cites a broad document but the claim needs a specific section;
- the source is stale or forbidden.

Citation evaluation should check claim-source mapping, not only whether a link exists.

### Refusal Accuracy

Grounded systems must know when not to answer.

Should-refuse cases include:

- no approved source exists;
- source is stale;
- source is forbidden for user role;
- sources conflict without authority rule;
- answer requires database fact not provided;
- user asks outside product scope.

Measure:

```text
correct refusal
over-refusal
under-refusal
refusal explanation quality
```

Under-refusal is often more dangerous than over-refusal. It means the system answered when it should not have.

### System Metrics

Grounded quality must still fit the product.

Track:

- average latency;
- p50 and p95 latency;
- average cost;
- cost per successful grounded answer;
- token usage;
- fallback rate;
- human-review rate.

A grounded system that is too slow or too expensive may still fail as a product.

### Versioned Reports

Every eval run should record versions:

```text
corpus version
index version
retriever version
reranker version
context policy version
prompt/interface version
model version
eval set version
```

Without versions, improvement cannot be explained.

## Real-World Example

Use case:

```text
Internal access policy assistant
```

Eval item:

```text
Question:
How do I request access to production logs?

User role:
employee

Expected source:
Security Access Policy > Production Logs

Forbidden source:
Old Onboarding Guide > Log Access

Expected answer:
Submit request through Security Access Portal, include justification, get manager and security approval.

Must cite:
Security Access Policy > Production Logs

Should refuse:
no
```

Bad result:

```text
Retrieval:
Top result: Old Onboarding Guide

Answer:
Ask your manager in Slack for access.
```

Diagnosis:

```text
Layer: retrieval
Failure: stale-source retrieval
Fix: filter archived docs and boost current approved policy
```

Another result:

```text
Retrieval:
Top result: Security Access Policy > Production Logs

Answer:
Submit the request through the portal. Manager approval is optional.
```

Diagnosis:

```text
Layer: generation/context
Failure: unfaithful answer
Fix: improve context policy, citation validation, or answer instruction
```

Same question. Different layer. Different fix.

## Common Mistakes and Failure Modes

Mistake 1: Evaluating only easy questions

Easy questions hide failures around missing evidence, conflicts, stale sources, permissions, and refusals.

Mistake 2: Judging final answer without inspecting retrieval

If the right source was not retrieved, generation is downstream of the problem.

Mistake 3: No should-refuse cases

If refusal is not tested, the system will learn to always answer.

Mistake 4: No forbidden-source labels

The system may appear accurate while relying on sources it should not use.

Mistake 5: Citations treated as links, not support

Citation accuracy requires checking whether the cited source supports the claim.

Mistake 6: Eval set changes during comparison

Changing examples makes before/after results hard to trust.

Mistake 7: Evaluator rewards fluent unsupported answers

A beautiful answer can still be ungrounded.

Mistake 8: Cost and latency ignored

The system must be accurate enough and operationally usable.

## System Design Decisions

Define eval categories:

```text
direct answer
normal question
multi-source synthesis
conflicting-source trap
stale-source trap
should-refuse
permission-boundary
exact-term query
```

Define retrieval metrics:

```text
top-1 accuracy
top-3 recall
expected-source recall
forbidden-source rate
stale-source rate
permission violation rate
```

Define answer metrics:

```text
relevance
faithfulness
citation accuracy
unsupported claim rate
refusal accuracy
over-refusal
under-refusal
```

Define system metrics:

```text
average latency
p95 latency
average cost
cost per successful grounded answer
fallback rate
human-review rate
```

Define release criteria:

```text
What score is good enough?
Which categories block release?
What forbidden-source rate is acceptable?
What under-refusal rate is acceptable?
What latency or cost limit matters?
```

Define failure diagnosis:

```text
If expected source missing -> retrieval fix.
If expected source present but answer wrong -> context/generation fix.
If citation wrong -> citation validation fix.
If answer unsupported -> grounding/refusal fix.
If stale source used -> freshness/filter fix.
```

Evaluation should tell the team where to work next.

## Build Artifact

Create `grounded-eval-set.md` and `grounded-eval-report.md`.

Use this template for `grounded-eval-set.md`:

```text
Grounded Eval Set

Use case:
Eval set version:
Corpus version:

Categories:
- category:
  target count:
  purpose:

Eval Items:
- id:
  category:
  question:
  user role:
  expected source:
  acceptable alternate sources:
  forbidden sources:
  expected answer:
  forbidden answer:
  must cite:
  should refuse:
  risk level:
```

Use this template for `grounded-eval-report.md`:

```text
Grounded Eval Report

Run date:
Eval set version:
Corpus version:
Index version:
Retriever version:
Reranker version:
Context policy version:
Prompt/interface version:
Model version:

Retrieval Results:
- top-1 accuracy:
- top-3 recall:
- expected-source recall:
- forbidden-source rate:
- stale-source rate:

Answer Results:
- answer relevance:
- faithfulness:
- citation accuracy:
- unsupported claim rate:
- refusal accuracy:
- over-refusal rate:
- under-refusal rate:

System Results:
- average latency:
- p95 latency:
- average cost:
- cost per successful grounded answer:

Top Failure Categories:
- category:
  count:
  example:
  likely layer:
  next fix:

Release Decision:
- approved:
- blocked by:
- owner:
```

Example eval item:

```text
id: access-001
category: direct answer
question: How do I request access to production logs?
user role: employee
expected source: Security Access Policy > Production Logs
forbidden source: Old Onboarding Guide > Log Access
expected answer: request through Security Access Portal with manager and security approval
must cite: Security Access Policy > Production Logs
should refuse: no
```

The artifact is complete when another engineer can run the same eval after retrieval, context, prompt, or model changes and compare results honestly.

## Active Recall Questions

1. Why should retrieval be evaluated separately from answer generation?
2. What is faithfulness?
3. Why should grounded eval sets include should-refuse questions?
4. What is forbidden-source rate?
5. Why is citation accuracy more than checking whether a link exists?
6. What does under-refusal mean?
7. Why should corpus, index, retriever, prompt, and model versions be recorded?
8. How do you decide whether a failure belongs to retrieval or generation?

## Summary

Grounded AI evaluation checks both evidence retrieval and answer behavior. The system must find allowed, current, authoritative sources and then answer faithfully with correct citations or refuse when evidence is insufficient.

A strong eval set includes easy, normal, multi-source, stale-source, conflicting-source, should-refuse, and permission-boundary cases. A strong report tracks retrieval metrics, answer metrics, system metrics, versions, and failure categories.

The most important rule:

> Do not evaluate grounded AI by vibes. Evaluate the evidence chain.

## Preview of Next Chapter

Chapter 24 measured grounded behavior.

Chapter 25 turns a grounded assistant into a maintained knowledge product.

Next, you will learn why a chat UI is not the full product. Reliable knowledge systems need source owners, freshness processes, feedback loops, admin controls, analytics, eval schedules, and review paths so answers stay trustworthy after documents, users, policies, and risks change.
