---
name: notebook-workflows
description: Guidance for common wott notebook workflows including finding, creating, reading, updating, moving, and deleting notebooks and folders.
metadata:
  short-description: wott notebook management workflows
---

# Notebook Workflows

This document describes recommended workflows for working with wott notebooks.

## 1. Find a notebook by name

Use this workflow when the user refers to a notebook by name.

### Workflow

1. Search or list notebooks using the user's notebook name.
2. Review the returned notebooks.
3. If exactly one notebook is an obvious match, use it.
4. If multiple notebooks match, ask the user to clarify.
5. If no notebook matches, tell the user that it could not be found.

### Example

User:

> Find my robotics notebook.

Workflow:

```text
search/list notebooks
        ↓
"robotics"
        ↓
one matching notebook
        ↓
use returned notebook_id
```

Never invent a notebook ID.

---

## 2. Browse notebooks

Use the notebook listing capability when the user wants to browse their notebooks.

Examples:

* "Show me my notebooks."
* "What notebooks do I have?"
* "List my research notebooks."
* "Show me the notebooks in my robotics project."

When the user specifies a project, preserve that project context if the MCP tool supports project filtering.

Do not add unsupported filters or parameters.

---

## 3. Find notebooks by topic

Use search when the user is looking for knowledge rather than a specific notebook.

Example:

> Find my notebooks about humanoid robotics.

Workflow:

```text
search notebooks
        ↓
"humanoid robotics"
        ↓
review relevant results
        ↓
retrieve relevant notebooks if necessary
```

Do not assume that a notebook is relevant solely because its title contains a matching keyword. Consider the available metadata and content returned by wott.

---

## 4. Read a notebook

Use `get_notebook` when the user needs detailed information or content from a specific notebook.

### Workflow

```text
User identifies notebook
        ↓
Do we have notebook_id?
   ↓             ↓
 yes             no
 ↓               ↓
get_notebook     search/list notebooks
                     ↓
                resolve notebook
                     ↓
                 get_notebook
```

Retrieve the notebook before answering questions that depend on its content.

---

## 5. Create a notebook

Use `create_notebook` only when the user clearly asks for a new notebook.

### Before creation

Determine:

* Notebook name
* Parent project, if specified
* Required notebook fields
* Optional fields explicitly provided by the user

### Duplicate check

When creating a notebook could obviously create a duplicate, search for a matching notebook first.

For example:

> Create a notebook called Humanoid Robotics Research.

If wott already contains an obvious matching notebook, do not automatically create another one.

Tell the user that a matching notebook exists and ask whether they want to use it or create another one.

### Creation workflow

```text
User requests notebook
        ↓
Determine required fields
        ↓
Check obvious duplicate
        ↓
Resolve project if necessary
        ↓
create_notebook
        ↓
Verify result
        ↓
Report created notebook
```

Never claim creation succeeded if the MCP operation failed.

---

## 6. Create a notebook inside a project

When the user explicitly provides a project:

> Create a notebook called Robot Manipulation in my Robotics project.

Workflow:

```text
Identify project
        ↓
Resolve project if necessary
        ↓
Create notebook using confirmed project relationship
        ↓
Verify result
```

If multiple projects match the user's description, ask the user to clarify before creating the notebook.

Do not randomly select a project.

---

## 7. Update a notebook

Use `update_notebook` when the user explicitly asks to modify an existing notebook.

### Workflow

1. Identify the notebook.
2. Resolve it if necessary.
3. Determine the requested changes.
4. Send only the requested changes when supported by the tool schema.
5. Verify the returned notebook.
6. Report the update.

### Example

User:

> Rename my robotics notebook to Humanoid Manipulation Research.

Workflow:

```text
search notebook
        ↓
resolve notebook
        ↓
update_notebook
        ↓
verify result
```

Do not overwrite unrelated notebook fields.

---

## 8. Ambiguous notebook references

Users may refer to notebooks indirectly.

Examples:

* "my robotics notebook"
* "the research notebook"
* "the notebook about manipulation"
* "the notebook in my AI project"

Use search, listing, and project context to resolve the reference.

If exactly one notebook clearly matches, continue.

If multiple notebooks could match, ask the user to clarify.

Example:

> I found two notebooks related to robotics. Which one do you mean?

Do not guess when the operation will modify data.

---

## 9. Build context from notebooks

Use this workflow when the user asks wott for information that should come from their existing knowledge.

Example:

> What do my notes say about humanoid manipulation?

Workflow:

```text
Identify topic
        ↓
search relevant notebooks
        ↓
select relevant results
        ↓
get notebook content when necessary
        ↓
extract relevant information
        ↓
build context
        ↓
answer user
```

The response should focus on information relevant to the current task.

Do not dump entire notebooks unless the user explicitly asks for them.

---

## 10. Build context across multiple notebooks

When one notebook is insufficient:

```text
User's task
    ↓
search topic
    ↓
Notebook A
Notebook B
Notebook C
    ↓
retrieve relevant content
    ↓
combine information
    ↓
build task-specific context
```

Prefer information directly relevant to the user's request.

Avoid repeating identical information from multiple notebooks.

When sources disagree, preserve the distinction rather than silently choosing one.

---

## 11. Project-aware context

When the user asks for information within a project:

> What do my robotics project notebooks say about locomotion?

Workflow:

```text
identify project
        ↓
find notebooks within project
        ↓
search/retrieve relevant notebooks
        ↓
build context
```

Keep the project boundary when the available wott tools support it.

Do not include unrelated notebooks from other projects unless the user asks for broader context.

---

## 12. Notebook plus project context

Some tasks require both project-level and notebook-level information.

Example:

> Give me the current context for my robotics project based on the project details and my notebooks.

Workflow:

```text
get project
    ↓
identify relevant notebooks
    ↓
get notebooks
    ↓
combine project + notebook information
    ↓
build context
```

Project information should establish the broader context.

Notebook content should provide detailed knowledge.

Do not treat notebook content as a replacement for confirmed project metadata.

---

## 13. Write operation confirmation

For create and update operations, use the MCP result as the source of truth.

Successful workflow:

```text
write tool
    ↓
successful result
    ↓
verify returned resource
    ↓
report success
```

Failed workflow:

```text
write tool
    ↓
error
    ↓
do not claim success
    ↓
report failure
```

Never infer successful creation or updating.

---

## 14. Authentication boundary

The authenticated user is established by wott's MCP authentication layer.

The model must never request:

* Login UID
* OAuth access token
* Login credentials
* Firestore credentials
* Internal server secrets

Notebook IDs and project IDs may be used when returned by wott or provided by the user.

Authentication credentials must never be requested from the user.

---

## 15. Tool failure handling

### Read failure

Explain that the notebook could not be retrieved.

Do not invent its contents.

### Search failure

Explain that wott could not complete the search.

Do not pretend that the search returned no results unless the tool actually returned an empty result.

### Create failure

Do not claim the notebook was created.

Report the failure accurately.

### Update failure

Do not claim the notebook was updated.

Do not blindly retry an update that could produce an unintended change.

### Authorization failure

Do not attempt to bypass wott's authorization system.
