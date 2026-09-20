---

name: wott
description: Use wott to search, read, create, update, and manage projects and notebooks in the authenticated user's wott workspace. Use when the user wants to work with wott projects or notebooks, search project knowledge, organize notebook content, or perform project and notebook management tasks.
metadata:
  short-description: Work with wott projects and notebooks
  
---

# wott

wott is a project and notebook workspace that lets users organize, store, search, and manage their project knowledge and notebook content.

Use wott when the user wants to:

* Find or review their projects
* Create or update projects
* Find, read, or organize notebooks
* Create, update, move, or delete notebooks
* Search for information inside a project's notebooks
* Use information stored in wott as context for a task

Do not use wott for general knowledge questions, public web research, or tasks that do not require the user's wott data.

## Workspace Structure

wott organizes content using projects and notebook items.

The hierarchy is:

```text
wott Workspace
├── Project
│   ├── Folder
│   │   ├── Notebook
│   │   └── Notebook
│   ├── Notebook
│   └── Notebook
│
└── Project
    ├── Folder
    └── Notebook
```

A **project** is the top-level workspace for a collection of related information.

A **notebook item** can be either:

* `file`: a Markdown notebook containing content
* `folder`: a container for notebooks and other folders

Notebooks and folders belong to a project and can be organized using parent-child relationships.

## Available Capabilities

wott provides three main capability areas:

### Project Management

Use the `project-management` skill for project operations.

Available tools:

* `get_projects`
* `create_project`
* `update_project`

Use these tools when the user wants to find, create, or update a project.

### Notebook Management

Use the `notebook-management` skill for notebook and folder operations.

Available tools:

* `get_notebook`
* `create_notebook`
* `update_notebook`
* `delete_notebook`

Use these tools when the user wants to find, read, create, update, move, or delete notebook items.

### Project Search

Use the `search-project` skill for searching notebook knowledge.

Available tool:

* `search_project`

Use this tool when the user wants to find information inside the notebooks of a specific wott project.

## Tool Selection

Choose the narrowest tool that directly satisfies the user's request.

### Finding Projects

For requests such as:

* "Show me my projects"
* "What projects do I have?"
* "Find my robotics project"

Use:

```text
get_projects
```

Do not use notebook tools when the user is only asking about projects.

### Creating Projects

For requests such as:

* "Create a project called Robotics Research"
* "Make a new project for my AI research"

Use:

```text
create_project
```

A project name is required. A description may also be provided.

### Updating Projects

For requests such as:

* "Rename my robotics project"
* "Update the description of my AI project"

Use:

```text
get_projects
→ identify the requested project
→ update_project
```

Only update fields explicitly requested by the user.

Supported project updates include:

* project name
* description
* location
* region
* country code
* region code

Do not modify other project properties unless the tool explicitly supports them.

## Notebook Navigation

Notebook navigation is based on project, item ID, and parent-child relationships.

A notebook item can be located using:

* project ID
* notebook item ID
* parent folder ID
* item name or search criteria

When the user provides an exact notebook or item ID, use it directly.

When the user provides only a notebook name, first identify the relevant project and notebook before performing a mutation.

### Reading a Notebook

Use:

```text
get_notebook
```

for requests such as:

* "Open this notebook"
* "Show me my research notes"
* "Read this notebook"
* "What is in this notebook?"
* "List the notebooks in this folder"

`get_notebook` supports reading notebook items and navigating notebook contents.

Use the appropriate mode supported by the tool:

* `item`: retrieve a specific notebook or folder
* `search`: search notebook items by name or related criteria
* `children`: retrieve items inside a folder
* `list`: list notebook items in a project

### Searching Notebook Knowledge

Use:

```text
search_project
```

when the user wants to search the **content of notebooks within a specific project**.

Example:

> "Search my Robotics Research project for notes about humanoid robots."

The workflow is:

```text
Identify project
        ↓
search_project
        ↓
Review matching notebooks
        ↓
get_notebook if detailed content is needed
```

`search_project` searches notebook knowledge inside one specified project. It does not perform public web search and does not search unrelated wott projects.

## Creating Notebooks

Use:

```text
create_notebook
```

for requests such as:

* "Create a notebook"
* "Create a folder"
* "Add a research note to this project"

A notebook can be created directly in a project or inside an existing folder using the appropriate parent item.

Before creating a notebook, determine the target project.

If the user specifies a folder, ensure the folder belongs to the target project.

## Updating Notebooks

Use:

```text
update_notebook
```

for requests such as:

* "Rename this notebook"
* "Update the content"
* "Add this information to my notes"
* "Move this notebook into the research folder"
* "Replace the notebook content"

Supported updates may include:

* notebook name
* Markdown content
* parent folder
* frontmatter
* metadata

Only modify the fields requested by the user.

When updating content, preserve existing content unless the user explicitly asks to replace it.

## Deleting Notebooks

Use:

```text
delete_notebook
```

only when the user explicitly requests deletion.

Deletion is permanent and may remove:

* the selected notebook
* descendant folders
* descendant notebooks
* associated stored content
* associated search/index data
* associated graph data

Because deletion is destructive, do not infer a deletion request from phrases such as "clean this up" or "remove unnecessary notes" without sufficient clarification.

When the target is ambiguous, ask the user which notebook or folder they want deleted.

## Project and Notebook Identification

Users may refer to resources by:

* exact ID
* exact name
* partial name
* description
* contextual reference such as "my robotics project"

When an exact ID is provided, prefer the ID.

When only a name is provided:

1. Find the relevant project if necessary.
2. Identify the matching notebook or project.
3. If exactly one resource matches, continue.
4. If multiple resources match and the operation is destructive or modifying, ask the user to clarify.
5. Do not modify a resource based on an uncertain match.

For example, if the user says:

> "Update my Research notebook"

and multiple notebooks named "Research" exist, ask which one they mean before updating it.

## Multi-Step Workflows

Some user requests require multiple tools.

Follow the smallest necessary sequence.

### Example: Find and Read

```text
get_projects
    ↓
identify project
    ↓
get_notebook
    ↓
return relevant content
```

### Example: Search and Read

```text
get_projects
    ↓
identify project
    ↓
search_project
    ↓
get_notebook
```

Use `get_notebook` after `search_project` when the search results identify a notebook whose full content is needed.

### Example: Create Project and Notebook

```text
create_project
    ↓
use returned project_id
    ↓
create_notebook
```

Do not ask the user for the newly created project ID when it is already returned by `create_project`.

### Example: Update Project

```text
get_projects
    ↓
identify project
    ↓
update_project
```

### Example: Update Notebook

```text
identify project/notebook
    ↓
update_notebook
```

### Example: Delete Notebook

```text
identify notebook
    ↓
confirm target if ambiguous
    ↓
delete_notebook
```

## Authentication and User Identity

All wott MCP operations operate on the authenticated wott user.

Never ask the user to provide:

* Login UID
* OAuth user ID
* access token
* internal authentication identifiers

The MCP server receives the authenticated user's identity from the OAuth context.

The authenticated identity is used to authorize access to wott resources.

## Authorization

Only access or modify projects and notebooks that the authenticated user is authorized to access.

Project and notebook IDs must never be treated as authorization credentials.

Always rely on wott's authorization checks.

Do not attempt to access another user's project or notebook by guessing or directly supplying its ID.

If wott returns an authorization error, explain that the user does not have access to the requested resource.

Do not bypass or retry authorization failures using another identity.

## Read Operations

Read-only operations include:

```text
get_projects
get_notebook
search_project
```

These operations should not modify wott data.

Use them freely when they are necessary to answer a user's request involving their wott workspace.

## Mutating Operations

The following operations modify wott data:

```text
create_project
update_project
create_notebook
update_notebook
delete_notebook
```

Only perform mutations when the user's request clearly indicates the intended action.

Do not invent values for required fields.

If required information is missing and cannot be safely inferred, ask the user for it.

## Destructive Operations

`delete_notebook` is destructive.

Before calling it:

1. Identify the exact notebook or folder.
2. Ensure the user's request clearly asks for deletion.
3. If multiple resources match, ask for clarification.
4. Do not delete unrelated resources.

Deleting a folder can also delete its descendants.

Do not claim that deleted content can be recovered unless wott explicitly provides a recovery mechanism.

## Search Behavior

`search_project` searches notebook knowledge within one specified project.

It currently searches wott notebook content and indexed notebook information.

It does not:

* search the public internet
* search other users' projects
* search unrelated wott projects
* perform general web research

For public web research, use an appropriate web search capability instead of wott.

For detailed content after finding a relevant notebook, use `get_notebook`.

## Content and Storage

wott notebooks use Markdown content.

Notebook content may include:

* Markdown text
* headings
* lists
* links
* code blocks
* frontmatter
* notebook metadata

When creating or updating Markdown content, preserve valid Markdown formatting.

Do not claim that content has been saved unless the corresponding wott tool succeeds.

When a tool returns an error, report the failure clearly rather than claiming the operation succeeded.

## Tool Result Handling

After a successful tool call:

* Use the returned IDs and metadata for subsequent operations.
* Do not invent IDs, names, timestamps, or results.
* Summarize the result clearly to the user.
* Mention important created or modified resources.
* For destructive operations, clearly state what was deleted.

For multi-step operations, continue using IDs returned by previous tools.

Example:

```text
create_project
    ↓
project_id returned
    ↓
create_notebook(project_id)
```

Do not substitute a guessed project ID.

## Ambiguous Requests

Ask for clarification when the intended wott resource cannot be determined safely.

Examples:

> "Update my research project."

If multiple projects match, ask which project.

> "Delete my notes."

If multiple notebooks match, ask which notebook.

For read-only operations, it may be acceptable to present multiple matching resources and ask the user to choose.

For destructive operations, never choose between multiple matching resources automatically.

## When Not to Use wott

Do not trigger wott for requests that do not require wott data or actions.

Examples:

* "What is the capital of Japan?"
* "Write a poem about space."
* "Explain how neural networks work."
* "Calculate 25 × 18."
* "Search the web for today's AI news."

wott should be used when the request concerns the user's wott workspace, projects, notebooks, or stored project knowledge.

## Response Guidelines

After completing an wott operation:

* Be concise.
* State what was done.
* Include relevant resource names and IDs when useful.
* For search results, summarize the most relevant matches.
* For creation operations, provide the created resource details.
* For updates, state which fields were changed.
* For deletion, clearly state that the resource was permanently deleted.

Do not expose internal authentication details, access tokens, or implementation secrets.

## Specialized Skills

Detailed instructions are available in the specialized skills:

```text
skills/
├── SKILL.md
│
├── project-management/
│   ├── SKILL.md
│   ├── agents/
│   │   └── agent.yaml
│   ├── reference/
│   │   ├── project-tool-reference.md
│   │   └── project-workflows.md
│   ├── examples/
│   │   ├── create-project.md
│   │   ├── update-project.md
│   │   └── find-project.md
│   └── evaluations/
│       ├── README.md
│       └── *.json
│
├── notebook-management/
│   ├── SKILL.md
│   ├── agents/
│   │   └── agent.yaml
│   ├── reference/
│   │   ├── notebook-tool-reference.md
│   │   └── notebook-workflows.md
│   ├── examples/
│   │   ├── create-notebook.md
│   │   ├── update-notebook.md
│   │   └── find-notebook.md
│   └── evaluations/
│       ├── README.md
│       └── *.json
│
└── search-project/
    ├── SKILL.md
    ├── agents/
    │   └── agent.yaml
    ├── reference/
    │   ├── search-tool-reference.md
    │   └── search-workflows.md
    ├── examples/
    │   ├── search-notebook.md
    │   ├── search-specific-topic.md
    │   └── search-project-context.md
    └── evaluations/
        ├── README.md
        └── *.json
```

Use the specialized skill when detailed guidance is required for a particular capability. The top-level wott skill provides routing and shared behavior across all wott operations.

## Capability Summary

| Capability                  | Tool              | Operation |
| --------------------------- | ----------------- | --------- |
| List projects               | `get_projects`    | Read      |
| Create project              | `create_project`  | Create    |
| Update project              | `update_project`  | Update    |
| Read/list/search notebooks  | `get_notebook`    | Read      |
| Create notebook/folder      | `create_notebook` | Create    |
| Update notebook             | `update_notebook` | Update    |
| Delete notebook/folder      | `delete_notebook` | Delete    |
| Search notebooks in project | `search_project`  | Read      |

wott operates only on the authenticated user's authorized workspace and does not perform public web search or interact with external real-world systems through these tools.
