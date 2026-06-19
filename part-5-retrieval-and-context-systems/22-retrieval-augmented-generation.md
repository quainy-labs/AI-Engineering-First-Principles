# Chapter 22: Retrieval-Augmented Generation

Chapter 21 built retrieval pipeline. Now the model uses retrieved evidence to answer.

Retrieval-Augmented Generation, or RAG, retrieves trusted information first, then asks model to answer using that information.

## Real Problem

A new employee asks:

```text
How do I request access to production logs?
```

The answer depends on current company policy. A language model may know general access-control ideas, but it does not know your company's latest process unless system provides that knowledge.

## Bad Default Solution

Bad RAG:

1. upload docs;
2. embed chunks;
3. retrieve top 5;
4. ask model to answer;
5. show answer.

This may work in demo. It often fails in real use.

Missing:

- document ownership;
- chunk strategy;
- metadata filters;
- source authority;
- retrieval evaluation;
- citation rules;
- refusal behavior;
- stale document handling;
- permissions.

RAG is not "vector database plus chat." RAG is knowledge system plus model interface plus evaluation.

## First Principles

RAG has two promises:

1. model answer should use trusted external context;
2. system should be easier to update than retraining model.

Basic RAG flow:

```text
user question
-> query transformation
-> retrieve candidate sources
-> rank/filter sources
-> build context
-> generate answer
-> cite evidence
-> validate/refuse
```

Each step can fail.

RAG works best when:

- documents are reliable;
- retrieval finds correct evidence;
- context is clean;
- model follows source constraints;
- answer includes citations;
- system refuses unsupported questions.

RAG fails when retrieval fails, context is noisy, sources conflict, or model ignores evidence.

## System Decision

Use RAG when:

- answer depends on private or changing knowledge;
- user needs cited source;
- knowledge lives in documents;
- facts update more often than model;
- model memory alone is not trusted.

Do not use RAG as default when:

- answer is exact database fact;
- workflow needs API action;
- documents are low quality;
- user needs calculation;
- source access permissions are unclear.

Sometimes correct design is:

```text
database lookup + policy retrieval + model explanation
```

not pure RAG.

## Minimal Build

Create `rag-design.md`:

```text
Use case:
User:
Question types:
Document sources:
Excluded sources:
Metadata filters:
Chunk strategy:
Retrieval method:
Top-k:
Context construction:
Citation rule:
Refusal rule:
Evaluation set:
```

Example:

```text
Use case: internal access policy assistant
Document sources: approved security policy docs
Excluded sources: Slack, drafts, outdated onboarding PDFs
Metadata filters: department, policy status, last_updated
Citation rule: every answer must cite policy section
Refusal rule: if no approved source, say unable to answer
```

## Failure Modes

RAG fails when:

- documents are outdated;
- chunking breaks meaning;
- retrieval returns related but wrong source;
- model uses prior knowledge instead of context;
- citations point to irrelevant text;
- answer blends conflicting documents;
- access-controlled documents leak;
- system answers when no source supports answer.

## Evaluation

Start with 10 questions:

- 4 easy questions with direct answer;
- 3 normal questions requiring synthesis;
- 1 conflicting-source question;
- 1 stale-source question;
- 1 should-refuse question.

For each question, define:

- expected source;
- acceptable answer;
- forbidden answer;
- refusal expectation.

Measure:

- correct source retrieved;
- answer grounded;
- citation correct;
- refusal correct.

## Improvement

If RAG fails:

- retrieval miss -> improve query, chunk, or metadata;
- stale source -> add freshness filter;
- unsupported claim -> improve prompt/refusal;
- wrong citation -> require answer-source mapping;
- conflicting docs -> add authority order;
- slow response -> reduce top-k or cache.

Do not tune prompt before inspecting retrieved context.

## Proof Artifact

Create `rag-design.md` and `rag-10-question-test.md`.

Include:

- source list;
- excluded sources;
- chunk strategy;
- retrieval method;
- context rules;
- citation/refusal rules;
- first eval questions.

## Decision Checklist

- [ ] RAG use case depends on trusted documents.
- [ ] Source documents are owned.
- [ ] Excluded sources are named.
- [ ] Metadata filters exist.
- [ ] Citation rule exists.
- [ ] Refusal rule exists.
- [ ] Eval questions exist.

## Active Recall

1. What problem does RAG solve?
2. Why is RAG not just vector search plus chat?
3. When should database lookup replace RAG?
4. What are common RAG failure modes?
5. Why inspect retrieved context before prompt tuning?

## Next

Next chapter goes deeper into context engineering and grounding: what evidence model sees, how it is ordered, and when answer must refuse.
