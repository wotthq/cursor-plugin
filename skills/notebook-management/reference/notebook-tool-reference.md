---
name: notebook-tool-reference
description: Reference for wott notebook management tools, including their inputs, outputs, permissions, content handling, and expected behavior. Use when detailed notebook tool information is needed.
metadata:
  short-description: wott notebook tool reference
---

# Notebook Tool Reference

This document describes the wott MCP tools used by the `notebook-management` skill.

These tools allow the authenticated user to create, read, search, update, move, and delete notebook files and folders inside wott projects.

The MCP tools are scoped to the authenticated user. The MCP layer receives the authenticated `user_id` and passes it to `notebookService`, which performs project access checks before executing notebook operations.

---

# 1. Notebook MCP tools

The current notebook MCP server exposes four tools:

| Tool              | Purpose                                      | `readOnlyHint` | `openWorldHint` | `destructiveHint` |
| ----------------- | -------------------------------------------- | -------------: | --------------: | ----------------: |
| `get_notebook`    | Get, search, or list notebook items          |         `true` |         `false` |           `false` |
| `create_notebook` | Create a notebook file or folder             |        `false` |         `false` |           `false` |
| `update_notebook` | Update a notebook file or folder             |        `false` |         `false` |           `false` |
| `delete_notebook` | Permanently delete a notebook file or folder |        `false` |         `false` |            `true` |

There is currently no separate `list_notebooks` tool.

`get_notebook` handles:

* retrieving a specific notebook item
* retrieving an item by path
* searching notebook items
* listing direct folder children
* listing notebook items in a project

---

# 2. Common response structure

Successful notebook MCP operations use a common response envelope:

```json
{
  "success": true,
  "...": "operation-specific fields",
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

The `success` and `timestamp` fields are added by the MCP tool layer.

The underlying notebook service and domain functions should return only operation-specific data.

For example:

```ts
const result = await notebookService.create({
  user_id: userId,
  project_id,
  type,
  name,
  parent_id,
  content,
  frontmatter,
});

return mcpResponse({
  success: true,
  ...result,
  timestamp: new Date().toISOString(),
});
```

---

# 3. Notebook item representation

The MCP API does not expose the complete internal `NotebookItemDoc`.

Notebook files are represented using the fields needed by the MCP client.

A file may look like:

```json
{
  "item_id": "notebook_123",
  "project_id": "project_456",
  "type": "file",
  "name": "robotics.md",
  "title": "Robotics Research",
  "parent_id": null,
  "path": "robotics.md",
  "storage_path": "projects/project_456/notebooks/robotics.md",
  "mime_type": "text/markdown",
  "size_bytes": 2450,
  "version": 2
}
```

When content is returned, the file may additionally contain:

```json
{
  "content": "# Robotics Research\n\nResearch notes.",
  "frontmatter": {
    "id": "notebook_123",
    "type": "note",
    "title": "Robotics Research",
    "created": "2026-09-03T10:00:00.000Z",
    "updated": "2026-09-03T10:00:00.000Z"
  }
}
```

A folder is represented as:

```json
{
  "item_id": "folder_123",
  "project_id": "project_456",
  "type": "folder",
  "name": "Research",
  "parent_id": null,
  "path": "Research"
}
```

Do not expose internal Firestore fields unless they are intentionally part of the MCP contract.

Internal fields include, but are not limited to:

* `created_by_id`
* `owner_id`
* `disabled_member_ids`
* internal block fields
* internal graph fields
* authentication information
* OAuth information
* Login credentials
* server secrets

---

# 4. `get_notebook`

## Purpose

Gets notebook files and folders from an wott project.

`get_notebook` supports four modes:

1. `item`
2. `search`
3. `children`
4. `list`

The mode is determined by the selector supplied in the request.

---

## Annotation

```json
{
  "readOnlyHint": true,
  "openWorldHint": false,
  "destructiveHint": false
}
```

---

## Input

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of the project containing the notebook items."
    },
    "item_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of a specific notebook item."
    },
    "path": {
      "type": "string",
      "minLength": 1,
      "description": "Path of a notebook item."
    },
    "search": {
      "type": "string",
      "minLength": 1,
      "description": "Search text for finding notebook items."
    },
    "parent_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of a folder whose direct children should be returned."
    },
    "type": {
      "type": "string",
      "enum": [
        "file",
        "folder"
      ],
      "description": "Optional filter for notebook item type."
    }
  },
  "additionalProperties": false
}
```

Only one of these selectors may be supplied:

* `item_id`
* `path`
* `search`
* `parent_id`

The `type` field can be used as an optional filter.

---

## 4.1 Get a specific item

Request:

```json
{
  "project_id": "project_123",
  "item_id": "notebook_456"
}
```

For a folder, the result contains an `item`.

For a file, the result contains the file metadata, Markdown content, and backlinks.

### Folder response

```json
{
  "success": true,
  "mode": "item",
  "item": {
    "item_id": "folder_456",
    "project_id": "project_123",
    "type": "folder",
    "name": "Research",
    "parent_id": null,
    "path": "Research"
  },
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

### File response

```json
{
  "success": true,
  "mode": "item",
  "file": {
    "item_id": "notebook_456",
    "project_id": "project_123",
    "type": "file",
    "name": "robotics.md",
    "title": "Robotics Research",
    "parent_id": null,
    "path": "robotics.md",
    "storage_path": "projects/project_123/notebooks/robotics.md",
    "mime_type": "text/markdown",
    "size_bytes": 2450,
    "version": 2,
    "created_at": "2026-09-03T10:00:00.000Z",
    "updated_at": "2026-09-03T10:30:00.000Z",
    "content": "# Robotics Research\n\nResearch notes.",
    "backlinks": []
  },
  "backlinks": [],
  "timestamp": "2026-09-03T10:30:00.000Z"
}
```

---

## 4.2 Get an item by path

Request:

```json
{
  "project_id": "project_123",
  "path": "research/robotics.md"
}
```

The path is resolved inside the specified project.

For a file, Markdown content and backlinks are returned.

For a folder, folder information is returned.

The response uses:

```json
{
  "success": true,
  "mode": "item"
}
```

---

## 4.3 Search notebook items

Request:

```json
{
  "project_id": "project_123",
  "search": "robotics"
}
```

Search matches notebook:

* `name`
* `path`
* `title`

Example response:

```json
{
  "success": true,
  "mode": "search",
  "query": "robotics",
  "items": [
    {
      "item_id": "notebook_123",
      "project_id": "project_123",
      "type": "file",
      "name": "robotics.md",
      "title": "Robotics Research",
      "parent_id": null,
      "path": "robotics.md"
    }
  ],
  "count": 1,
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

An optional type filter can be supplied:

```json
{
  "project_id": "project_123",
  "search": "robotics",
  "type": "file"
}
```

---

## 4.4 Get folder children

Request:

```json
{
  "project_id": "project_123",
  "parent_id": "folder_456"
}
```

Returns the direct children of the specified folder.

Example:

```json
{
  "success": true,
  "mode": "children",
  "parent_id": "folder_456",
  "items": [
    {
      "item_id": "folder_789",
      "project_id": "project_123",
      "type": "folder",
      "name": "Papers",
      "parent_id": "folder_456",
      "path": "Research/Papers"
    },
    {
      "item_id": "notebook_123",
      "project_id": "project_123",
      "type": "file",
      "name": "notes.md",
      "title": "Research Notes",
      "parent_id": "folder_456",
      "path": "Research/notes.md"
    }
  ],
  "count": 2,
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

---

## 4.5 List project notebook items

Request:

```json
{
  "project_id": "project_123"
}
```

Returns notebook items in the project.

Example:

```json
{
  "success": true,
  "mode": "list",
  "items": [
    {
      "item_id": "folder_123",
      "project_id": "project_123",
      "type": "folder",
      "name": "Research",
      "parent_id": null,
      "path": "Research"
    },
    {
      "item_id": "notebook_456",
      "project_id": "project_123",
      "type": "file",
      "name": "notes.md",
      "title": "Notes",
      "parent_id": null,
      "path": "notes.md"
    }
  ],
  "count": 2,
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

An optional `type` filter can be supplied:

```json
{
  "project_id": "project_123",
  "type": "folder"
}
```

---

# 5. `create_notebook`

## Purpose

Creates a notebook file or folder inside an wott project.

The authenticated user must have access to the project.

---

## Annotation

```json
{
  "readOnlyHint": false,
  "openWorldHint": false,
  "destructiveHint": false
}
```

---

## Input

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of the project where the notebook item will be created."
    },
    "type": {
      "type": "string",
      "enum": [
        "file",
        "folder"
      ],
      "description": "Whether to create a Markdown file or folder."
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "description": "Name of the notebook file or folder."
    },
    "parent_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "ID of the parent folder. Omit or use null for the project root."
    },
    "content": {
      "type": "string",
      "description": "Markdown content for a notebook file."
    },
    "frontmatter": {
      "type": "object",
      "description": "Optional frontmatter metadata for a notebook file."
    }
  },
  "required": [
    "project_id",
    "type",
    "name"
  ],
  "additionalProperties": false
}
```

---

## File creation

Example:

```json
{
  "project_id": "project_123",
  "type": "file",
  "name": "Robotics Research",
  "content": "# Robotics Research\n\nInitial notes."
}
```

The file name is normalized to Markdown:

```text
Robotics Research.md
```

The content is converted into canonical Markdown with frontmatter.

---

## Folder creation

Example:

```json
{
  "project_id": "project_123",
  "type": "folder",
  "name": "Research"
}
```

Folders do not use Markdown content or frontmatter.

---

## File response

```json
{
  "success": true,
  "file": {
    "item_id": "notebook_123",
    "project_id": "project_123",
    "type": "file",
    "name": "robotics.md",
    "title": "Robotics Research",
    "parent_id": null,
    "path": "robotics.md",
    "storage_path": "projects/project_123/notebooks/robotics.md",
    "mime_type": "text/markdown",
    "size_bytes": 2450,
    "content": "# Robotics Research\n\nInitial notes.",
    "frontmatter": {
      "id": "notebook_123",
      "type": "note",
      "title": "Robotics Research",
      "created": "2026-09-03T10:00:00.000Z",
      "updated": "2026-09-03T10:00:00.000Z"
    }
  },
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

---

## Folder response

```json
{
  "success": true,
  "item": {
    "item_id": "folder_123",
    "project_id": "project_123",
    "type": "folder",
    "name": "Research",
    "parent_id": null,
    "path": "Research"
  },
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

---

# 6. `update_notebook`

## Purpose

Updates an existing notebook file or folder.

Supported operations include:

* updating Markdown content
* renaming a file or folder
* moving an item within the project
* updating frontmatter
* updating structured notebook metadata

The actual item type is determined from Firestore.

The MCP client does not provide a `type` field.

---

## Annotation

```json
{
  "readOnlyHint": false,
  "openWorldHint": false,
  "destructiveHint": false
}
```

---

## Input

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of the project containing the notebook."
    },
    "item_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of the notebook file or folder to update."
    },
    "content": {
      "type": "string",
      "description": "New Markdown content for a notebook file."
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "description": "New name for the notebook file or folder."
    },
    "parent_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "New parent folder ID. Use null to move the item to the project root."
    },
    "frontmatter": {
      "type": "object",
      "description": "Frontmatter fields to update."
    },
    "metadata": {
      "type": "object",
      "properties": {
        "keywords": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "hashtags": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "mentions": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "entities": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "type": {
                "type": "string",
                "minLength": 1
              },
              "value": {
                "type": "string",
                "minLength": 1
              }
            },
            "required": [
              "type",
              "value"
            ],
            "additionalProperties": false
          }
        }
      },
      "additionalProperties": false
    }
  },
  "required": [
    "project_id",
    "item_id"
  ],
  "additionalProperties": false
}
```

At least one update field must be supplied:

* `content`
* `name`
* `parent_id`
* `frontmatter`
* `metadata`

---

## Content update

When `content` is supplied, the notebook Markdown content is replaced.

Example:

```json
{
  "project_id": "project_123",
  "item_id": "notebook_456",
  "content": "# Robotics\n\nUpdated research notes."
}
```

Content updates can trigger:

* metadata extraction
* relationship compilation
* wiki-link resolution
* notebook index updates
* notebook graph compilation

---

## Rename

Example:

```json
{
  "project_id": "project_123",
  "item_id": "notebook_456",
  "name": "Robotics Research"
}
```

Markdown files are normalized to use the `.md` extension.

---

## Move

Example:

```json
{
  "project_id": "project_123",
  "item_id": "notebook_456",
  "parent_id": "folder_789"
}
```

To move the item to the project root:

```json
{
  "project_id": "project_123",
  "item_id": "notebook_456",
  "parent_id": null
}
```

When a folder is renamed or moved, descendant paths are updated.

---

## Frontmatter update

Example:

```json
{
  "project_id": "project_123",
  "item_id": "notebook_456",
  "frontmatter": {
    "title": "Humanoid Robotics",
    "status": "active"
  }
}
```

Frontmatter is stored inside the Markdown file.

---

## Metadata update

Example:

```json
{
  "project_id": "project_123",
  "item_id": "notebook_456",
  "metadata": {
    "keywords": [
      "robotics",
      "humanoid"
    ],
    "hashtags": [
      "ai",
      "robotics"
    ],
    "mentions": [
      "openai"
    ],
    "entities": [
      {
        "type": "technology",
        "value": "humanoid robotics"
      }
    ]
  }
}
```

Explicitly supplied metadata fields replace the current value of those fields.

When content is updated, automatically extracted metadata can add values to the existing metadata.

---

## Output

For a file update:

```json
{
  "success": true,
  "file": {
    "item_id": "notebook_456",
    "project_id": "project_123",
    "type": "file",
    "name": "robotics.md",
    "title": "Robotics Research",
    "parent_id": null,
    "path": "robotics.md",
    "storage_path": "projects/project_123/notebooks/robotics.md",
    "mime_type": "text/markdown",
    "size_bytes": 2450,
    "version": 3,
    "created_at": "2026-09-03T10:00:00.000Z",
    "updated_at": "2026-09-03T10:30:00.000Z",
    "content": "# Robotics Research\n\nUpdated notes.",
    "frontmatter": {
      "title": "Robotics Research"
    }
  },
  "timestamp": "2026-09-03T10:30:00.000Z"
}
```

`content` is returned when content was updated.

`frontmatter` is returned when frontmatter was updated.

---

# 7. `delete_notebook`

## Purpose

Permanently deletes a notebook file or folder.

If the target is a folder, all descendant notebook items are deleted.

Deletion may remove:

* notebook files
* notebook folders
* Markdown files from Cloud Storage
* Firestore notebook items
* notebook indexes
* notebook blocks
* notebook relationships
* notebook graph nodes
* notebook graph edges
* orphaned graph metadata nodes

---

## Annotation

```json
{
  "readOnlyHint": false,
  "openWorldHint": false,
  "destructiveHint": true
}
```

Because this operation is permanent and destructive, the MCP client should treat it as a destructive action.

---

## Input

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of the project containing the notebook item."
    },
    "item_id": {
      "type": "string",
      "minLength": 1,
      "description": "ID of the notebook file or folder to permanently delete."
    }
  },
  "required": [
    "project_id",
    "item_id"
  ],
  "additionalProperties": false
}
```

---

## Output

```json
{
  "success": true,
  "deleted_item_id": "folder_123",
  "deleted_item_ids": [
    "folder_123",
    "file_456",
    "file_789"
  ],
  "deleted_file_ids": [
    "file_456",
    "file_789"
  ],
  "deleted_count": 3,
  "deleted_file_count": 2,
  "graph": {
    "deleted_edge_count": 8,
    "deleted_node_count": 2,
    "deleted_orphan_node_count": 3,
    "deleted_orphan_node_ids": [
      "keyword_ai",
      "entity_robotics",
      "hashtag_research"
    ]
  },
  "timestamp": "2026-09-03T10:30:00.000Z"
}
```

`deleted_item_ids` contains the requested item and all descendants deleted with it.

`deleted_file_ids` contains only notebook files.

---

# 8. Authorization

Notebook operations are always performed in the context of the authenticated MCP user.

The MCP tool receives the authenticated user ID from the MCP authentication layer.

The tool must never accept `user_id` from the model.

The service layer performs project access validation:

```text
Authenticated OAuth user
        ↓
MCP tool
        ↓
user_id
        ↓
notebookService
        ↓
requireProjectAccess()
        ↓
Notebook domain function
```

The current service access requirements are:

| Operation | Project access check |
| --------- | -------------------- |
| Create    | Required             |
| Get       | Required             |
| Update    | Required             |
| Delete    | Required             |

Future permission-specific checks can be added at the service layer, for example:

```ts
permissions: ["notebooks:read"]
```

```ts
permissions: ["notebooks:create"]
```

```ts
permissions: ["notebooks:update"]
```

```ts
permissions: ["notebooks:delete"]
```

---

# 9. Notebook content and storage

Notebook Markdown content is stored in Cloud Storage.

Firestore stores notebook metadata and related application state.

For a notebook file, the Storage path follows:

```text
projects/{project_id}/notebooks/{path}
```

For example:

```text
projects/project_123/notebooks/research/robotics.md
```

`get_notebook` loads file content from Cloud Storage when retrieving a specific file.

`create_notebook` writes the canonical Markdown content to Cloud Storage.

`update_notebook` updates or moves the corresponding Storage file when necessary.

`delete_notebook` removes the corresponding Storage files.

---

# 10. Notebook metadata

Notebook files can contain Markdown frontmatter.

The canonical frontmatter generated during creation includes fields such as:

```yaml
---
id: notebook_123
type: note
title: Robotics Research
created: 2026-09-03T10:00:00.000Z
updated: 2026-09-03T10:00:00.000Z
---
```

Structured notebook metadata is maintained separately in Firestore.

The current structured metadata fields are:

```json
{
  "keywords": [],
  "hashtags": [],
  "mentions": [],
  "entities": []
}
```

Entities use:

```json
{
  "type": "technology",
  "value": "humanoid robotics"
}
```

---

# 11. Relationships and graph processing

Notebook updates can update wott's notebook knowledge structures.

When notebook content changes, the system can:

* extract wiki links
* resolve wiki links
* compile notebook relationships
* extract notebook metadata
* update notebook metadata indexes
* compile notebook graph nodes and edges

When notebooks are deleted, associated relationships, indexes, graph edges, graph nodes, and orphaned metadata nodes are cleaned up where applicable.

These internal graph and indexing mechanisms should not be exposed to the model unless the MCP response explicitly requires them.

The delete response may include graph deletion counts because those counts are useful confirmation of the destructive operation.

---

# 12. Error handling

Notebook MCP tools use the common `mcpError()` helper for failures.

Typical errors include:

* `Project not found`
* `Notebook item not found`
* `Parent folder not found`
* `Parent item must be a folder`
* duplicate notebook or folder names
* missing notebook Storage files
* invalid notebook item types

The exact MCP error response is controlled by the server's `mcpError()` implementation.

This document should not duplicate the error implementation unless that contract changes.

---

# 13. MCP output schemas

If an `outputSchema` is added to any notebook tool, the schema must exactly match the structured result returned by that tool.

For example, a successful file creation currently has the conceptual structure:

```json
{
  "success": true,
  "file": {
    "item_id": "notebook_123",
    "project_id": "project_456",
    "type": "file",
    "name": "notes.md",
    "title": "Notes",
    "parent_id": null,
    "path": "notes.md",
    "storage_path": "projects/project_456/notebooks/notes.md",
    "mime_type": "text/markdown",
    "size_bytes": 500,
    "content": "# Notes\n",
    "frontmatter": {}
  },
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

If an output schema is introduced, it must describe this actual structure rather than a generic `notebook` object.

The following must remain synchronized:

```text
MCP tool input schema
        +
MCP output schema
        +
Actual TypeScript result
        +
Notebook skill reference
```

---

# 14. Data exposure rules

Only expose notebook information necessary to fulfill the user's request.

Never expose:

* Login credentials
* OAuth access tokens
* OAuth authorization codes
* internal authentication context
* server secrets
* service account credentials
* unrelated users' notebook data
* internal Firestore implementation details
* internal graph implementation details unless intentionally included in an operation result

Notebook content can be returned when it is necessary for the user's request and the authenticated user has access to the project.

The MCP server must always use the authenticated user's identity when determining access to notebook data.
