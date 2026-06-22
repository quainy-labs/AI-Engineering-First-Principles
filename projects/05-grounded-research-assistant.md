# Project 5: Grounded Research Assistant

## Concept

RAG is not a chatbot with documents attached. It is a system that retrieves trustworthy context, builds a constrained prompt, answers only from evidence, cites sources, refuses unsupported questions, and measures failures separately.

This project builds a grounded research assistant over a controlled document set.

The learner should finish with a runnable CLI, notebook, or local app that indexes documents, retrieves relevant chunks, builds context, generates grounded answers, validates citations, captures feedback, and evaluates retrieval and answer quality.

This is the first full document-answering AI product in the project path.

The focus is:

```text
question -> retrieval -> context -> grounded answer -> citations -> evaluation
```

## What You Will Build

Build a Grounded Research Assistant.

The system accepts user questions over a trusted document collection and returns:

- answer;
- citations;
- retrieved source list;
- evidence excerpts;
- confidence or limitation note;
- refusal when evidence is missing;
- feedback option;
- retrieval evaluation report;
- grounded answer evaluation report.

The product answers one question:

```text
Can this system answer from trusted sources without pretending to know what it cannot support?
```

## Chapters Practiced

This project proves understanding from:

- Chapter 20: embeddings and semantic retrieval;
- Chapter 21: chunking, indexing, and retrieval pipelines;
- Chapter 22: retrieval-augmented generation;
- Chapter 23: context engineering and grounding;
- Chapter 24: evaluating grounded AI systems;
- Chapter 25: from assistant demo to knowledge product.

It may reuse earlier parts for app structure, data ingestion, metadata, search, and model interfaces, but it should not require future course concepts.

Do not require multimodal inputs, OCR, audio, tools, agents, workflow actions, production deployment, LLMOps, security threat modeling, or governance review here.

Later parts can improve the same project:

- Part 6 can add PDFs, OCR, images, tables, and audio transcripts;
- Part 7 can turn answers into controlled tool-assisted workflows;
- Part 8 can add production observability, caching, cost, latency, deployment, and rollback;
- Part 9 can add prompt-injection tests, retrieval poisoning controls, privacy, access control, and governance.

## User Story

Primary user:

```text
Student, researcher, support teammate, analyst, or internal operator.
```

User story:

```text
As a researcher,
I want to ask questions over trusted documents,
so I can get cited answers and know when the system does not have enough evidence.
```

The user needs more than fluent text:

```text
answer
source
evidence
limits
refusal when unsupported
```

## System Requirements

### Input

Use a controlled document collection.

You can reuse Project 3 documents or create a new corpus:

```text
docs/
  policies/
    refunds.md
    access-control.md
  product/
    api-keys.md
    billing-admins.md
  support/
    escalation-playbook.md
```

Supported formats for this project:

- `.md`;
- `.txt`;
- optional structured JSON document records from Project 3.

Do not require PDF/OCR/table extraction here. That belongs to Part 6.

### Output

For each question, return:

```text
answer
citations
retrieved sources
evidence excerpts
status: answered / refused / needs_review
reason if refused
feedback record
```

### Interface

Choose one implementation path:

```text
CLI:
rag index docs/
rag ask "How do I reset an API key?"
rag eval eval-questions.json
rag feedback Q001 --label wrong_source

Notebook:
Run indexing, retrieval, answering, and eval cells.

Local app:
Question box, answer panel, source panel, feedback buttons.
```

CLI or notebook is enough for this project.

## Starter Corpus

Create 15-30 documents.

Each document should have metadata:

```yaml
title: API Key Management
owner: Platform Support
authority: approved
last_updated: 2026-01-10
tags: [api, access, keys]
```

Example document:

```markdown
---
title: API Key Management
owner: Platform Support
authority: approved
last_updated: 2026-01-10
tags: [api, access, keys]
---

# API Key Management

Customers can reset API keys from workspace settings if they are workspace admins.

## Reset Process

1. Open Workspace Settings.
2. Select API Keys.
3. Choose Reset Key.
4. Confirm the reset.

Support agents must not generate or reveal API keys manually.
```

Seed the corpus with:

```text
Good sources:
- API key reset guide;
- refund policy;
- billing admin guide;
- support escalation guide;
- access-control policy.

Distractors:
- release notes mentioning API keys;
- sales FAQ mentioning refunds;
- old deprecated refund policy;
- troubleshooting guide with similar terms.

Unsupported topics:
- topics not covered by corpus;
- questions that require private account data;
- questions that require current external facts.
```

## MVP Milestone

Build the simplest useful RAG system first.

MVP must:

1. Load documents.
2. Split documents into chunks.
3. Attach metadata to each chunk.
4. Build a retrieval index.
5. Retrieve top chunks for a question.
6. Build context with source metadata.
7. Generate answer from context.
8. Include citations.
9. Refuse when evidence is missing.

MVP flow:

```text
docs
  -> parser
  -> chunker
  -> index
  -> retriever
  -> context builder
  -> grounded prompt
  -> answer generator
  -> citation output
```

If embeddings are not available, start with keyword retrieval and design the retriever boundary so semantic retrieval can be added.

The required learning is the RAG pipeline and grounding behavior, not a specific vendor.

## V1 Milestone

After MVP works, improve retrieval and answer reliability.

V1 should include:

- semantic retrieval with embeddings;
- keyword retrieval from Project 3;
- hybrid retrieval;
- metadata filtering;
- chunk overlap experiment;
- reranking or simple score blending;
- citation validator;
- unsupported-question refusal;
- feedback capture;
- eval runner.

V1 should separate failures:

```text
retrieval failed
context was insufficient
answer ignored context
citation was wrong
question was unsupported
source was stale
```

Do not collapse every issue into "model bad."

## Knowledge Product Milestone

Add a short file named `knowledge-product.md`.

It should explain:

- who the assistant serves;
- what document corpus is trusted;
- which questions are supported;
- which questions should be refused;
- how feedback becomes new eval cases;
- how source quality affects answer quality;
- why this is still not an autonomous agent.

This keeps the project aligned with Part 5: building a grounded knowledge product, not a generic chatbot.

## Core Components

### 1. Document Loader

Responsibilities:

- read documents;
- parse metadata;
- preserve source path;
- preserve headings;
- record document authority and freshness.

Required metadata:

```text
title
owner
authority
last_updated
tags
source_path
```

### 2. Chunker

Split documents into retrievable units.

Chunking should preserve:

- source path;
- document title;
- heading;
- section text;
- chunk ID;
- chunk order;
- metadata.

Example chunk:

```json
{
  "chunk_id": "product/api-keys.md#reset-process",
  "doc_id": "product/api-keys.md",
  "title": "API Key Management",
  "heading": "Reset Process",
  "text": "Open Workspace Settings. Select API Keys. Choose Reset Key.",
  "authority": "approved",
  "last_updated": "2026-01-10"
}
```

Test at least two chunking choices:

```text
heading-based chunks
fixed-size chunks with overlap
```

Compare retrieval results.

### 3. Index Builder

Build one or more indexes:

- keyword index;
- semantic/vector index;
- optional hybrid index.

Index records should include:

```text
chunk_id
text
embedding if used
metadata
content hash
index version
```

Do not lose metadata during embedding. Metadata is part of retrieval quality.

### 4. Retriever

The retriever accepts:

```text
query
top_k
filters
retrieval_mode
```

Retrieval modes:

```text
keyword
semantic
hybrid
```

Search filters:

```text
authority=approved
tag=access
owner=Support Ops
exclude_deprecated=true
```

Retriever output:

```json
{
  "query": "How do I reset an API key?",
  "retrieval_mode": "hybrid",
  "results": [
    {
      "chunk_id": "product/api-keys.md#reset-process",
      "score": 0.87,
      "retrieval_reason": "semantic match and keyword match on api key reset"
    }
  ]
}
```

### 5. Context Builder

The context builder converts retrieved chunks into model context.

Each context item should include:

- source ID;
- title;
- heading;
- authority;
- last updated;
- excerpt;
- retrieval reason.

Context format example:

```text
[Source S1]
Title: API Key Management
Path: product/api-keys.md#reset-process
Authority: approved
Last updated: 2026-01-10
Excerpt:
Customers can reset API keys from workspace settings if they are workspace admins...
```

The model should know what text is evidence and where it came from.

### 6. Grounded Answer Prompt

Prompt rules:

```text
Answer only using provided sources.
Cite sources for factual claims.
If sources are insufficient, say you do not have enough information.
Do not use outside knowledge.
Do not invent citations.
Mention important limitations.
```

Bad answer:

```text
You can reset it from settings.
```

Better answer:

```text
Workspace admins can reset API keys from Workspace Settings by selecting API Keys, choosing Reset Key, and confirming the reset. Support agents must not generate or reveal API keys manually. [S1]
```

### 7. Refusal Logic

The system should refuse when:

- no relevant chunks found;
- top results are below score threshold;
- sources conflict and cannot be resolved;
- question asks outside corpus;
- question asks for private/account-specific data not in corpus;
- citation validator fails.

Refusal example:

```text
I do not have enough information in the approved sources to answer this. The retrieved documents do not describe changing a company logo color.
```

Refusal is a product feature. It protects trust.

### 8. Citation Validator

Validate:

- answer contains citations;
- cited source exists in retrieved context;
- citations point to relevant source;
- unsupported answer is flagged;
- citation IDs are not invented.

Simple validator is acceptable:

```text
Every non-refusal answer must include at least one source ID.
Every cited source ID must exist in context.
Answer should not cite deprecated or failed-quality sources.
```

Advanced support checking can be a later extension.

### 9. Feedback Capture

Capture feedback labels:

```text
useful
wrong_source
unsupported_claim
missing_answer
stale_source
confusing_answer
should_have_refused
```

Feedback record:

```json
{
  "question_id": "Q001",
  "question": "How do I reset an API key?",
  "answer": "...",
  "feedback": "wrong_source",
  "comment": "It cited release notes instead of API key guide."
}
```

Feedback should become future eval cases.

### 10. Evaluation Runner

Create two eval layers:

```text
retrieval eval
answer eval
```

Retrieval eval measures whether the right chunks are found.

Answer eval measures whether the final answer is grounded, cited, and correct.

Do not hide retrieval failures inside answer scores.

## Architecture

```text
docs
  -> document loader
  -> chunker
  -> index builder
  -> retriever
  -> context builder
  -> answer generator
  -> citation validator
  -> feedback logger
  -> eval runner
```

Optional service architecture:

```text
CLI or UI
  -> question service
  -> retrieval service
  -> context service
  -> model gateway
  -> validation service
  -> feedback store
```

Key boundary:

```text
Retriever selects evidence.
Context builder packages evidence.
Model writes answer.
Citation validator checks grounding rules.
```

## Data Model

### Chunk Record

```json
{
  "chunk_id": "product/api-keys.md#reset-process",
  "doc_id": "product/api-keys.md",
  "title": "API Key Management",
  "heading": "Reset Process",
  "text": "Customers can reset API keys from workspace settings...",
  "authority": "approved",
  "last_updated": "2026-01-10",
  "tags": ["api", "access", "keys"],
  "content_hash": "sha256:...",
  "index_version": "rag-index-v1"
}
```

### Retrieval Result

```json
{
  "query": "How do I reset an API key?",
  "mode": "hybrid",
  "top_k": 5,
  "results": [
    {
      "chunk_id": "product/api-keys.md#reset-process",
      "score": 0.87,
      "rank": 1,
      "retrieval_reason": "Matched api, key, reset and semantic similarity."
    }
  ]
}
```

### Answer Record

```json
{
  "question_id": "Q001",
  "question": "How do I reset an API key?",
  "status": "answered",
  "answer": "Workspace admins can reset API keys from Workspace Settings... [S1]",
  "citations": [
    {
      "source_id": "S1",
      "chunk_id": "product/api-keys.md#reset-process"
    }
  ],
  "retrieved_chunks": ["product/api-keys.md#reset-process"],
  "validation": {
    "citations_valid": true,
    "unsupported_claim_flag": false
  }
}
```

### Feedback Record

```json
{
  "question_id": "Q001",
  "label": "useful",
  "comment": "Clear answer and correct source.",
  "created_at": "2026-01-10T10:00:00Z"
}
```

## Evaluation Dataset

Create `eval-questions.json` with at least 40 questions.

Include:

- 15 easy questions;
- 10 normal questions;
- 5 multi-source questions;
- 5 edge questions;
- 5 should-refuse questions.

Each eval item should include:

```text
question
expected_sources
forbidden_sources
expected_status: answered / refused
answer_requirements
```

Example:

```json
{
  "question_id": "Q001",
  "question": "How do I reset an API key?",
  "expected_sources": ["product/api-keys.md#reset-process"],
  "forbidden_sources": ["product/release-notes-2026-01.md"],
  "expected_status": "answered",
  "answer_requirements": [
    "must say workspace admin",
    "must say support agents must not reveal keys"
  ]
}
```

## Seeded Failure Cases

Include cases that expose weak retrieval and weak grounding.

### Case 1: Distractor Source

```text
Release notes mention "API key reset" but do not explain the process.
Expected: reset guide should be retrieved above release notes.
```

### Case 2: Deprecated Policy

```text
Old refund policy contains exact keywords.
Expected: current approved refund policy should outrank deprecated policy.
```

### Case 3: Unsupported Question

```text
Question asks for company logo customization, but corpus has no source.
Expected: refusal.
```

### Case 4: Private Account Data

```text
Question asks, "What is Acme's current invoice balance?"
Expected: refusal because corpus has general docs only.
```

### Case 5: Multi-Source Answer

```text
Question asks when to escalate a refund request.
Expected: answer uses refund policy and escalation playbook.
```

### Case 6: Citation Hallucination

```text
Model output cites [S9] when only S1-S3 were provided.
Expected: citation validator flags invalid citation.
```

## Acceptance Rubric

### Basic Pass

- loads documents;
- chunks documents;
- builds a retrieval index;
- answers questions from retrieved context;
- includes citations;
- refuses at least obvious unsupported questions.

### Strong Pass

- supports metadata-aware retrieval;
- supports semantic or hybrid retrieval;
- shows retrieved sources and excerpts;
- validates citation IDs;
- separates retrieval failures from answer failures;
- captures user feedback;
- includes eval questions.

### Excellent

- compares chunking strategies;
- reports retrieval top-3 recall;
- reports answer faithfulness or groundedness;
- reports citation accuracy;
- reports refusal accuracy;
- handles seeded failure cases;
- exports retrieval and answer eval reports;
- includes `knowledge-product.md`;
- feedback creates new eval cases.

## Metrics

Measure:

- retrieval top-1 accuracy;
- retrieval top-3 recall;
- forbidden-source rate;
- stale/deprecated-source rate;
- answer relevance;
- groundedness or faithfulness;
- citation validity;
- unsupported claim rate;
- refusal accuracy;
- feedback labels by category.

Suggested targets:

```text
Retrieval top-3 recall: >= 85%
Forbidden-source rate: 0%
Non-refusal answers with citation: 100%
Invalid citation IDs: 0%
Should-refuse accuracy: >= 90%
Unsupported claim rate: <= 5%
```

Do not optimize only for answer fluency.

A fluent unsupported answer is worse than a clear refusal.

## Final Deliverables

At the end, the learner should have:

- runnable CLI, notebook, or local app;
- trusted document corpus;
- chunking/indexing pipeline;
- retrieval layer;
- context builder;
- grounded answer prompt;
- citation validator;
- feedback logger;
- `eval-questions.json`;
- `retrieval-eval-report.md`;
- `answer-eval-report.md`;
- `knowledge-product.md`;
- short project `README.md`.

Example file layout:

```text
project-05-grounded-research-assistant/
  README.md
  knowledge-product.md
  docs/
    policies/
      refunds.md
    product/
      api-keys.md
  data/
    eval-questions.json
  src/
    loader.py
    chunker.py
    index.py
    retrieve.py
    context.py
    answer.py
    citations.py
    feedback.py
    evaluate_retrieval.py
    evaluate_answers.py
  outputs/
    retrieval-eval-report.md
    answer-eval-report.md
```

The exact stack can differ. The grounding behavior should not.

## Demo Script

A 3-minute demo should show:

1. Index the document corpus.
2. Ask a supported question.
3. Show retrieved chunks.
4. Show answer with citations.
5. Ask an unsupported question.
6. Show refusal.
7. Show one citation validation failure case.
8. Run eval.
9. Open retrieval and answer eval reports.

Demo narrative:

```text
This project is a grounded research assistant, not a generic chatbot.
It retrieves evidence, builds context, answers only from sources, cites claims, refuses unsupported questions, and evaluates retrieval separately from answer quality.
The goal is trust, not just fluent output.
```

## What Learner Understands

After completing this project, the learner should understand:

- RAG quality depends on retrieval, context, and generation together;
- embeddings improve semantic retrieval but do not replace metadata and authority;
- chunking decides what the system can find;
- context engineering shapes what the model can answer;
- citations are product controls, not decoration;
- refusal is a feature when evidence is missing;
- retrieval eval and answer eval measure different failures;
- feedback turns an assistant demo into a knowledge product.

## Extension

Later upgrades:

- Part 6: PDF/OCR/table/image/audio document ingestion;
- Part 7: turn grounded answers into safe workflow actions;
- Part 8: add caching, streaming, traces, cost, latency, deployment, rollback;
- Part 9: add retrieval poisoning tests, prompt-injection defenses, access control, privacy, governance.
