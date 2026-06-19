# Project 5: Grounded Research Assistant

## Build

Build a research assistant that answers questions from trusted documents with citations and refuses unsupported questions.

This is a real RAG product, not a chat demo.

This project belongs after Part 5. It should prove that learner can build the full retrieval and context path: embeddings, hybrid retrieval, chunking, metadata, RAG, grounding, evaluation, and knowledge-product feedback loops.

## User

Student, researcher, support teammate, or internal operator who needs answers from a controlled document set.

## Product Outcome

User asks a question. System returns:

- answer;
- citations;
- retrieved sources;
- confidence/limits;
- refusal when unsupported;
- feedback buttons;
- eval report.

## Sample Corpus

Use documents from Project 3, or create:

```text
docs/
  policies/refunds.md
  policies/access-control.md
  product/api-keys.md
  product/billing-admins.md
  support/escalation-playbook.md
```

## Core Features

### 1. Question Interface

CLI or web:

```bash
ask "How do I reset an API key?"
```

### 2. Retrieval Layer

Use search engine from Project 3:

- keyword;
- semantic;
- hybrid;
- metadata filters.

### 3. Indexing Pipeline

Build:

- parser;
- chunker;
- metadata attachment;
- keyword index;
- vector index;
- reindex command;
- stale/deleted document handling.

### 4. Context Builder

Construct context with:

- title;
- source path;
- last updated;
- authority;
- excerpt;
- retrieval reason.

### 5. Grounded Answer Generator

Rules:

- answer only from context;
- cite source after each claim;
- refuse if evidence missing;
- mention uncertainty;
- never use hidden outside knowledge for final answer.

### 6. Citation Validator

Check:

- every factual claim has citation;
- citation source appears in retrieved context;
- cited excerpt supports claim;
- unsupported answer becomes refusal or review.

### 7. Feedback Capture

User can mark:

- useful;
- wrong source;
- unsupported claim;
- missing answer;
- stale source.

### 8. RAG Eval Runner

Run eval set and produce report.

## Architecture

```text
question
-> query processor
-> keyword/vector retrievers
-> metadata filter
-> reranker
-> context builder
-> answer generator
-> citation validator
-> feedback logger
-> eval runner
```

## Data Model

```json
{
  "question_id": "Q001",
  "question": "How do I reset an API key?",
  "retrieved_sources": [
    {
      "doc_id": "product/api-keys.md",
      "title": "API Key Management",
      "excerpt": "To reset an API key...",
      "heading": "Reset Process",
      "last_updated": "2026-01-10",
      "authority": "approved_doc",
      "score": 0.91
    }
  ],
  "answer": "To reset an API key...",
  "citations": ["product/api-keys.md#reset-process"],
  "status": "answered"
}
```

## Evaluation

Use 40 questions:

- 15 easy;
- 15 normal;
- 5 edge;
- 5 should-refuse.

Measure:

- retrieval top-3 recall;
- stale-source rate;
- forbidden-source rate;
- answer relevance;
- faithfulness;
- citation accuracy;
- refusal accuracy;
- unsupported claim rate;
- p95 latency;
- cost per answer.

Acceptance:

- unsupported questions refused;
- every answer has citation;
- citations point to relevant source;
- retrieval failures visible separately from answer failures;
- feedback creates new eval cases.

## What Learner Understands

- RAG quality depends on retrieval and context.
- Embeddings help meaning search but do not replace metadata and authority.
- Chunking and indexing decide what can be retrieved.
- Grounding requires source rules.
- Refusal is feature, not failure.
- Knowledge product needs feedback loop.

## Extension

Add:

- source admin page;
- stale document warnings;
- answer comparison across models;
- citation validator;
- eval dashboard;
- query rewrite.
