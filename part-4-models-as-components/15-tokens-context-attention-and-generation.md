# Chapter 15: Tokens, Context, Attention, and Generation

Chapter 14 bounded the model role. This chapter explains the model mechanics that affect that boundary.

You do not need to become a researcher. You need enough first principles to design context, cost, latency, and failure behavior.

## Real Problem

Your assistant gives worse answers after you add more documents to context.

You expected more context to improve the answer.

Instead:

- answer becomes vague;
- citations point to weaker source;
- important snippet gets ignored;
- latency increases;
- cost increases.

Why?

Because a language model does not "read documents" like a human. It processes tokens inside limited context, attends across them, then generates output tokens.

## Bad Default Solution

Bad solution:

> Put everything in the prompt.

This creates:

- noisy context;
- high cost;
- slow response;
- conflicting instructions;
- buried evidence;
- weak grounding.

More context is not always better. Better context is better.

## First Principles

### Tokens

Models do not directly see words. They see tokens.

Text:

```text
Reset the API key.
```

becomes tokens roughly like:

```text
["Reset", " the", " API", " key", "."]
```

Token count affects:

- cost;
- latency;
- context capacity;
- how much evidence fits;
- how much output can be generated.

### Context Window

Context window is maximum tokens the model can consider in one request.

Context includes:

- system instructions;
- developer instructions;
- user message;
- retrieved documents;
- tool results;
- examples;
- conversation history;
- expected output tokens.

If context window is full, the system must choose what to include and what to remove.

### Attention

Attention lets model relate tokens to other tokens in context.

Engineering lesson:

> Model behavior depends on what context is present, where it appears, and how clear its relevance is.

Attention is powerful but not a guarantee. Important evidence can be diluted by irrelevant context.

### Generation

Model generates output token by token.

Each next token depends on:

- input context;
- instructions;
- model weights;
- sampling settings;
- prior generated tokens.

This is why small changes in context can change output.

## System Decision

Design context deliberately:

```text
Instruction:
Task:
Output contract:
User input:
Trusted facts:
Retrieved evidence:
Conversation state:
Refusal rule:
```

Separate:

- facts from instructions;
- user text from system rules;
- retrieved sources from generated text;
- current task state from old conversation.

Never treat context as dumping ground.

## Minimal Build

Create `context-budget.md`:

```text
Task:
Max context budget:

Sections:
- system instruction: target tokens
- user input: target tokens
- retrieved context: target tokens
- examples: target tokens
- tool results: target tokens
- conversation history: target tokens
- output allowance: target tokens

Must include:
-

May include:
-

Must exclude:
-

Compression rule:
Truncation rule:
Conflict rule:
```

Example:

```text
Task: Draft support answer
Max context budget: 8,000 tokens
Retrieved context: 3,000 tokens max
Conversation history: only current ticket
Must exclude: stale policy docs
Conflict rule: newest approved policy wins
```

## Failure Modes

Context design fails when:

- too many documents are included;
- retrieved snippets lose title/source;
- conversation history includes irrelevant old turns;
- user text can override system rules;
- examples conflict with instructions;
- output budget is forgotten;
- context includes stale facts;
- tool results are not separated from user claims.

## Evaluation

Create context stress tests:

- relevant evidence only;
- relevant plus irrelevant;
- conflicting sources;
- stale source;
- missing evidence;
- long user message;
- malicious instruction inside document;
- long conversation history.

Measure:

- answer grounded in included evidence;
- irrelevant distractor resistance;
- refusal when evidence missing;
- authority order followed;
- latency;
- cost.

## Improvement

Improve context by:

- reducing top-k;
- improving chunking;
- adding source titles;
- reranking;
- summarizing long context;
- removing stale docs;
- separating instructions from evidence;
- adding conflict policy.

## Proof Artifact

Create `context-budget.md`.

Include:

- token budget;
- context sections;
- inclusion rules;
- exclusion rules;
- compression rule;
- conflict rule;
- stress tests.

## Decision Checklist

- [ ] Context budget exists.
- [ ] Output token budget included.
- [ ] Retrieved evidence has source labels.
- [ ] Conversation history is scoped.
- [ ] Stale context is excluded.
- [ ] Conflict rule exists.
- [ ] Context stress tests exist.

## Active Recall

1. What is token?
2. What counts toward context window?
3. Why can more context reduce answer quality?
4. What is context budget?
5. Why separate evidence from instructions?

## Next

Next chapter reframes prompting. Prompting is not magic words. It is interface design between software and model.
