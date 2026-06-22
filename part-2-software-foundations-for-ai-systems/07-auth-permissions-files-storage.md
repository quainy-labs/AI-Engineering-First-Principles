# Chapter 7: Auth, Permissions, Files, and Storage

Chapter 6 showed how work moves through an AI system:

```text
API -> service -> queue -> worker -> database -> user-visible status
```

That movement is not enough by itself.

Before an AI system retrieves a document, reads a file, calls a tool, stores memory, or writes a result, it must answer a more basic question:

> Is this system allowed to use this information for this user, in this workspace, for this task?

This chapter gives you the access and storage foundation that later chapters will depend on. Retrieval, memory, agents, tools, logs, and governance all become dangerous if this layer is weak.

## Concept Overview

Auth, permissions, files, and storage decide what the AI system is allowed to see, remember, and act on.

They answer different questions:

```text
Authentication
Who is the user?

Authorization
What is this user allowed to access or do?

Storage
Where does information live?

Ownership
Who does this information belong to?

Retention
How long should this information exist?

Access metadata
What labels does the system need in order to enforce rules later?
```

In AI systems, access control is not a side feature. It is part of intelligence.

A model can only behave safely if the surrounding system gives it the right context. If the system retrieves unauthorized documents and then asks the model not to reveal them, the system has already failed.

The rule is simple:

> Unauthorized data should not enter the model context.

This applies to:

- documents;
- files;
- database records;
- memories;
- tool outputs;
- logs;
- traces;
- retrieved chunks;
- uploaded media;
- background jobs;
- generated artifacts.

## Why It Exists

AI applications often create new paths for data to move.

Before AI, a user might manually open a document only if they had permission. With an AI assistant, the system may search across many sources, summarize private records, call tools, store memory, and generate answers from hidden context.

That makes access control more important, not less.

Consider an internal company assistant.

An intern asks:

> What is our salary adjustment policy for senior engineers?

The assistant answers from an HR-only document.

The model did not "hack" the system. The architecture allowed unauthorized retrieval.

The bad design was:

```text
ingest all company documents
-> put chunks into one index
-> retrieve top matches
-> send chunks to model
-> tell model not to reveal restricted content
```

The safer design is:

```text
authenticate user
-> determine allowed resources
-> filter searchable documents by permission
-> retrieve only allowed chunks
-> send allowed context to model
-> log safely
```

This matters because prompts are not permission systems.

A prompt can express policy. It cannot enforce access. Enforcement must happen in backend code, storage design, retrieval filters, tool authorization, and audit logs.

AI systems need this foundation because they can fail in quiet ways:

- retrieving documents from the wrong tenant;
- leaking sensitive fields through logs;
- using deleted documents because chunks remain indexed;
- allowing a low-privilege user to trigger a high-privilege tool;
- storing personal data without retention rules;
- showing worker outputs to the wrong user;
- mixing test data and production data;
- letting one shared vector index ignore document ownership.

## Mental Model

Think of the AI system as a librarian inside a secured company building.

The librarian may be very smart, but intelligence does not grant permission.

Before answering, the system must check:

```text
Who is asking?
Which workspace are they in?
What resources can they access?
What action are they trying to perform?
Which data is allowed for this task?
What must be hidden, redacted, or forgotten?
```

The mental model:

```text
identity
-> permission
-> allowed resources
-> allowed retrieval
-> allowed model context
-> allowed action
-> safe storage and logs
```

The model is near the end of the chain, not the beginning.

This is the most important shift:

```text
Weak system:
retrieve everything -> ask model to behave

Strong system:
authorize first -> retrieve allowed data -> ask model to reason
```

The model should not be responsible for deciding whether the user may see a source document. That decision belongs to the application.

## Internal Mechanics

Access and storage design usually starts with resources.

A resource is anything the system stores, reads, retrieves, modifies, or exposes:

- user;
- organization;
- workspace;
- document;
- file;
- chunk;
- embedding;
- ticket;
- customer record;
- tool action;
- memory;
- job;
- log;
- trace;
- generated report.

Each resource needs metadata. Without metadata, permissions cannot be enforced reliably.

Useful metadata includes:

```text
resource_id
resource_type
owner_user_id
workspace_id
organization_id
created_by
created_at
updated_at
access_level
allowed_roles
source_system
retention_policy
sensitivity_level
deletion_status
```

### Authentication

Authentication identifies the user or service making the request.

Examples:

- signed-in user;
- service account;
- API key;
- integration token;
- background worker identity.

The system should not treat workers as permission-free. A worker processing a job should still know which user, workspace, or service authority created that job.

### Authorization

Authorization decides whether the actor can perform an action.

Authorization should happen before:

- uploading files;
- reading files;
- retrieving documents;
- calling tools;
- writing memory;
- viewing logs;
- exporting reports;
- deleting records;
- running jobs on behalf of another user.

Common permission questions:

```text
Can this user read this document?
Can this user upload to this workspace?
Can this user ask questions over this source?
Can this user call this tool?
Can this worker process this job?
Can this admin view these logs?
```

### File Storage

Files need more care than plain text fields.

For every uploaded file, define:

```text
storage location
owner
workspace
allowed file types
maximum size
virus or malware scanning
parsing status
retention policy
deletion behavior
sensitivity label
```

The file itself and the extracted text should both be controlled. A common mistake is deleting the original file but leaving extracted chunks or embeddings active.

### Retrieval Filtering

Retrieval must be permission-aware.

The correct flow:

```text
authenticate user
-> identify allowed resources
-> search only allowed resources
-> retrieve allowed chunks
-> send allowed chunks to model
```

The dangerous flow:

```text
search all resources
-> retrieve best chunks
-> ask model to hide restricted content
```

For retrieval systems, chunks should carry access metadata from their source:

```text
chunk_id
document_id
workspace_id
source_uri
owner
allowed_roles
sensitivity_level
document_status
```

When the source document is deleted, archived, or permission-changed, the derived chunks and embeddings must follow.

### Logs and Traces

AI systems create logs that may contain sensitive information:

- user prompt;
- retrieved context;
- model output;
- tool arguments;
- tool results;
- error messages;
- uploaded file names;
- extracted text;
- traces and spans.

Logging is useful for debugging, evaluation, and compliance. But logs can become a second data leak if they store secrets without access control.

Design decisions:

```text
What should be logged?
What should be redacted?
Who can view logs?
How long are logs retained?
Can user data be deleted from logs?
Are prompts and outputs treated as sensitive?
```

### Storage Boundaries

AI applications often use multiple storage systems:

- relational database for users, jobs, permissions, and durable state;
- object storage for files;
- search index for text search;
- vector index for embeddings;
- cache for temporary results;
- analytics store for metrics;
- logging system for traces.

Every storage system must preserve the access model. Permission cannot exist only in the main database if the vector index, cache, or logs ignore it.

## Real-World Example

Imagine a customer support AI system for a SaaS company.

The product goal:

> Support agents can ask questions about customer accounts and draft replies using account data, past tickets, and help-center articles.

Resources:

```text
support_agent
customer_account
ticket
attachment
help_center_article
internal_policy
generated_reply
tool_action
trace
```

Access rules:

- support agents can view tickets assigned to their team;
- managers can view all tickets in their region;
- billing records require special permission;
- customer attachments must be scanned before parsing;
- generated replies should be stored with the ticket;
- traces should redact payment details and secrets;
- help-center articles are public;
- internal policy documents are employee-only;
- deleted attachments must be removed from retrieval.

Bad design:

```text
all tickets + all policies + all attachments -> one vector index
```

Then every question searches the same index and hopes the model does the right thing.

Better design:

```text
user asks question
-> backend authenticates support agent
-> permission service finds allowed customer accounts and documents
-> retriever searches allowed sources only
-> model drafts answer using allowed context
-> backend checks whether requested tool action is permitted
-> generated reply is stored on the ticket
-> trace is logged with redaction
```

If the user asks for billing data without permission, the system does not retrieve billing records. The model never sees them.

This is the key product behavior:

> The answer is safe because the context was safe before generation began.

## Common Mistakes and Failure Modes

Mistake 1: Using prompts as access control

Instruction can guide model behavior, but it cannot replace backend enforcement. Do not send restricted context to the model and hope the prompt prevents leakage.

Mistake 2: One shared index without permission metadata

If all documents go into one index and chunks do not carry ownership or access labels, retrieval cannot reliably filter results.

Mistake 3: Filtering after retrieval

Filtering after retrieval can still leak through ranking, logs, traces, or accidental context construction. The safer default is to restrict the candidate set before retrieval.

Mistake 4: Deleting files but not derived data

When a source document is deleted, the system must handle extracted text, chunks, embeddings, summaries, caches, and generated artifacts that depend on it.

Mistake 5: Logging sensitive data by default

Prompts, retrieved context, tool outputs, and model responses can contain private data. Logs need redaction, retention, and viewer permissions.

Mistake 6: Workers bypass permissions

Background jobs often run with broad system privileges. They still need job ownership, workspace scope, and permission-aware behavior.

Mistake 7: Tool permissions are broader than user permissions

If the model can call tools that the user cannot use directly, the AI system becomes a privilege escalation path.

Mistake 8: No retention model

If the system never decides when data should expire, it will keep prompts, files, logs, embeddings, and generated outputs longer than needed.

## System Design Decisions

Design the access model before building retrieval, tools, memory, or agents.

Start with identity:

```text
Who can use the system?
Users, admins, service accounts, workers, integrations?
```

Define resources:

```text
What does the system store or access?
Documents, tickets, files, memories, jobs, traces, actions?
```

Define actions:

```text
read
write
upload
delete
retrieve
export
share
approve
execute tool
view logs
```

Define ownership:

```text
Does the resource belong to a user, team, workspace, organization, or external customer?
```

Define access model:

```text
role-based access
attribute-based access
tenant isolation
document-level permissions
source-system permissions
hybrid model
```

Define storage rules:

```text
Where is the raw file stored?
Where is extracted text stored?
Where are chunks stored?
Where are embeddings stored?
Where are logs stored?
How are deletes propagated?
```

Define retrieval rules:

```text
Which metadata must every chunk carry?
How is permission filtering applied before retrieval?
What sources are forbidden for this user?
What happens when permission changes?
```

Define log rules:

```text
What fields are redacted?
Who can view traces?
How long are prompts and outputs kept?
How are secrets detected?
```

The system should be able to explain why a piece of information was allowed into the model context.

## Build Artifact

Create `access-and-storage-model.md` for one AI feature.

Choose a feature such as:

- document knowledge assistant;
- support reply assistant;
- invoice extraction system;
- HR policy assistant;
- operations action assistant;
- multimodal document analyst.

Use this template:

```text
Access and Storage Model

Feature:
User goal:

Actors:
- actor:
  identity type:
  allowed actions:

Resources:
- name:
  owner:
  workspace or tenant:
  storage system:
  sensitivity:
  read permission:
  write permission:
  delete behavior:
  retention:

Files:
- file type:
  storage location:
  allowed uploaders:
  scanning rule:
  parsing rule:
  max size:
  deletion behavior:

Retrieval:
- source:
  required metadata:
  pre-filter rule:
  forbidden condition:
  permission-change behavior:

Tools:
- tool:
  allowed roles:
  required approval:
  side effect:
  audit record:

Logs and Traces:
- logged fields:
  redacted fields:
  viewer roles:
  retention:

Tests:
- unauthorized read:
- authorized read:
- role change:
- deleted source:
- restricted tool:
- sensitive log:
- retention expiry:
```

Example:

```text
Feature: HR policy assistant
User goal: Employees ask policy questions

Resource: HR compensation policy
Owner: HR department
Workspace: company
Sensitivity: restricted
Read permission: HR team and executives
Retrieval rule: only retrieve if user role includes hr_policy_reader
Deletion behavior: remove file, extracted text, chunks, embeddings, and cache
Logs: redact employee names and compensation amounts

Test:
Intern asks compensation question -> restricted document is not retrieved
HR manager asks compensation question -> restricted document can be retrieved
```

The artifact is complete when another engineer can tell exactly what data the AI system may access, where that data lives, and how access is enforced.

## Active Recall Questions

1. Why is a prompt not a permission system?
2. What is the difference between authentication and authorization?
3. Why should retrieval filter by permission before sending context to the model?
4. What metadata should document chunks carry for access control?
5. How can logs and traces leak sensitive information?
6. Why must workers know the ownership and scope of the jobs they process?
7. What derived data should be removed when a source document is deleted?
8. How can tool use become a privilege escalation path?

## Summary

AI systems do not become safe because the model is told to be careful. They become safer when the software decides what data is allowed before the model sees it.

Authentication identifies the actor. Authorization decides what they can do. Storage defines where information lives. Ownership and metadata let the system enforce permissions across documents, files, chunks, embeddings, jobs, tools, logs, and traces.

The most important rule:

> Do not retrieve unauthorized data and ask the model to hide it. Keep unauthorized data out of the context.

## Preview of Next Chapter

Chapter 7 defined what the system is allowed to access and where information lives.

Chapter 8 finishes the software foundation by handling time.

Next, you will learn async jobs, background processing, webhooks, retries, timeouts, duplicate events, and state transitions. This matters because real AI work often continues after the user request: files finish indexing, external systems respond later, approvals expire, and jobs fail or retry.
