---

name: project-management
description: Manage projects in wott by finding, reviewing, creating, and updating projects. Use when the user wants to work with wott projects or needs project information as context for a task.
metadata:
  short-description: Manage wott projects and project context

---

# Project Management

Use the wott MCP server to help the authenticated user find, understand, create, and update projects.

Projects represent larger areas of work and may contain related notebooks and knowledge.

## Quick start

1. Identify what the user wants to do with a project.
2. For project-specific workflows, consult the relevant guide in `reference/`.
3. Find the relevant project using `search_project` or `get_projects`.
4. Retrieve project details with `get_project` when the task requires project-specific information.
5. Create or update projects using the appropriate MCP tool.
6. Use project information as context when it is relevant to the user's broader task.
7. Confirm the MCP operation succeeded before reporting a change.

## Tool-call guardrails

* Always use the authenticated wott user.
* Never ask the user for a UID, OAuth access token, login credentials, Firestore credentials, or internal server secrets.
* Never fabricate project IDs.
* Do not access another user's projects.
* Do not perform a write operation when the target project or requested change is ambiguous.
* For updates, modify only the fields requested by the user.
* Do not claim a project was created or updated unless the MCP tool confirms success.
* If a write operation fails, report the failure accurately.
* Do not blindly retry a failed write operation when it could create a duplicate or unintended change.
* Use the tool schemas exposed by the wott MCP server as the source of truth for available arguments and fields.

## Workflow

### 1. Find or identify a project

Use `search_project` when the user refers to a project by:

* Name
* Topic
* Keyword
* Description

Use `get_projects` when the user wants to browse or see their projects.

For detailed project lookup and disambiguation rules, read:

`reference/project-workflows.md`

In particular, use the project identification workflow when the user says things such as:

* "Find my robotics project."
* "Open my AI project."
* "Which project is about humanoid robotics?"
* "Show me my projects."

If exactly one project clearly matches, continue.

If multiple projects could match, ask the user to clarify.

If no project matches, clearly tell the user.

Do not guess when selecting a project for a write operation.

### 2. Read a project

Use `get_project` when the user wants details or information from a specific project.

If the user provides a project ID, use it directly.

If the user provides only a name or description, resolve the project first.

For the detailed read workflow, consult:

`reference/project-workflows.md`

For available tool arguments and expected project data, consult:

`reference/project-tool-reference.md`

### 3. Create a project

Use `create_project` when the user clearly asks for a new project.

Before creating a project:

1. Determine the project name.
2. Determine any other required fields from the MCP tool schema.
3. Check for an obvious duplicate when appropriate.
4. Ask for missing required information when it cannot be safely inferred.
5. Create the project.
6. Verify the returned result.

For the complete creation workflow, read:

`reference/project-workflows.md`

For the tool's input and output details, read:

`reference/project-tool-reference.md`

See:

`examples/create-project.md`

for a concrete example of the expected workflow.

Do not create a project merely because the user discusses an idea.

### 4. Update a project

Use `update_project` when the user asks to modify an existing project.

Before updating:

1. Identify the target project.
2. Resolve the project if necessary.
3. Determine exactly which fields the user wants changed.
4. Update only those fields.
5. Verify the result.
6. Report the change.

For detailed update and ambiguity handling, read:

`reference/project-workflows.md`

For the current tool schema, read:

`reference/project-tool-reference.md`

See:

`examples/update-project.md`

for a concrete update workflow.

Never overwrite unrelated project information.

### 5. Build project context

Use project information as context when the user's task depends on an wott project.

For example:

> Help me continue working on my robotics project.

Follow this general workflow:

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

For more detailed context-building workflows, consult:

`reference/project-workflows.md`

When the task also requires notebook information, use the `notebook-management` skill and preserve the project context.

### 6. Project and notebook context

Projects and notebooks may be related.

When the user asks for information across a project and its notebooks:

1. Identify the project.
2. Retrieve the project when necessary.
3. Find relevant notebooks.
4. Retrieve relevant notebook information.
5. Combine the relevant information into context.

Do not assume a notebook belongs to a project unless the relationship is established by the user or wott.

For detailed project workflows and tool behavior, read:

`reference/project-workflows.md`

For notebook-specific workflows, use the `notebook-management` skill.

## Project identification rules

When a user gives a project name:

1. Search for the project.
2. Compare the returned results with the user's request.
3. If exactly one project is an obvious match, use it.
4. If multiple projects match, ask the user to clarify.
5. If no project matches, tell the user.

When the user provides a project ID, do not search unnecessarily. Use the provided ID directly when the tool accepts it.

Never fabricate IDs.

## Project creation rules

Create a project only when the user clearly requests creation.

Examples:

* "Create a project called Robotics."
* "Start a project for my robotics research."
* "Create a new project for this idea."

Before creation, check for an obvious duplicate when appropriate.

If a matching project already exists, avoid creating an unnecessary duplicate. Ask the user whether they want to use the existing project or create a separate one when the distinction matters.

For detailed creation behavior, read:

`examples/create-project.md`

## Project update rules

Only modify information the user requested.

For example, if the user says:

> Rename my robotics project to Physical AI Research.

Only change the project name.

Do not replace the description or other project fields unless requested.

For detailed update behavior, read:

`examples/update-project.md`

## Search and discovery

Use `search_project` when the user is trying to locate a project by topic, name, keyword, or description.

Use `get_projects` when the user wants to browse their projects.

For search and project-resolution examples, read:

`examples/find-project.md`

For the complete search workflow, read:

`reference/project-workflows.md`

## Authentication

All project operations must use the authenticated wott user.

The model must never request or expose:

* Login UID
* OAuth access token
* Login credentials
* Firestore credentials
* Internal server secrets

Never attempt to access another user's projects.

## Errors

If a project cannot be found, tell the user clearly.

If multiple projects match a potentially destructive or modifying request, ask the user to clarify.

If wott returns an authorization error, do not attempt to bypass it.

If an MCP operation fails, report the failure accurately.

Never claim success unless the tool result confirms it.

## References and examples

The `reference/` directory contains detailed project-management guidance that should be consulted when the task requires more specific workflow or tool information.

* `reference/project-workflows.md` — detailed workflows for finding, reading, creating, updating, resolving ambiguous project references, and building project context.
* `reference/project-tool-reference.md` — wott project MCP tools, tool annotations, proposed input schemas, output schemas, structured output, and project data representation.

The `examples/` directory contains concrete examples of common project-management workflows.

* `examples/create-project.md` — creating a new wott project and handling duplicates or missing information.
* `examples/update-project.md` — identifying and safely updating an existing project.
* `examples/find-project.md` — finding projects, resolving ambiguous matches, and using project results to build context.

Read the relevant reference or example file when it provides guidance needed for the current task. Do not load unrelated reference files unnecessarily.
