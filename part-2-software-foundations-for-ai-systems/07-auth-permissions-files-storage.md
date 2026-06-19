# Chapter 7: Auth, Permissions, Files, and Storage

Chapter 6 showed how work moves through APIs, workers, and queues. This chapter asks what the system is allowed to access and where information lives.

AI systems often fail by retrieving too much, remembering too much, or exposing data to wrong user.

## Real Problem

Internal assistant answers:

> The salary adjustment policy is...

Problem: user is intern. That document was HR-only.

Model did not "hack" system. Architecture allowed unauthorized retrieval.

## Bad Default Solution

Bad design:

- ingest all documents into one index;
- retrieve before permission check;
- rely on prompt: "do not reveal restricted info";
- store raw uploaded files without owner;
- log sensitive data;
- give tools same access for every user.

Prompt is not permission system.

## First Principles

Auth answers:

> Who is user?

Permission answers:

> What can this user access or do?

Storage answers:

> Where does information live and who owns it?

AI system needs permission checks before:

- document retrieval;
- database lookup;
- tool execution;
- file download;
- memory access;
- log viewing.

## System Decision

Define resources:

- users;
- organizations;
- documents;
- files;
- tickets;
- tool actions;
- memories;
- logs.

For each resource:

```text
Owner:
Access roles:
Read permission:
Write permission:
Delete permission:
Retention:
Sensitive fields:
```

Retrieval rule:

```text
authorize user -> filter allowed sources -> retrieve -> answer
```

Never:

```text
retrieve all -> ask model not to leak
```

## Minimal Build

Create `access-and-storage-model.md`.

Template:

```text
Users:
- role:
  permissions:

Resources:
- name:
  storage:
  owner:
  read roles:
  write roles:
  retention:
  sensitive fields:

File storage:
- upload path:
- allowed types:
- size limits:
- scanning:
- ownership:

Retrieval access:
- pre-filter:
- metadata required:
- forbidden sources:

Logs:
- redaction:
- retention:
- viewer roles:
```

## Failure Modes

Access/storage fails when:

- one shared vector index ignores permissions;
- metadata lacks owner/access level;
- logs store secrets;
- uploaded files unscanned;
- deleted documents remain indexed;
- model can call tools beyond user role;
- workers process files after permission revoked;
- user cannot delete personal data.

## Evaluation

Test:

- unauthorized document query;
- authorized document query;
- role change;
- deleted document;
- restricted tool action;
- sensitive data in logs;
- file type rejection;
- retention expiry.

Pass if unauthorized content never reaches model context.

## Improvement

Improve by:

- adding access metadata;
- permission-filtering before retrieval;
- separate indexes per tenant when needed;
- redacting logs;
- scanning uploads;
- enforcing role checks in backend;
- deleting indexed chunks when source deleted.

## Proof Artifact

Create `access-and-storage-model.md`.

Include:

- roles;
- resources;
- storage choices;
- permission rules;
- file rules;
- log redaction;
- access tests.

## Decision Checklist

- [ ] Auth exists before sensitive action.
- [ ] Retrieval filters by permission before model call.
- [ ] Documents have owner/access metadata.
- [ ] Files have storage and retention rules.
- [ ] Logs redact sensitive data.
- [ ] Tool permissions match user role.
- [ ] Deleted docs leave indexes.

## Active Recall

1. Why is prompt not permission system?
2. What is difference between auth and permission?
3. Why filter before retrieval?
4. What metadata does document need for access control?
5. How can logs leak private data?

## Next

Access and storage define what system can use. Next chapter handles events over time: background jobs, webhooks, retries, and state changes.

