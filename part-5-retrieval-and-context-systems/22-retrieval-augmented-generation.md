# Chapter 22: Retrieval-Augmented Generation

Chapter 21 built the retrieval pipeline:

```text
source
-> chunk
-> metadata
-> index
-> retrieve
-> filter
-> rerank
-> evidence packet
```

This chapter combines retrieval with generation.

Retrieval-Augmented Generation, or RAG, retrieves trusted external information first, then asks a model to answer using that information.

RAG is not "vector database plus chat." RAG is a knowledge system, retrieval pipeline, context builder, model interface, citation policy, refusal policy, and evaluation loop working together.

## Concept Overview

RAG exists to ground model output in external knowledge.

It has two promises:

```text
1. The answer should use trusted context supplied by the system.
2. The knowledge should be easier to update than retraining the model.
```

Basic RAG flow:

```text
user question
-> understand question
-> retrieve candidate evidence
-> filter and rank evidence
-> build context
-> generate answer
-> cite sources
-> validate or refuse
```

RAG is useful when the answer depends on:

- private documents;
- changing policy;
- internal procedures;
- product docs;
- support playbooks;
- contracts;
- knowledge that should be cited;
- information not trusted from model memory.

The first principle:

> RAG does not make the model know more. It gives the model selected evidence for the current task.

The quality of the answer depends on source quality, retrieval quality, context quality, and model behavior.

## Why It Exists

Imagine a new employee asks:

```text
How do I request access to production logs?
```

A general language model may know common access-control ideas, but it does not know your company's current process unless the system provides it.

The answer may depend on:

- current security policy;
- employee role;
- department;
- approval process;
- production access request form;
- manager approval state;
- audit requirements;
- recent policy changes.

RAG lets the system retrieve current approved policy and ask the model to explain it.

Bad default:

```text
upload docs
-> embed chunks
-> retrieve top 5
-> ask model to answer
-> show answer
```

This may work in a demo. It often fails in real use because it ignores:

- document ownership;
- source quality;
- chunk strategy;
- metadata filters;
- permissions;
- source authority;
- freshness;
- retrieval evaluation;
- citation rules;
- conflict handling;
- refusal behavior.

This chapter exists so learners understand RAG as an engineered workflow, not a feature checkbox.

## Mental Model

Think of RAG as an open-book exam.

The model is allowed to answer using the pages the system gives it. If the system gives the wrong pages, stale pages, forbidden pages, or no relevant page, the answer is at risk.

The mental model:

```text
question
-> evidence search
-> evidence selection
-> evidence packet
-> answer from evidence
-> citation and validation
```

A RAG system should be able to explain:

- which sources were searched;
- which sources were excluded;
- which evidence was retrieved;
- which evidence entered the context;
- which evidence supports the answer;
- when the answer should refuse.

RAG should not be used to hide uncertainty. If approved evidence is missing, the system should say so.

## Internal Mechanics

RAG contains several stages.

### Question Understanding

The system may need to classify or rewrite the user question before retrieval.

Examples:

```text
raw question: How do I rotate it?
conversation state: user is discussing API keys
retrieval query: rotate API key for workspace
```

Question rewriting should preserve intent. It should not invent facts, user role, product version, or permissions.

The system may also detect:

- product area;
- audience;
- language;
- exact terms;
- required permissions;
- whether database lookup is needed.

### Retrieval

Retrieval finds candidate evidence.

It may use:

- keyword search;
- semantic search;
- hybrid retrieval;
- metadata filters;
- exact lookup;
- reranking.

The retrieval step should consider:

```text
What source types are allowed?
What user permissions apply?
What product area is relevant?
Which sources are current?
Which sources are authoritative?
Which sources are forbidden?
```

### Context Building

The context builder decides what evidence the model sees.

It should include:

- source title;
- source URI or ID;
- section heading;
- excerpt;
- last updated date;
- authority level;
- retrieval reason;
- citation identifier.

It should exclude:

- forbidden sources;
- stale sources when current source exists;
- duplicate low-value chunks;
- unrelated high-similarity chunks;
- document text that should not be used as instruction.

Chapter 23 goes deeper into context engineering and grounding.

### Generation

The model receives the task, evidence, and output instructions.

Good RAG instruction:

```text
Answer using only the supplied evidence.
If the evidence does not support the answer, say that approved information is insufficient.
Cite the source sections used.
Do not use outside knowledge for company-specific policy.
```

The model's job is to explain or synthesize from evidence, not invent missing facts.

### Citation

Citations should map answer claims to evidence.

Weak citation:

```text
Answer has a source link somewhere at the end.
```

Stronger citation:

```text
Each important claim cites the supporting source section.
Citation points to the exact retrieved evidence.
Citation is validated against context.
```

Citations are not decoration. They are part of trust and debugging.

### Refusal

RAG should refuse when evidence is missing, forbidden, stale, or insufficient.

Example:

```text
I do not have enough approved information to answer this. The retrieved sources do not describe the current production log access process.
```

This is better than filling gaps with model memory.

### Validation

Validation checks whether the answer follows the RAG contract.

Useful checks:

- answer cites retrieved sources;
- cited sources were actually in context;
- answer does not include unsupported claims;
- answer respects permission scope;
- answer follows refusal rule;
- answer does not cite stale or forbidden sources;
- required source type was used.

## Real-World Example

Use case:

```text
Internal access policy assistant
```

User asks:

```text
How do I request access to production logs?
```

A strong RAG system:

```text
1. authenticates employee
2. detects topic: production log access
3. filters to approved security policy docs
4. excludes drafts, Slack threads, old onboarding PDFs
5. retrieves current access request policy
6. retrieves approval workflow section
7. builds context with source titles, dates, and sections
8. instructs model to answer only from supplied evidence
9. cites the policy section
10. refuses if approved evidence is missing
```

Possible answer:

```text
To request production log access, submit an access request through the Security Access Portal, select "Production Logs," include the incident or business justification, and get approval from your manager and the on-call security reviewer. The access is time-limited and audited. [Security Access Policy > Production Logs]
```

If retrieval finds only an outdated onboarding PDF, the system should not answer confidently.

Better response:

```text
I do not have enough approved current policy to answer. The only matching source is outdated, so this should be routed to security or the current policy source should be updated.
```

That refusal is a product feature, not a failure.

## Common Mistakes and Failure Modes

Mistake 1: Treating RAG as top-k chunks plus chat

Top-k retrieval is only one piece. RAG needs filters, ranking, context policy, citations, refusal, and evals.

Mistake 2: No source authority

The model may blend draft docs, old docs, and approved policy unless the system defines authority.

Mistake 3: No refusal rule

If the system always answers, it will answer unsupported questions.

Mistake 4: Citations are not validated

Models can cite irrelevant or weak sources unless citations are checked against retrieved evidence.

Mistake 5: Using RAG for exact database facts

Account status, payment state, inventory, and current workflow status usually belong in database/API lookup.

Mistake 6: Retrieval failure is hidden

If the expected source was not retrieved, prompt changes may not fix the answer.

Mistake 7: Access control is vague

RAG can leak private documents if permission filtering is not enforced before context assembly.

Mistake 8: No eval questions

Without expected-source tests and grounded-answer checks, RAG quality is anecdotal.

## System Design Decisions

Decide whether RAG is the right pattern.

Use RAG when:

- answer depends on trusted documents;
- knowledge changes more often than model weights;
- citations are valuable;
- source authority matters;
- users need explanations from policy, docs, or playbooks.

Do not use RAG alone when:

- answer is exact database fact;
- workflow needs API action;
- documents are low quality;
- source permissions are unclear;
- calculation or deterministic rule is required;
- current state matters more than documents.

Sometimes the right design is:

```text
database lookup
+ policy retrieval
+ model explanation
```

Define RAG components:

```text
document sources
excluded sources
chunk strategy
retrieval method
metadata filters
authority order
context construction
citation rule
refusal rule
validation rule
eval set
```

Define question types:

```text
direct document answer
synthesis across sources
missing evidence
conflicting sources
stale source
permission-restricted source
database fact needed
human review needed
```

Define fallback behavior:

```text
If retrieval returns no approved source, refuse or ask clarification.
If sources conflict, follow authority order or escalate.
If database fact is required, call API instead of guessing.
If source is stale, do not answer from it unless explicitly allowed.
```

The design goal is grounded usefulness, not constant answer generation.

## Build Artifact

Create `rag-design.md` and `rag-10-question-test.md` for one RAG workflow.

Use this template for `rag-design.md`:

```text
RAG Design

Use case:
User:
Question types:

Document Sources:
- source:
  owner:
  authority:
  included: yes/no
  reason:

Excluded Sources:
- source:
  reason:

Retrieval:
- chunk strategy:
- retrieval method:
- metadata filters:
- authority order:
- freshness rule:
- top-k or candidate limit:
- reranking:

Context Construction:
- evidence fields:
- ordering rule:
- conflict rule:
- compression rule:

Generation:
- model task:
- answer style:
- citation rule:
- refusal rule:
- forbidden behavior:

Validation:
- check:
  failure behavior:

Fallback:
- condition:
  behavior:
```

Use this template for `rag-10-question-test.md`:

```text
RAG 10-Question Test

Use case:
Corpus version:
Index version:

Questions:
- question:
  type:
  expected source:
  acceptable answer:
  forbidden answer:
  refusal expected: yes/no
  notes:

Metrics:
- correct source retrieved:
- answer grounded:
- citation correct:
- refusal correct:
- forbidden answer avoided:
```

Test mix:

```text
4 direct answer questions
3 synthesis questions
1 conflicting-source question
1 stale-source question
1 should-refuse question
```

Example:

```text
Question: How do I request access to production logs?
Type: direct policy answer
Expected source: Security Access Policy > Production Logs
Forbidden answer: generic access-control advice without company process
Refusal expected: no, if approved source is retrieved
```

The artifact is complete when another engineer can implement the RAG flow and evaluate whether it is grounded.

## Active Recall Questions

1. What problem does RAG solve?
2. Why is RAG not just vector search plus chat?
3. What are the main stages in a RAG flow?
4. When should database/API lookup replace or complement RAG?
5. Why are citations part of system design, not decoration?
6. What should happen when no approved source supports the answer?
7. Why should retrieved context be inspected before prompt tuning?
8. What should a 10-question RAG test include?

## Summary

RAG retrieves trusted external evidence and asks a model to answer using that evidence. It is useful for private, changing, document-based knowledge where citations and source authority matter.

But RAG is not a shortcut around system design. It needs reliable sources, retrieval pipelines, metadata filters, context construction, model interfaces, citation rules, refusal behavior, validation, and evals.

The most important rule:

> RAG should answer from approved evidence, not from model confidence.

## Preview of Next Chapter

Chapter 22 combined retrieval with generation.

Chapter 23 controls the evidence the model actually sees.

Next, you will learn context engineering and grounding. This matters because even when retrieval finds the right document, the answer can fail if the evidence packet is noisy, stale, poorly ordered, missing metadata, or missing a refusal rule.
