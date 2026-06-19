# Chapter 44: Privacy, PII, Data Retention, and Access Control

Chapter 43 handled manipulation attacks. This chapter handles sensitive data.

AI systems often touch customer records, documents, transcripts, screenshots, prompts, traces, and tool results. Privacy must be designed before data flows.

## First Principles

Classify data:

- public;
- internal;
- confidential;
- regulated;
- personal;
- secret.

Design:

- least privilege;
- access before retrieval;
- redaction;
- retention limits;
- deletion path;
- audit logs;
- user consent when needed.

## Minimal Build

Create `privacy-and-access-policy.md`:

```text
Data classes:
PII fields:
Allowed storage:
Forbidden storage:
Retention:
Deletion:
Access roles:
Redaction:
Audit:
```

## Failure Modes

Privacy fails when:

- logs store raw sensitive data;
- model sees documents user cannot access;
- traces are visible to wrong team;
- retention is indefinite;
- deletion does not remove derived data;
- screenshots/transcripts are treated as harmless.

## Proof Artifact

Create `privacy-and-access-policy.md`.

## Next

Next chapter covers responsible AI, red teaming, and risk management.
