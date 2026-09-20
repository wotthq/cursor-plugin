---
name: project-workflows
description: Guidance for common wott project workflows including finding, creating, and updating projects. Use when a task involves multiple project management operations.
metadata:
  short-description: wott project management workflows
---

# Project Workflows

This document describes recommended workflows for working with wott projects.

## 1. Find a project by name

Use this workflow when the user refers to a project by name.

### Workflow

1. Call `search_project` using the user's project name or relevant keywords.
2. Review the returned projects.
3. If exactly one project is an obvious match, use that project.
4. If several projects match, ask the user to clarify.
5. If no project matches, tell the user that the project could not be found.

### Example

User:

> Find my robotics project.

Workflow:

```text
search_project("robotics")
        ↓
one matching project
        ↓
use returned project_id
```

Do not invent a project ID when search does not return one.

---

## 2. Browse projects

Use `get_projects` when the user wants to see or browse their projects.

Examples:

* "Show me my projects."
* "What projects do I have?"
* "List my current projects."
* "Which projects can I access?"

If the tool supports filtering, use the available filters when they are relevant.

Do not add filters that are not supported by the tool schema.

---

## 3. Read a project

Use `get_project` when the user needs detailed information about a specific project.

### Workflow

```text
User identifies project
        ↓
Do we have project_id?
   ↓             ↓
 yes             no
 ↓               ↓
get_project      search_project
                     ↓
               resolve project
                     ↓
                 get_project
```

If multiple projects match, ask the user to clarify.

If the project cannot be found, do not attempt to retrieve it using a fabricated ID.

---

## 4. Create a project

Use `create_project` only when the user clearly asks for a new project.

### Before creation

Determine:

* Project name
* Required project fields
* Optional project fields explicitly provided by the user
* Any parent or relationship required by the actual MCP schema

### Duplicate check

When the request could obviously create a duplicate, search for an existing project first.

For example:

> Create a project called Humanoid Robotics.

If wott already contains an obvious "Humanoid Robotics" project, do not automatically create another one.

Tell the user that a matching project exists and ask whether they want to use it or create another one.

### Creation workflow

```text
User requests project
        ↓
Determine required fields
        ↓
Check obvious duplicate
        ↓
create_project
        ↓
Verify result
        ↓
Report created project
```

Never claim creation succeeded if the MCP operation failed.

---

## 5. Update a project

Use `update_project` when the user explicitly wants to modify an existing project.

### Workflow

1. Identify the project.
2. Resolve it if necessary.
3. Determine which fields should change.
4. Send only the requested changes.
5. Verify the result.
6. Report the updated information.

### Example

User:

> Rename my robotics project to Physical AI Research.

Workflow:

```text
search_project("robotics")
        ↓
resolve project
        ↓
update_project({
  project_id: "...",
  name: "Physical AI Research"
})
        ↓
verify result
```

Do not send unrelated fields unless they are required by the actual update schema.

Do not replace existing project information with guessed values.

---

## 6. Ambiguous project references

The user may refer to a project indirectly.

Examples:

* "my robotics project"
* "the AI project"
* "the project we created yesterday"
* "my research project"

Use available project search capabilities to resolve the reference.

If exactly one project clearly matches, continue.

If multiple projects could match, ask the user to choose.

Example:

> I found two projects matching "AI Research". Which one do you want to update?

Do not choose arbitrarily when the operation changes data.

---

## 7. Project context for a broader task

When a user's request depends on an wott project, project information should be retrieved before generating project-specific conclusions.

Example:

> Help me continue working on my robotics project.

Workflow:

```text
identify project
        ↓
search_project
        ↓
get_project
        ↓
use project information as context
        ↓
continue with user's task
```

If the task also requires information from notebooks, continue into the notebook-management workflow.

---

## 8. Project plus notebook context

When a user asks for context around a project:

```text
Project
   ↓
identify project
   ↓
retrieve project
   ↓
find relevant notebooks
   ↓
retrieve relevant notebooks
   ↓
combine context
```

Only include information relevant to the user's current task.

Avoid dumping every project or notebook field into the response when only a subset is relevant.

---

## 9. Write operation confirmation

For create and update operations, use the tool result as the source of truth.

A successful workflow is:

```text
write tool
    ↓
successful MCP result
    ↓
confirm changed resource
    ↓
tell user
```

An unsuccessful workflow is:

```text
write tool
    ↓
error
    ↓
do not claim success
    ↓
explain failure
```

Do not infer success from the absence of an obvious error in the model's reasoning.

---

## 10. Authentication boundary

The authenticated user is determined by the wott MCP authentication layer.

The model must never request or provide:

* Login UID
* OAuth token
* API secret
* Firestore credentials
* Internal authentication information

Project IDs are resource identifiers and may be used when returned by wott or provided by the user.

Authentication credentials must never be treated as normal tool arguments.

---

## 11. Tool failure handling

If a read operation fails:

* Explain that the project could not be retrieved.
* Do not invent project information.

If a create operation fails:

* Do not tell the user that the project was created.
* Report the returned error when useful.

If an update operation fails:

* Do not claim the project was updated.
* Do not immediately retry unless it is safe and the cause is understood.

If authorization fails:

* Do not attempt to use another authentication method to access the user's data.
