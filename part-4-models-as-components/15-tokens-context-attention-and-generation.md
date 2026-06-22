# Chapter 15: Tokens, Context, Attention, and Generation

Chapter 14 bounded the model's role:

```text
system supplies trusted context
-> model performs bounded language task
-> system validates output
-> system decides next action
```

This chapter explains the mechanics that shape that role.

You do not need to become a machine learning researcher to build good AI systems. But you do need enough first principles to design context, cost, latency, reliability, and failure behavior.

The model does not read like a human. It processes tokens inside a context window, relates those tokens through attention, and generates output tokens one at a time.

That has engineering consequences.

## Concept Overview

Tokens are the units of text the model processes.

Context is the information placed into the model request.

Attention is the mechanism that lets the model relate parts of the context to other parts.

Generation is the process of producing output tokens.

Together, they answer:

```text
How much information can fit?
What does the model see?
What does the model ignore or dilute?
Why does cost increase?
Why does latency increase?
Why can more context make answers worse?
Why can small context changes change output?
How should context be assembled?
```

This chapter turns model mechanics into system design decisions.

The first principle:

> More context is not automatically better. Better selected context is better.

## Why It Exists

Imagine your assistant becomes worse after you add more documents to the prompt.

You expected:

```text
more documents -> more knowledge -> better answer
```

Instead:

- the answer becomes vague;
- citations point to weaker sources;
- important evidence gets ignored;
- stale policy appears beside current policy;
- latency increases;
- cost increases;
- output becomes less consistent.

This happens because context is not a dumping ground.

A language model receives a limited sequence of tokens. If the context contains irrelevant, conflicting, stale, or poorly labeled information, the model must generate from that noisy mixture.

Bad default:

```text
Put everything in the prompt.
```

This creates:

- noisy evidence;
- buried important facts;
- conflicting instructions;
- higher token cost;
- slower responses;
- weaker grounding;
- larger attack surface;
- less predictable output.

This chapter exists because context design is one of the core skills of AI engineering.

## Mental Model

Think of the model request as a meeting packet.

If you give someone a focused packet with the task, relevant facts, source labels, decision rules, and output format, they can help.

If you give them every email, every document, stale policies, old chat history, contradictory notes, and unclear instructions, they may still sound confident, but their answer will be less reliable.

The mental model:

```text
token budget
-> context sections
-> relevance selection
-> source labeling
-> conflict handling
-> output allowance
-> generation
```

Every model request is a design object.

It should answer:

```text
What must the model know?
What must the model ignore?
What is trusted?
What is user-provided?
What is retrieved evidence?
What is workflow state?
What is the output contract?
How many tokens can the answer use?
```

Context should be assembled, not accumulated.

## Internal Mechanics

Models process tokens, not pages, files, or human concepts directly.

A short sentence:

```text
Reset the API key.
```

may become tokens roughly like:

```text
["Reset", " the", " API", " key", "."]
```

Tokenization differs by model and language. Some words are one token. Some words split into multiple tokens. Code, tables, numbers, unusual names, and non-English text can tokenize differently.

Token count affects:

- cost;
- latency;
- context capacity;
- retrieval budget;
- conversation history budget;
- number of examples that fit;
- output length.

The context window is the maximum number of tokens the model can consider in one request.

Context usually includes:

- system instructions;
- developer or application instructions;
- user message;
- retrieved documents;
- database facts;
- tool results;
- examples;
- conversation history;
- workflow state;
- output schema;
- expected output tokens.

A common mistake is forgetting output allowance.

If a model has a context limit, the request must leave room for the answer. A full input context can crowd out useful output or force truncation.

Context budget should include:

```text
instructions
task
user input
trusted facts
retrieved evidence
tool results
conversation history
examples
output allowance
```

Attention lets the model relate tokens across the context.

Engineering lesson:

> The model's behavior depends on what context is present, where it appears, and how clearly relevance is signaled.

Attention is powerful, but it is not a guarantee that every token is used equally. Important evidence can be diluted when surrounded by irrelevant material. Conflicting sources can pull the answer in different directions. Long histories can distract from the current task.

Generation happens token by token.

Each generated token depends on:

- model weights;
- instructions;
- supplied context;
- retrieved evidence;
- tool results;
- output contract;
- previous generated tokens;
- decoding or sampling settings.

This explains why small changes can matter.

Adding one stale paragraph, changing source order, including old conversation history, or mixing facts with instructions can change the output.

Context sections should be separated by role and purpose.

Example:

```text
System rules:
- follow permission and safety rules
- answer only from supplied evidence

Task:
- draft a support reply

Trusted account facts:
- plan: enterprise
- duplicate charge: under review

Retrieved policy:
- title
- source
- updated_at
- excerpt

User message:
- raw customer message

Output contract:
- draft_reply
- cited_sources
- missing_information
- needs_human_review
```

This is stronger than mixing all text into one blob.

Generation settings also matter.

Lower randomness is often useful for:

- extraction;
- classification;
- structured output;
- policy-grounded answers;
- repeatable workflows.

Higher randomness may be useful for:

- brainstorming;
- creative alternatives;
- style variation;
- early exploration.

For production AI systems, generation should usually be constrained by:

- clear output schema;
- relevant context;
- validation;
- deterministic checks;
- fallback paths.

Do not use randomness to compensate for unclear task design.

## Real-World Example

Imagine a support assistant that drafts replies from company policy.

User asks:

> Can this enterprise customer rotate API keys without downtime?

Weak context assembly:

```text
System: Be helpful.
User: Can this enterprise customer rotate API keys without downtime?
Context:
- old API key FAQ from 2023
- enterprise workspace guide from 2026
- internal migration notes
- unrelated billing policy
- previous conversation about refunds
- draft docs from product team
```

The answer may be vague or wrong because the model sees too much mixed context.

Better context assembly:

```text
System instruction:
- answer only from supplied evidence
- prefer canonical active docs
- do not use draft or internal-only docs for customer-facing answer

Task:
- draft a support reply for an enterprise admin

Trusted facts:
- customer_plan: enterprise
- workspace_type: enterprise

Retrieved evidence:
1. Enterprise Workspace API Key Management
   status: active
   authority: canonical
   updated: 2026-05-10
   section: Rotating keys without downtime
   excerpt: ...

2. Enterprise Admin Security Guide
   status: active
   authority: supporting
   updated: 2026-04-18
   section: Key overlap window
   excerpt: ...

Excluded:
- API Key Rotation Draft
- Internal Security Escalation Policy
- old 2023 API FAQ

Output:
- customer-facing answer
- cited source titles
- missing information if any
```

The model now has less context, but better context.

This improves:

- grounding;
- citation quality;
- latency;
- cost;
- user trust;
- debugging.

## Common Mistakes and Failure Modes

Mistake 1: Adding more context when retrieval is bad

If retrieval returns weak sources, adding more weak sources creates more noise. Improve retrieval and ranking first.

Mistake 2: No token budget

Without a budget, context grows until cost, latency, and quality become unpredictable.

Mistake 3: Forgetting output allowance

The system must reserve tokens for the answer, especially for structured outputs, long summaries, or citations.

Mistake 4: Mixing instructions and evidence

User text, retrieved documents, tool results, and system rules should be clearly separated. A document should not be allowed to act like an instruction.

Mistake 5: Including stale conversation history

Old chat turns can distract the model or revive decisions that should no longer apply.

Mistake 6: Missing source labels

Evidence without title, source, authority, and freshness is harder for the model to use correctly and harder for the system to audit.

Mistake 7: Conflicting sources with no conflict rule

If one document says 30 days and another says 60 days, the system must define which source wins.

Mistake 8: Treating attention as perfect memory

The model may miss or underuse important evidence when context is long, noisy, or poorly organized.

## System Design Decisions

Design context before tuning prompts.

Define context sections:

```text
system instruction
task instruction
user input
trusted facts
retrieved evidence
tool results
workflow state
conversation history
examples
output contract
```

Define token budget:

```text
maximum input tokens
reserved output tokens
retrieved evidence budget
conversation history budget
tool result budget
examples budget
```

Define inclusion rules:

```text
What must always be included?
What should be included only when relevant?
What should be summarized?
What should be excluded?
```

Define ordering:

```text
Where do instructions go?
Where do trusted facts go?
Where does retrieved evidence go?
Should strongest evidence appear first?
Should citations stay near excerpts?
```

Define compression:

```text
When should long documents be summarized?
What information must summaries preserve?
Can summaries be trusted?
Should source excerpts be kept for citations?
```

Define conflict handling:

```text
Which source wins when documents conflict?
What happens when retrieved evidence is stale?
What happens when database facts conflict with documents?
Should the model answer or escalate?
```

Define generation behavior:

```text
Is output free text or structured?
How long should output be?
Should citations be required?
Should uncertainty be explicit?
Should unsupported claims be forbidden?
```

Context design should make the correct answer easier than the wrong answer.

## Build Artifact

Create `context-budget.md` for one model task.

Choose a task such as:

- support reply drafting;
- ticket classification;
- policy question answering;
- invoice extraction;
- document summarization;
- tool action proposal.

Use this template:

```text
Context Budget

Workflow:
Model task:
Model role:

Token Budget:
- maximum context window:
- maximum input tokens:
- reserved output tokens:
- system instruction budget:
- task instruction budget:
- user input budget:
- trusted facts budget:
- retrieved evidence budget:
- tool results budget:
- conversation history budget:
- examples budget:

Context Sections:
- section:
  purpose:
  source:
  max tokens:
  required: yes/no

Must Include:
- item:
  reason:

May Include:
- item:
  condition:

Must Exclude:
- item:
  reason:

Ordering:
- rule:
  reason:

Compression:
- source:
  compression rule:
  must preserve:

Conflict Rule:
- conflict:
  winning source:
  fallback:

Generation:
- output format:
- max output length:
- citation rule:
- uncertainty rule:

Stress Tests:
- test:
  expected behavior:
```

Example:

```text
Workflow: Support reply assistant
Model task: Draft response from approved policy and account facts

Token Budget:
- maximum context window: 8,000 tokens
- reserved output tokens: 800
- retrieved evidence budget: 2,500
- conversation history budget: 500
- trusted facts budget: 500

Must Include:
- current ticket message
- live account facts
- top approved policy snippets
- output contract

Must Exclude:
- stale policy docs
- internal-only docs for customer-facing reply
- unrelated previous tickets

Conflict Rule:
- live account API beats document examples for account-specific facts
- newest approved policy beats older policy

Stress Test:
- include relevant policy plus stale policy
- expected: answer follows newest approved policy and cites it
```

The artifact is complete when another engineer can assemble the model request without guessing what to include, what to exclude, and how much room to reserve for output.

## Active Recall Questions

1. What is a token, and why does token count matter?
2. What counts toward the context window?
3. Why can adding more context reduce answer quality?
4. Why should output tokens be reserved in the budget?
5. Why should instructions, user text, retrieved evidence, and tool results be separated?
6. What is the difference between more context and better context?
7. How can stale conversation history harm model behavior?
8. What should a context stress test measure?

## Summary

Language models process tokens inside a limited context window, relate those tokens through attention, and generate output tokens step by step. These mechanics shape cost, latency, reliability, grounding, and failure behavior.

Context should be designed deliberately. Include the right instructions, trusted facts, retrieved evidence, state, examples, and output contract. Exclude stale, irrelevant, unauthorized, or conflicting material unless the task requires handling conflict.

The most important rule:

> Context is not storage. Context is selected evidence for the current task.

## Preview of Next Chapter

Chapter 15 showed that model behavior depends on context design and generation mechanics.

Chapter 16 reframes prompting as interface design.

Next, you will learn why a prompt is not magic wording. It is the model-facing interface between software and the model: task definition, input fields, allowed sources, output schema, refusal behavior, validation, examples, and evaluation.
