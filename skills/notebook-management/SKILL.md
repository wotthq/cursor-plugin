---

name: notebook-management
description: Manage notebooks in wott by finding, reviewing, creating, and updating notebooks. Use when the user wants to work with wott notebooks or needs notebook information and context for a task.
metadata:
  short-description: Manage wott notebooks and build context
  
---

# Notebook Management

Use the wott MCP server to help the authenticated user find, understand, create, and update notebooks.

Notebooks contain the user's knowledge and can provide context for research, projects, and other tasks.

## Quick start

1. Identify what the user wants to do with a notebook.
2. For notebook-specific workflows, consult the relevant guide in `reference/`.
3. Find the relevant notebook using the available notebook listing or search capabilities.
4. Retrieve notebook details or content with `get_notebook` when needed.
5. Create or update notebooks using the appropriate MCP tool.
6. Use relevant notebook information to build context for the user's task.
7. Confirm the MCP operation succeeded before reporting a change.

## Tool-call guardrails

* Always use the authenticated wott user.
* Never ask the user for a login UID, OAuth access token, login credentials, Firestore credentials, or internal server secrets.
* Never fabricate notebook IDs.
* Never fabricate notebook content or project relationships.
* Do not access another user's notebooks.
* Do not perform a write operation when the target notebook or requested change is ambiguous.
* For updates, modify only the fields requested by the user.
* Do not claim a notebook was created or updated unless the MCP tool confirms success.
* If a write operation fails, report the failure accurately.
* Do not blindly retry a failed write operation when it could create a duplicate or unintended change.
* Use the tool schemas exposed by the wott MCP server as the source of truth for available arguments and fields.

## Workflow

### 1. Find or identify a notebook

Use the available notebook search or listing capability when the user refers to a notebook by:

* Name
* Topic
* Keyword
* Description
* Project

For detailed notebook identification and search workflows, read:

`reference/notebook-workflows.md`

Examples of requests that should trigger this workflow:

* "Find my robotics notebook."
* "Show me my research notebooks."
* "Find my notebooks about humanoid robotics."
* "Which notebooks are in my Robotics project?"

If exactly one notebook clearly matches, continue.

If multiple notebooks could match, ask the user to clarify.

If no relevant notebook is found, tell the user.

### 2. Read a notebook

Use `get_notebook` when the user wants information, details, or content from a specific notebook.

If the user provides a notebook ID, use it directly.

If the user provides only a notebook name or description, resolve the notebook first.

For detailed read and resolution workflows, read:

`reference/notebook-workflows.md`

For the available tool arguments and expected notebook output, read:

`reference/notebook-tool-reference.md`

### 3. Create a notebook

Use `create_notebook` when the user clearly asks for a new notebook.

Before creating:

1. Determine the notebook name.
2. Determine any project relationship specified by the user.
3. Determine other required fields from the MCP tool schema.
4. Check for an obvious duplicate when appropriate.
5. Resolve the project when necessary.
6. Create the notebook.
7. Verify the returned result.

For the detailed creation workflow, read:

`reference/notebook-workflows.md`

For the tool schema and output details, read:

`reference/notebook-tool-reference.md`

See:

`examples/create-notebook.md`

for a concrete creation workflow.

Do not create a notebook merely because the user discusses a topic.

### 4. Update a notebook

Use `update_notebook` when the user asks to modify an existing notebook.

Before updating:

1. Identify the target notebook.
2. Resolve it if necessary.
3. Determine exactly what the user wants changed.
4. Update only the requested fields.
5. Preserve unrelated notebook information.
6. Verify the result.
7. Report the change.

For detailed update behavior, read:

`reference/notebook-workflows.md`

For the tool schema, read:

`reference/notebook-tool-reference.md`

See:

`examples/update-notebook.md`

for a concrete example.

If multiple notebooks match, ask the user to clarify before making a change.

### 5. Build context from notebooks

Use wott notebooks as a knowledge source when the user's task depends on information stored in their notebooks.

For example:

> What do my notes say about humanoid robotics?

Follow this general workflow:

```text
identify topic
      ↓
search relevant notebooks
      ↓
select relevant notebooks
      ↓
get_notebook
      ↓
extract relevant information
      ↓
build context
      ↓
answer user
```

For detailed context-building workflows, read:

`reference/notebook-workflows.md`

For a concrete example, read:

`examples/find-notebook.md`

Do not summarize unrelated notebook content.

Do not invent information that is not present in wott.

### 6. Build context across multiple notebooks

When one notebook is insufficient, search for additional relevant notebooks.

```text
user's task
     ↓
search topic
     ↓
relevant notebooks
     ↓
retrieve relevant content
     ↓
combine information
     ↓
build task-specific context
```

Use only information relevant to the user's request.

When multiple notebooks contain overlapping information, avoid unnecessary repetition.

When notebooks contain conflicting information, preserve the distinction rather than silently choosing one.

For detailed workflows, read:

`reference/notebook-workflows.md`

### 7. Project-aware notebook workflows

When the user specifies a project:

> Find the notebooks about locomotion in my Robotics project.

Preserve the project context throughout the workflow.

```text
identify project
      ↓
find notebooks in project
      ↓
search/filter for topic
      ↓
retrieve relevant notebooks
      ↓
build context
```

Do not assume a notebook belongs to a project unless wott or the user establishes the relationship.

For project-aware workflows, read:

`reference/notebook-workflows.md`

When resolving the project itself is necessary, use the `project-management` skill.

## Notebook identification rules

When the user provides a notebook name:

1. Search or list notebooks.
2. Compare the returned results with the user's request.
3. If exactly one notebook is an obvious match, use it.
4. If multiple notebooks match, ask the user to clarify.
5. If no notebook matches, tell the user.

When the user provides a notebook ID, use it directly when the tool accepts it.

Never fabricate notebook IDs.

## Notebook creation rules

Create a notebook only when the user clearly requests creation.

Examples:

* "Create a notebook called Robotics Research."
* "Create a notebook for this research."
* "Create a notebook in my Robotics project."

Before creating, check for an obvious duplicate when appropriate.

If the user specifies a project, preserve that relationship.

If the project is ambiguous and the relationship is required, ask the user to clarify.

For detailed creation behavior, read:

`examples/create-notebook.md`

## Notebook update rules

Only modify information the user requested.

For example:

> Rename my robotics notebook to Humanoid Manipulation.

Only change the notebook name.

Do not overwrite its content, description, project relationship, or other fields unless the user explicitly requests those changes.

For detailed update behavior, read:

`examples/update-notebook.md`

## Context-building rules

When the user asks wott to help answer a question using their existing knowledge:

1. Identify the topic.
2. Search relevant notebooks.
3. Retrieve relevant notebook content when necessary.
4. Extract information relevant to the current task.
5. Build a concise context from that information.
6. Use the context to answer the user's request.

Prefer relevant information over exhaustive retrieval.

Do not present information as coming from wott when it was inferred independently.

For detailed context workflows, read:

`reference/notebook-workflows.md`

For examples, read:

`examples/find-notebook.md`

## Authentication

All notebook operations must use the authenticated wott user.

The model must never request or expose:

* Login UID
* OAuth access token
* Login credentials
* Firestore credentials
* Internal server secrets

Never attempt to access another user's notebooks.

## Errors

If a notebook cannot be found, tell the user clearly.

If multiple notebooks match, ask the user to clarify.

If a project relationship is ambiguous and affects the requested operation, ask for clarification.

If wott returns an authorization error, do not attempt to bypass it.

If an MCP operation fails, report the failure accurately.

Never claim success unless the MCP tool confirms it.

## References and examples

The `reference/` directory contains detailed notebook-management guidance that should be consulted when the task requires more specific workflow or tool information.

* `reference/notebook-workflows.md` — detailed workflows for finding, reading, creating, updating, resolving ambiguous notebook references, handling project relationships, and building context from one or more notebooks.
* `reference/notebook-tool-reference.md` — wott notebook MCP tools, tool annotations, proposed input schemas, output schemas, structured output, and notebook data representation.

The `examples/` directory contains concrete examples of common notebook-management workflows.

* `examples/create-notebook.md` — creating a notebook, checking duplicates, and preserving project context.
* `examples/update-notebook.md` — identifying and safely updating an existing notebook.
* `examples/find-notebook.md` — finding notebooks, resolving ambiguous results, retrieving notebook content, and building context.

Read the relevant reference or example file when it provides guidance needed for the current task. Do not load unrelated reference files unnecessarily.
