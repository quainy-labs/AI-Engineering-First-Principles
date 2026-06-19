# Project 4: Grounded Research Assistant

## Build

Build a research assistant that answers questions from trusted documents with citations and refuses unsupported questions.

This is a real RAG product, not a chat demo.

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

Use documents from Project 2, or create:

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

Use search engine from Project 2:

- keyword;
- semantic;
- hybrid;
- metadata filters.

### 3. Context Builder

Construct context with:

- title;
- source path;
- last updated;
- authority;
- excerpt.

### 4. Grounded Answer Generator

Rules:

- answer only from context;
- cite source after each claim;
- refuse if evidence missing;
- mention uncertainty;
- never use hidden outside knowledge for final answer.

### 5. Feedback Capture

User can mark:

- useful;
- wrong source;
- unsupported claim;
- missing answer;
- stale source.

### 6. RAG Eval Runner

Run eval set and produce report.

## Architecture

```text
question
-> query processor
-> retriever
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
- retrieval failures visible separately from answer failures.

## What Learner Understands

- RAG quality depends on retrieval and context.
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
