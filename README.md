# WOTT Plugin

WOTT connects ChatGPT to the user's wott workspace.

Wott allows the authenticated user to search, read, create, update, and manage projects and notebook content through the wott MCP server.

This skill is the top-level routing skill for the wott plugin. Use the specialized skills and their reference documentation when a request requires detailed project, notebook, or search behavior.

---

# Default Prompt

Use wott to search, read, create, and update my projects and notebooks.

---

# Capabilities

wott currently provides three specialized capabilities:

## Project Management

Use the `project-management` skill for project operations.

Supported operations:

* List accessible projects
* Find projects
* Read project information
* Create projects
* Update project information

Primary MCP tools:

```text
get_projects
create_project
update_project
```

For detailed project behavior, read:

```text
skills/project-management/SKILL.md
skills/project-management/reference/
```

Use the project-management skill when the request is about project-level information or project changes.

---

## Notebook Management

Use the `notebook-management` skill for notebook filesystem and content operations.

Supported operations:

* Find notebook files and folders
* Read notebook files and folders
* Search notebook items
* List notebook items
* Create notebook files
* Create notebook folders
* Update notebook content
* Update notebook names
* Move notebook files and folders
* Update notebook frontmatter
* Update notebook metadata
* Delete notebook files
* Delete notebook folders and their descendants

Primary MCP tools:

```text
get_notebook
create_notebook
update_notebook
delete_notebook
```

For detailed notebook behavior, read:

```text
skills/notebook-management/SKILL.md
skills/notebook-management/reference/
```

Use the notebook-management skill when the request is about creating, reading, modifying, moving, or deleting notebook files or folders.

---

## Project Search

Use the `search-project` skill when the user wants to find information inside project notebook knowledge.

Supported operations:

* Search project notebook knowledge
* Find relevant notebooks
* Find technical information
* Find previous notes or research
* Find project context
* Search before reading or modifying a notebook

Primary MCP tool:

```text
search_project
```

The current search implementation uses hybrid retrieval:

```text
BM25
TF-IDF
Index Search
Weighted RRF
```

The current search scope is notebook knowledge.

File-source search is not currently supported by the MCP search contract.

For detailed search behavior, read:

```text
skills/search-project/SKILL.md
skills/search-project/reference/
```

---

# Skill Routing

Route requests to the most specific skill.

```text
User request
     │
     ├── Project discovery or project changes
     │        ↓
     │   project-management
     │
     ├── Search for information across project knowledge
     │        ↓
     │   search-project
     │
     └── Read/create/update/move/delete notebook items
              ↓
         notebook-management
```

When a request spans multiple capabilities, use the skills together.

For example:

```text
"Find my AI Research project and search it for humanoid robotics."
```

Workflow:

```text
project-management
        ↓
identify project
        ↓
search-project
        ↓
return relevant notebooks
```

Another example:

```text
"Find my deployment notes and update the notebook with this new information."
```

Workflow:

```text
search-project
        ↓
identify relevant notebook
        ↓
notebook-management
        ↓
read/update notebook
```

Another example:

```text
"Create a project called Robotics Research and create a notebook called Research Notes inside it."
```

Workflow:

```text
project-management
        ↓
create project
        ↓
notebook-management
        ↓
create notebook
```

Maintain the IDs returned by previous tool calls when passing context between skills.

---

# Tool Selection

Use the current MCP tools exactly as defined by the specialized skills.

Do not invent tool names or parameters.

## Project tools

```text
get_projects
create_project
update_project
```

## Notebook tools

```text
get_notebook
create_notebook
update_notebook
delete_notebook
```

## Search tool

```text
search_project
```

---

# Authentication and User Identity

wott MCP operations are performed in the context of the authenticated wott user.

The model must never ask the user for or provide:

```text
user_id
```

User identity is obtained from the MCP authentication layer.

The authentication flow is responsible for:

* OAuth authentication
* access token verification
* authenticated user identification
* authorization context

The service layer is responsible for validating access to project resources.

Do not attempt to bypass these controls.

---

# Authorization

Project and notebook operations are scoped to projects accessible to the authenticated user.

The MCP server performs authorization before executing protected operations.

The model must not:

* assume access to an arbitrary project
* guess project IDs
* guess notebook IDs
* access another user's data
* bypass project access checks
* request credentials from the user

When the target project or notebook is ambiguous, resolve the ambiguity before performing a mutation.

---

# Read and Mutation Rules

Read-only operations include:

```text
get_projects
get_notebook
search_project
```

Mutation operations include:

```text
create_project
update_project

create_notebook
update_notebook
delete_notebook
```

Search and read operations must not modify wott data.

Do not create, update, or delete anything unless the user's request requires that operation.

---

# Destructive Operations

Notebook deletion is supported.

The following operation is destructive:

```text
delete_notebook
```

Deleting a notebook folder can permanently delete:

* the folder
* descendant folders
* descendant notebook files
* associated Markdown files
* related notebook indexes
* notebook blocks
* notebook relationships
* associated graph data

Do not perform destructive operations against an ambiguous target.

The current wott MCP does not expose a `delete_project` tool.

Therefore:

```text
delete_project
```

is currently unsupported.

---

# Project Update Scope

The current `update_project` MCP operation supports only project information exposed by its input schema.

Supported fields include:

```text
project_name
description
location
region
country_code
region_code
```

Do not attempt to modify administrative project settings through `update_project`.

This includes, unless a future MCP tool explicitly supports them:

* project status
* enabled state
* visibility
* membership
* member permissions
* billing
* API keys
* service accounts
* integrations
* assistants
* workflows
* other administrative settings

---

# Notebook Content

wott notebooks are Markdown files or folders.

Notebook file content is stored in Cloud Storage, while Firestore stores notebook metadata and related application state.

Notebook operations can maintain:

* Markdown content
* frontmatter
* structured metadata
* wiki links
* relationships
* notebook indexes
* notebook graph data

Do not expose internal storage, Firestore, graph, or authentication implementation details unless they are intentionally part of the operation result.

---

# Search

`search_project` searches notebook knowledge within a specific project.

A search requires:

```text
project_id
query
```

and optionally:

```text
limit
```

The model should:

1. Identify the project.
2. Obtain the actual `project_id`.
3. Search the project.
4. Present relevant results.
5. Use `get_notebook` when full notebook content is required.

Do not treat search scores as confidence percentages.

Do not claim that information does not exist merely because a search returns no results.

---

# Ambiguity Handling

Do not guess when a project or notebook cannot be uniquely identified.

For example, if the user says:

```text
"Update my robotics notebook."
```

and multiple notebooks match, search first.

If multiple plausible notebooks remain:

```text
Ask the user to clarify.
```

Do not call:

```text
update_notebook
```

until the target is sufficiently identified.

The same rule applies to destructive operations.

---

# Tool Result Handling

Treat MCP responses as the source of truth.

Do not claim that an operation succeeded unless the MCP tool confirms success.

Do not invent:

* project IDs
* notebook IDs
* project slugs
* notebook paths
* search results
* notebook content
* update results
* deletion counts

When a tool returns an ID, preserve and reuse that ID for subsequent operations.

---

# Security

Never expose or request:

* OAuth access tokens
* OAuth authorization codes
* Login credentials
* Login service account credentials
* private keys
* MCP OAuth secrets
* server environment variables
* internal authentication context
* unrelated users' data

The wott MCP server handles authentication and authorization.

The Skills layer should focus on interpreting the user's request and selecting the appropriate capability.

---

# Specialized Skill Documentation

For detailed behavior, always prefer the specialized skill documentation over this top-level file.

Project management:

```text
skills/project-management/SKILL.md
skills/project-management/reference/
skills/project-management/examples/
```

Notebook management:

```text
skills/notebook-management/SKILL.md
skills/notebook-management/reference/
skills/notebook-management/examples/
```

Project search:

```text
skills/search-project/SKILL.md
skills/search-project/reference/
skills/search-project/examples/
```

The specialized skills contain the detailed workflows, tool schemas, examples, and evaluation scenarios.

---

# Scope

wott currently provides:

```text
Projects
├── List
├── Read
├── Create
└── Update

Notebook Files and Folders
├── Find
├── Search
├── Read
├── Create
├── Update
├── Move
└── Delete

Project Knowledge
└── Search notebook knowledge
```

The current MCP does not provide:

```text
Delete project
Cross-project search
Web search
Internet search
File-source project search
```

Do not claim support for capabilities that are not exposed by the current MCP server.
