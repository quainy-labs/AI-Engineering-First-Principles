# AI Engineering from First Principles

**Build Reliable Intelligent Systems That Improve Real Work**

> Take a messy real-world workflow, decide whether AI is useful, design a reliable intelligent system, build it, evaluate it, deploy it responsibly, and improve it with feedback.

## Reader Promise

After finishing this book, reader should be able to:

- analyze a real workflow;
- decide when not to use AI;
- design system boundaries;
- organize data, documents, and state;
- use models as components;
- build grounded retrieval systems;
- connect AI to tools and workflows;
- evaluate quality with real examples;
- reason about cost, latency, security, and risk;
- ship proof of useful capability.

## Book Shape

- Core structure updated.
- Parts 1-9 are ordered from first principles.
- Remaining work is polish: cross-links, examples, diagrams, and copy tightening.
- Target: full AI engineering path from problem to software, data, models, retrieval, multimodal interfaces, tools, agents, production, security, governance, product economics, and capstone.
- Serious build projects after each major capability.
- 1 capstone.

Each chapter ends with a proof artifact. No chapter ends with only "understanding."

## Projects

[Project Guide](projects/README.md)

## Table of Contents

### Part 1: Problem-First AI Engineering

Capability: decide what to build and why.

1. [AI Engineering Is System Design](part-1-problem-first-ai-engineering/01-ai-engineering-is-system-design.md)
2. [When Not to Use AI](part-1-problem-first-ai-engineering/02-when-not-to-use-ai.md)
3. [From Messy Workflow to System Boundary](part-1-problem-first-ai-engineering/03-from-messy-workflow-to-system-boundary.md)
4. [Metrics, Baselines, and Meaningful Impact](part-1-problem-first-ai-engineering/04-metrics-baselines-meaningful-impact.md)

Project: [Support Workflow Intelligence Console](projects/01-workflow-teardown.md)

### Part 2: Software Foundations for AI Systems

Capability: understand where AI components live inside real software.

5. [AI Application Architecture](part-2-software-foundations-for-ai-systems/05-ai-application-architecture.md)
6. [APIs, Services, Workers, and Queues](part-2-software-foundations-for-ai-systems/06-apis-services-workers-queues.md)
7. [Auth, Permissions, Files, and Storage](part-2-software-foundations-for-ai-systems/07-auth-permissions-files-storage.md)
8. [Async Jobs, Background Processing, and Webhooks](part-2-software-foundations-for-ai-systems/08-async-jobs-background-processing-webhooks.md)

Project: [AI App Skeleton](projects/02-ai-app-skeleton.md)

### Part 3: Data and Knowledge Foundations

Capability: turn messy knowledge into usable system context.

9. [Data, Documents, and State](part-3-data-and-knowledge-foundations/09-data-documents-and-state.md)
10. [Data Ingestion, Parsing, and Cleaning](part-3-data-and-knowledge-foundations/10-data-ingestion-parsing-and-cleaning.md)
11. [Schemas, Contracts, and Structured Outputs](part-3-data-and-knowledge-foundations/11-schemas-contracts-and-structured-outputs.md)
12. [Data Quality, Lineage, and Versioning](part-3-data-and-knowledge-foundations/12-data-quality-lineage-and-versioning.md)
13. [Search, Ranking, and Information Architecture](part-3-data-and-knowledge-foundations/13-search-ranking-and-information-architecture.md)

Project: [Knowledge Base Search Engine](projects/03-knowledge-base-search-engine.md)

### Part 4: Models as Components

Capability: use models with understanding, not worship.

14. [What Language Models Actually Do](part-4-models-as-components/14-what-language-models-actually-do.md)
15. [Tokens, Context, Attention, and Generation](part-4-models-as-components/15-tokens-context-attention-and-generation.md)
16. [Prompting as Interface Design](part-4-models-as-components/16-prompting-as-interface-design.md)
17. [Model Selection, Routing, and Fallbacks](part-4-models-as-components/17-model-selection-routing-and-fallbacks.md)
18. [Fine-Tuning, Adaptation, and When Not to Tune](part-4-models-as-components/18-fine-tuning-adaptation-and-when-not-to-tune.md)
19. [Model Failure Modes and Error Analysis](part-4-models-as-components/19-model-failure-modes-and-error-analysis.md)

Project: [Structured Intake Extractor](projects/04-structured-intake-extractor.md)

### Part 5: Retrieval and Context Systems

Capability: build systems that answer from trusted knowledge.

20. [Embeddings and Semantic Retrieval](part-5-retrieval-and-context-systems/20-embeddings-and-semantic-retrieval.md)
21. [Chunking, Indexing, and Retrieval Pipelines](part-5-retrieval-and-context-systems/21-chunking-indexing-and-retrieval-pipelines.md)
22. [Retrieval-Augmented Generation](part-5-retrieval-and-context-systems/22-retrieval-augmented-generation.md)
23. [Context Engineering and Grounding](part-5-retrieval-and-context-systems/23-context-engineering-and-grounding.md)
24. [Evaluating Grounded AI Systems](part-5-retrieval-and-context-systems/24-evaluating-grounded-ai-systems.md)
25. [From Assistant Demo to Knowledge Product](part-5-retrieval-and-context-systems/25-from-assistant-demo-to-knowledge-product.md)

Project: [Grounded Research Assistant](projects/05-grounded-research-assistant.md)

### Part 6: Multimodal and Human Interfaces

Capability: handle documents, images, audio, realtime inputs, and human correction loops.

26. [Document AI: OCR, Tables, Forms, and PDFs](part-6-multimodal-and-human-interfaces/26-document-ai-ocr-tables-forms-and-pdfs.md)
27. [Vision, Image, and Multimodal Inputs](part-6-multimodal-and-human-interfaces/27-vision-image-and-multimodal-inputs.md)
28. [Audio, Speech, and Realtime AI](part-6-multimodal-and-human-interfaces/28-audio-speech-and-realtime-ai.md)
29. [AI UX, Trust, Feedback, and Human Correction](part-6-multimodal-and-human-interfaces/29-ai-ux-trust-feedback-and-human-correction.md)

Project: [Multimodal Document Analyst](projects/06-multimodal-document-analyst.md)

### Part 7: Tools, Workflows, and Agents

Capability: build AI systems that take bounded action.

30. [Tool Use and API Actions](part-7-tools-workflows-and-agents/30-tool-use-and-api-actions.md)
31. [Connectors, MCP, and Tool Registries](part-7-tools-workflows-and-agents/31-connectors-mcp-and-tool-registries.md)
32. [Workflow Orchestration](part-7-tools-workflows-and-agents/32-workflow-orchestration.md)
33. [Memory, State, and Human Approval](part-7-tools-workflows-and-agents/33-memory-state-and-human-approval.md)
34. [Computer Use and Browser Automation](part-7-tools-workflows-and-agents/34-computer-use-and-browser-automation.md)
35. [Agents Without Hype](part-7-tools-workflows-and-agents/35-agents-without-hype.md)
36. [Agent Evaluation and Trace Review](part-7-tools-workflows-and-agents/36-agent-evaluation-and-trace-review.md)

Project: [Operations Action Assistant](projects/07-operations-action-assistant.md)

### Part 8: Reliable Production AI

Capability: measure, observe, deploy, and improve AI systems without guessing.

37. [Evaluation as Product Infrastructure](part-8-reliable-production-ai/37-evaluation-as-product-infrastructure.md)
38. [Observability, Traces, Cost, and Latency](part-8-reliable-production-ai/38-observability-traces-cost-and-latency.md)
39. [Caching, Streaming, Batching, and Inference Optimization](part-8-reliable-production-ai/39-caching-streaming-batching-and-inference-optimization.md)
40. [LLMOps: Versioning Prompts, Models, Evals, Data, and Indexes](part-8-reliable-production-ai/40-llmops-versioning-prompts-models-evals-data-and-indexes.md)
41. [Deployment, Rollbacks, Feedback, and Continuous Improvement](part-8-reliable-production-ai/41-deployment-rollbacks-feedback-and-continuous-improvement.md)

Project: [Production AI Control Center](projects/08-production-ai-control-center.md)

### Part 9: Security, Governance, and Product Reality

Capability: make AI systems safe, governed, economically justified, and worth keeping.

42. [AI Security Threat Model](part-9-security-governance-and-product-reality/42-ai-security-threat-model.md)
43. [Prompt Injection, Retrieval Poisoning, and Tool Abuse](part-9-security-governance-and-product-reality/43-prompt-injection-retrieval-poisoning-and-tool-abuse.md)
44. [Privacy, PII, Data Retention, and Access Control](part-9-security-governance-and-product-reality/44-privacy-pii-data-retention-and-access-control.md)
45. [Responsible AI, Red Teaming, and Risk Management](part-9-security-governance-and-product-reality/45-responsible-ai-red-teaming-and-risk-management.md)
46. [AI Product Economics: Cost, ROI, Build-vs-Buy](part-9-security-governance-and-product-reality/46-ai-product-economics-cost-roi-build-vs-buy.md)

Project: [AI Risk and Governance Review](projects/09-ai-risk-and-governance-review.md)

## Capstone

[Build One Real Intelligent System](capstone.md)

Final proof: choose one real workflow, build a narrow intelligent system, evaluate it, analyze failure, document risk, and show deployment readiness.
