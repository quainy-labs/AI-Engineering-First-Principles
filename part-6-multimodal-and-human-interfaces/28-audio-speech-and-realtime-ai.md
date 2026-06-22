# Chapter 28: Audio, Speech, and Realtime AI

Chapter 27 handled visual inputs.

This chapter handles voice, audio, speech-to-text, text-to-speech, and realtime interaction.

Realtime AI is not faster chat. It is interaction under timing pressure. Speech systems must handle transcription errors, turn-taking, interruption, latency, consent, storage, correction, and trust.

## Concept Overview

Audio AI systems turn sound into useful interaction or structured records.

They may include:

- audio capture;
- speech-to-text;
- speaker detection;
- turn detection;
- realtime response;
- text-to-speech;
- summarization;
- extraction;
- action routing;
- transcript correction;
- retention policy.

Common modes:

| Mode | Use When |
|---|---|
| Batch transcription | Meetings, calls, recordings |
| Voice command | Short controlled actions |
| Realtime assistant | Interactive support, coaching, tutoring |
| Text-to-speech | Accessibility or hands-free workflows |
| Call summarization | Post-call records and action items |

The first principle:

> Transcript is not truth. Transcript is a model-produced observation.

Speech can be noisy, accented, interrupted, ambiguous, emotional, or incomplete. System must preserve uncertainty and allow correction.

## Why It Exists

Voice is natural for humans, but hard for systems.

A customer calls support and says:

```text
I was charged twice and need this fixed today.
```

System may need to:

- transcribe speech;
- detect urgency;
- identify billing issue;
- handle interruption;
- summarize call;
- extract action items;
- avoid storing sensitive data unnecessarily;
- route to human when needed.

Bad default:

```text
stream audio to model
-> let model respond freely
-> store full transcript forever
-> use transcript as perfect truth
```

Problems:

- transcription errors change meaning;
- speaker intent may be ambiguous;
- latency breaks conversation;
- interruptions are mishandled;
- sensitive speech enters logs;
- user cannot correct transcript;
- model may speak with false certainty.

This chapter exists because audio systems are interaction systems, not only transcription systems.

## Mental Model

Think of audio AI as a live conversation recorder, interpreter, and assistant.

It hears imperfectly. It may respond before all context is clear. It must manage timing. It must know when to pause, ask clarification, stop speaking, or route to human.

Mental model:

```text
audio stream
-> capture
-> transcribe
-> detect turn
-> interpret intent
-> update state
-> respond or wait
-> summarize/extract
-> user correction
-> durable record
```

For every audio workflow, ask:

```text
Does user know recording is happening?
How accurate must transcript be?
What latency is acceptable?
Can user interrupt?
What should be stored?
What should be redacted?
What needs human review?
How can user correct summary or transcript?
```

Realtime systems fail when they ignore conversation rhythm.

## Internal Mechanics

Audio systems have more moving parts than text systems.

### Capture

Capture quality affects everything downstream.

Consider:

- microphone quality;
- background noise;
- multiple speakers;
- network jitter;
- dropped audio;
- audio format;
- language and accent;
- device permissions.

Poor audio should trigger uncertainty or fallback, not confident automation.

### Consent

Voice data is sensitive.

Design:

```text
Is user informed?
Is consent required?
Is recording stored?
Can user opt out?
Can user delete recording?
Are summaries stored separately from raw audio?
Who can access transcript?
```

Consent and retention are product requirements, not legal footnotes.

### Transcription

Speech-to-text converts audio into transcript.

Transcript errors can change meaning:

```text
"charged twice" -> "charged price"
"cancel plan" -> "cancer plan"
"do not refund" -> "do refund"
```

Transcription should carry:

```text
text
timestamps
speaker label if available
confidence
uncertain spans
language
audio quality warnings
```

Important extracted fields should trace back to transcript timestamps or audio regions.

### Speaker Diarization

Diarization identifies who spoke when.

Useful for:

- meetings;
- support calls;
- interviews;
- multi-party approvals;
- action-item ownership.

Failure risk:

- wrong speaker assigned;
- customer statement attributed to agent;
- approval attributed to wrong person.

High-impact workflows should not trust diarization blindly.

### Turn Detection

Realtime systems need to know when user is done speaking.

Bad turn detection:

- interrupts user too early;
- waits too long;
- speaks over user;
- misses correction;
- treats pause as final intent.

Turn-taking affects trust.

### Interruption Handling

Users interrupt.

System must define:

```text
If user interrupts while model speaks, stop speaking?
Should current action cancel?
Should partial answer be discarded?
Should user correction override previous interpretation?
```

Realtime AI needs stop behavior, not just response behavior.

### Latency

Latency shapes experience.

Different tasks need different budgets:

```text
voice command: very low latency
realtime coaching: low latency
support call summary: can be slower
meeting transcript: batch acceptable
```

Use realtime only when conversation flow matters. Batch transcription is easier to evaluate and often safer.

### Text-to-Speech

Text-to-speech output affects user trust.

Decide:

- voice style;
- speed;
- interruption support;
- whether uncertainty is spoken;
- whether risky actions are read back before approval;
- whether user can switch to text.

Never let confident voice hide uncertain system state.

### Storage and Retention

Audio systems may store:

- raw audio;
- transcript;
- summary;
- extracted fields;
- action items;
- consent record;
- corrections;
- trace.

Retention should be explicit.

For sensitive workflows, store less when possible:

```text
store summary and approved fields
delete raw audio after processing
redact sensitive transcript spans
```

### Correction Path

Users should be able to correct:

- transcript;
- summary;
- extracted fields;
- action items;
- speaker labels;
- intent.

Corrections should become eval data.

## Real-World Example

Use case:

```text
Support call assistant
```

Customer says:

```text
I was charged twice and need this fixed today.
```

Strong system:

```text
1. gets consent for recording/transcription
2. transcribes speech with timestamps and confidence
3. detects billing issue and urgency
4. separates transcript from extracted fields
5. asks clarification if invoice/account ID missing
6. routes refund/payment action to human approval
7. summarizes call
8. lets agent edit summary and action items
9. stores correction and adds failure cases to eval
```

Structured output:

```text
Observed transcript:
"I was charged twice and need this fixed today."

Extracted:
issue_type: billing_duplicate_charge
urgency: high
requested_action: investigate_charge
missing_fields: invoice_id, account_id
needs_human_review: true

Uncertain:
exact invoice not identified

Next step:
ask for invoice/account context or query billing system if permitted
```

Bad system:

```text
Transcript says "charged price"
Model misses duplicate-charge issue
No correction path
Summary stored as truth
```

In audio, correction is not optional. It is how system survives imperfect hearing.

## Common Mistakes and Failure Modes

Mistake 1: Treating transcript as perfect truth

Transcription is model output. It needs confidence, timestamps, and correction path.

Mistake 2: Ignoring consent and retention

Audio may contain sensitive personal, financial, health, or business data.

Mistake 3: Realtime used when batch would be safer

Realtime adds latency, interruption, and state complexity. Use it only when interaction needs it.

Mistake 4: No interruption handling

If user says "stop" or corrects mid-sentence, system must respond predictably.

Mistake 5: No speaker attribution caution

Wrong speaker labels can create false approvals or wrong action ownership.

Mistake 6: Sensitive speech stored in logs

Raw transcripts can leak secrets, account numbers, or private data.

Mistake 7: Model speaks with false certainty

Voice output can sound authoritative even when evidence is weak.

Mistake 8: Corrections do not improve system

Transcript and summary corrections should become eval and improvement data.

## System Design Decisions

Choose interaction mode:

```text
batch transcription
voice command
realtime assistant
call summarization
text-to-speech
```

Define consent and retention:

```text
Is consent required?
Is raw audio stored?
How long are transcripts kept?
What is redacted?
Who can access recordings?
Can user delete data?
```

Define transcription policy:

```text
required accuracy
confidence threshold
timestamp requirement
speaker diarization need
uncertain span handling
manual correction path
```

Define realtime behavior:

```text
latency budget
turn detection
interruption behavior
barge-in support
fallback to text
fallback to human
```

Define extraction and validation:

```text
What fields are extracted?
What evidence timestamps support them?
Which fields need DB/API validation?
Which actions require human approval?
```

Define eval set:

```text
clear speech
noisy speech
accent variation
multiple speakers
interruption
ambiguous command
sensitive information
should-escalate call
long meeting summary
```

The goal is not just hearing user. The goal is reliable interaction.

## Build Artifact

Create `audio-realtime-spec.md`.

Use this template:

```text
Audio Realtime Spec

Workflow:
User goal:

Mode:
- batch transcription / voice command / realtime assistant / text-to-speech

Audio Source:
- source:
  quality requirements:
  fallback:

Consent and Retention:
- consent:
- raw audio storage:
- transcript storage:
- redaction:
- deletion path:

Transcription:
- method:
- confidence rule:
- timestamps:
- diarization:
- uncertain span behavior:

Realtime Behavior:
- latency budget:
- turn detection:
- interruption behavior:
- stop behavior:
- fallback to text/human:

Extraction:
- field:
  evidence timestamp:
  validation:
  human review rule:

Text-to-Speech:
- use case:
- uncertainty wording:
- approval readback:

Correction:
- user can correct:
- correction storage:
- becomes eval: yes/no

Eval Cases:
- case:
  expected behavior:
```

Example:

```text
Workflow: Support call assistant

Mode: call summarization + realtime suggestions

Consent:
announce transcription before call summary starts

Retention:
raw audio deleted after 7 days
summary and approved action items retained with ticket

Interruption:
if agent interrupts assistant suggestion, stop speaking and keep previous action as draft only

Extraction:
issue_type, urgency, requested_action, missing_fields
human review required for refunds, legal, security, or low-confidence transcript
```

Artifact complete when engineer can build speech workflow with timing, consent, correction, and review rules clear.

## Active Recall Questions

1. Why is realtime AI not just faster chat?
2. Why should transcript be treated as observation instead of truth?
3. What makes audio data sensitive?
4. When is batch transcription better than realtime assistant?
5. Why does interruption handling matter?
6. What should be attached to important transcript-derived fields?
7. Why should users correct transcript or summary?
8. Which audio workflows require human review?

## Summary

Audio and realtime AI systems handle speech, transcription, turn-taking, interruption, latency, consent, storage, correction, and trust. They are interaction systems under timing pressure.

Strong systems preserve transcript uncertainty, handle user interruption, protect sensitive audio, validate extracted fields, and let users correct summaries or transcripts.

The most important rule:

> Speech input is evidence under uncertainty, not automatic truth.

## Preview of Next Chapter

Chapter 28 covered audio, speech, and realtime AI.

Chapter 29 brings multimodal systems back to users.

Next, you will learn AI UX, trust, feedback, and human correction. This matters because users need to inspect evidence, correct outputs, approve risky actions, stop long-running work, and understand what system plans to do before automation becomes safe.
