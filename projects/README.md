# Projects

These are guided build projects, not chapter exercises.

Each project appears after a major course part and should be buildable using only the concepts taught up to that point. Later topics are listed as extensions, not requirements.

The goal is not to complete worksheets. The goal is to build inspectable systems that prove AI engineering judgment.

## Project Path

| Part | Chapters | Project | Capability Proved |
|---|---:|---|---|
| Part 1 | 1-4 | [Support Workflow Intelligence Console](01-workflow-teardown.md) | Choose meaningful AI work from workflow evidence |
| Part 2 | 5-8 | [AI App Skeleton](02-ai-app-skeleton.md) | Place AI behavior inside real software architecture |
| Part 3 | 9-13 | [Knowledge Base Search Engine](03-knowledge-base-search-engine.md) | Build document, metadata, quality, lineage, and search foundations |
| Part 4 | 14-19 | [Structured Intake Extractor](04-structured-intake-extractor.md) | Use models as bounded components with schema validation and error analysis |
| Part 5 | 20-25 | [Grounded Research Assistant](05-grounded-research-assistant.md) | Build grounded retrieval and cited answer systems |
| Part 6 | 26-29 | [Multimodal Document Analyst](06-multimodal-document-analyst.md) | Extract structured records from documents, images, tables, and human correction |
| Part 7 | 30-36 | [Operations Action Assistant](07-operations-action-assistant.md) | Connect AI to tools, workflow state, approval, and trace review |
| Part 8 | 37-41 | [Production AI Control Center](08-production-ai-control-center.md) | Operate AI with evals, traces, cost, latency, versions, rollback, and feedback |
| Part 9 | 42-46 | [AI Risk and Governance Review](09-ai-risk-and-governance-review.md) | Review security, privacy, responsible AI, economics, and launch decision |

After these, complete the [Capstone](../capstone.md).

## Project Standard

Every project should produce one of:

- runnable CLI;
- reproducible notebook;
- local API service;
- local web app;
- small full-stack prototype;
- inspectable review package, for Project 9.

Every project should include:

- concrete user;
- product goal;
- starter data;
- MVP milestone;
- V1 milestone;
- clear system boundary;
- core components;
- data model;
- seeded failure cases;
- evaluation dataset or review evidence;
- acceptance rubric;
- final deliverables;
- demo script.

## Scope Rule

Projects must not depend on future parts.

Examples:

- Project 1 should not require app architecture, models, retrieval, or production observability.
- Project 3 should not require embeddings or RAG.
- Project 5 may require RAG, but should not require tools or agents.
- Project 8 should not require security/governance review.
- Project 9 may combine the full Part 9 security, governance, and economics surface.

Later capabilities can improve earlier projects, but they should appear as extensions.

## How To Use These Projects

For each project:

1. Read the project spec.
2. Build the MVP first.
3. Add V1 only after MVP works.
4. Include seeded failure cases.
5. Run the evaluation or review.
6. Produce final deliverables.
7. Record a short demo.
8. Explain one important tradeoff.

Do not start by choosing a model.

Start by understanding the workflow, boundary, data, output contract, risk, and evaluation.

## Portfolio Evidence

A strong learner portfolio should show:

- what problem the project solves;
- what input data was used;
- how the system is structured;
- where AI is used and where it is not used;
- how bad outputs are caught;
- how failure cases are evaluated;
- what tradeoffs were made;
- what would be improved next.

The best project demos show both:

- a happy path;
- a failure path handled safely.

## Capstone Bridge

The capstone is learner-chosen.

Use the guided projects as building blocks:

- Project 1 helps choose the workflow.
- Project 2 provides the software skeleton.
- Project 3 provides knowledge foundations.
- Project 4 provides model interface and validation.
- Project 5 provides grounded retrieval.
- Project 6 provides multimodal extraction.
- Project 7 provides tools, approvals, and agent traces.
- Project 8 provides production readiness.
- Project 9 provides risk, governance, and economics review.

Capstone standard:

```text
one narrow real workflow
  -> baseline
  -> architecture
  -> data and model/retrieval/tool design
  -> runnable prototype
  -> evals and error analysis
  -> traces and deployment plan
  -> security/risk/economics review
  -> demo and proof of capability
```
