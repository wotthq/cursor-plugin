---
name: project-tool-reference
description: Reference for wott project management tools, including their inputs, outputs, permissions, and expected behavior. Use when detailed project tool information is needed.
metadata:
  short-description: wott project tool reference
---

# Project Tool Reference

This document describes the wott MCP tools used by the `project-management` skill.

The schemas and examples below describe the current project-management MCP contract.

The MCP layer is responsible for the response envelope and should return predictable, explicitly defined fields.

---

# 1. Common Response Structure

All successful project-management tools should use the following common response structure:

```json
{
  "success": true,
  "...operation-specific fields...": "...",
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

`success` indicates that the operation completed successfully.

`timestamp` is the time at which the MCP response was generated.

The `timestamp` belongs to the MCP response layer. It should not be added to the domain/service functions.

Example:

```json
{
  "success": true,
  "projects": [],
  "count": 0,
  "timestamp": "2026-09-03T10:00:00.000Z"
}
```

---

# 2. Tool Annotations

Every project-management MCP tool should declare all three behavior annotations.

| Tool             | readOnlyHint | openWorldHint | destructiveHint |
| ---------------- | -----------: | ------------: | --------------: |
| `get_projects`   |       `true` |       `false` |         `false` |
| `get_project`    |       `true` |       `false` |         `false` |
| `search_project` |       `true` |       `false` |         `false` |
| `create_project` |      `false` |       `false` |         `false` |
| `update_project` |      `false` |       `false` |         `false` |

These annotations describe the intended behavior of the tools.

`openWorldHint` is `false` because these tools operate on the authenticated user's wott data and do not modify or interact with public external systems.

---

# 3. MCP Project Object

When a project is returned as a normalized project object, wott exposes only the fields required by the MCP client.

The MCP project object is:

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "description": "Unique identifier of the project."
    },
    "visibility": {
      "type": "string",
      "enum": [
        "private",
        "unlisted",
        "public"
      ],
      "description": "Visibility of the project."
    },
    "project_slug": {
      "type": "string",
      "description": "URL-safe slug of the project."
    },
    "public_slug": {
      "type": [
        "string",
        "null"
      ],
      "description": "Public slug of the project when applicable."
    },
    "status": {
      "type": "string",
      "enum": [
        "active",
        "suspended",
        "archived"
      ],
      "description": "Current project status."
    },
    "enabled": {
      "type": "boolean",
      "description": "Whether the project is enabled."
    },
    "name": {
      "type": "string",
      "description": "Name of the project."
    },
    "description": {
      "type": "string",
      "description": "Project description."
    },
    "notebook_count": {
      "type": "integer",
      "minimum": 0,
      "description": "Number of notebooks in the project."
    },
    "file_count": {
      "type": "integer",
      "minimum": 0,
      "description": "Number of files in the project."
    },
    "storage_bytes": {
      "type": "integer",
      "minimum": 0,
      "description": "Storage used by the project in bytes."
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "Time when the project was created."
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "Time when the project was last updated."
    }
  },
  "required": [
    "project_id",
    "visibility",
    "project_slug",
    "public_slug",
    "status",
    "enabled",
    "name",
    "notebook_count",
    "file_count",
    "storage_bytes",
    "created_at",
    "updated_at"
  ],
  "additionalProperties": false
}
```

The normalized MCP project object is based on the following TypeScript type:

```ts
type McpProject = {
  project_id: string;
  visibility: "private" | "unlisted" | "public";
  project_slug: string;
  public_slug: string | null;
  status: "active" | "suspended" | "archived";
  enabled: boolean;
  name: string;
  description?: string;
  notebook_count: number;
  file_count: number;
  storage_bytes: number;
  created_at: string;
  updated_at: string;
};
```

Internal project fields should not be exposed unless a specific MCP tool requires them.

Examples of internal fields that should generally remain private include:

```text
owner_id
share_id
membership_ids
member_count
api_key_count
service_account_count
metadata
version
deleted_at
```

---

# 4. `get_projects`

## Purpose

Returns projects accessible to the authenticated wott user.

## Annotation

```json
{
  "readOnlyHint": true,
  "openWorldHint": false,
  "destructiveHint": false
}
```

## Input Schema

The tool accepts no arguments.

```json
{
  "type": "object",
  "additionalProperties": false
}
```

## Output Schema

```json
{
  "type": "object",
  "properties": {
    "success": {
      "type": "boolean",
      "const": true,
      "description": "Whether the project retrieval succeeded."
    },
    "projects": {
      "type": "array",
      "description": "Projects accessible to the authenticated user.",
      "items": {
        "$ref": "#/$defs/project"
      }
    },
    "count": {
      "type": "integer",
      "minimum": 0,
      "description": "Number of projects returned."
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "Time when the MCP response was generated."
    }
  },
  "required": [
    "success",
    "projects",
    "count",
    "timestamp"
  ],
  "additionalProperties": false,
  "$defs": {
    "project": {
      "type": "object",
      "properties": {
        "project_id": {
          "type": "string"
        },
        "visibility": {
          "type": "string",
          "enum": [
            "private",
            "unlisted",
            "public"
          ]
        },
        "project_slug": {
          "type": "string"
        },
        "public_slug": {
          "type": [
            "string",
            "null"
          ]
        },
        "status": {
          "type": "string",
          "enum": [
            "active",
            "suspended",
            "archived"
          ]
        },
        "enabled": {
          "type": "boolean"
        },
        "name": {
          "type": "string"
        },
        "description": {
          "type": "string"
        },
        "notebook_count": {
          "type": "integer",
          "minimum": 0
        },
        "file_count": {
          "type": "integer",
          "minimum": 0
        },
        "storage_bytes": {
          "type": "integer",
          "minimum": 0
        },
        "created_at": {
          "type": "string",
          "format": "date-time"
        },
        "updated_at": {
          "type": "string",
          "format": "date-time"
        }
      },
      "required": [
        "project_id",
        "visibility",
        "project_slug",
        "public_slug",
        "status",
        "enabled",
        "name",
        "notebook_count",
        "file_count",
        "storage_bytes",
        "created_at",
        "updated_at"
      ],
      "additionalProperties": false
    }
  }
}
```

## Example

```json
{
  "success": true,
  "projects": [
    {
      "project_id": "project_123",
      "visibility": "private",
      "project_slug": "physical_ai_research",
      "public_slug": null,
      "status": "active",
      "enabled": true,
      "name": "Physical AI Research",
      "description": "Research and development of physical AI systems.",
      "notebook_count": 12,
      "file_count": 24,
      "storage_bytes": 4589231,
      "created_at": "2026-08-20T10:30:00Z",
      "updated_at": "2026-09-01T15:20:00Z"
    }
  ],
  "count": 1,
  "timestamp": "2026-09-03T10:00:00Z"
}
```

---

# 5. `get_project`

## Purpose

Returns detailed information about one project.

## Annotation

```json
{
  "readOnlyHint": true,
  "openWorldHint": false,
  "destructiveHint": false
}
```

## Input Schema

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "minLength": 1,
      "description": "Unique identifier of the project."
    }
  },
  "required": [
    "project_id"
  ],
  "additionalProperties": false
}
```

## Output

The output should use the same common response envelope:

```json
{
  "success": true,
  "project": {},
  "timestamp": "2026-09-03T10:00:00Z"
}
```

The `project` object should use the normalized MCP project object defined in Section 3.

## Example

```json
{
  "success": true,
  "project": {
    "project_id": "project_123",
    "visibility": "private",
    "project_slug": "physical_ai_research",
    "public_slug": null,
    "status": "active",
    "enabled": true,
    "name": "Physical AI Research",
    "description": "Research and development of physical AI systems.",
    "notebook_count": 12,
    "file_count": 24,
    "storage_bytes": 4589231,
    "created_at": "2026-08-20T10:30:00Z",
    "updated_at": "2026-09-01T15:20:00Z"
  },
  "timestamp": "2026-09-03T10:00:00Z"
}
```

---

# 6. `search_project`

## Purpose

Searches the authenticated user's projects using a user-provided search query.

## Annotation

```json
{
  "readOnlyHint": true,
  "openWorldHint": false,
  "destructiveHint": false
}
```

## Input Schema

The exact schema must match the implementation of the `search_project` tool.

If the current implementation accepts `query` and `limit`, use:

```json
{
  "type": "object",
  "properties": {
    "query": {
      "type": "string",
      "minLength": 1,
      "description": "Text used to find relevant projects."
    },
    "limit": {
      "type": "integer",
      "minimum": 1,
      "maximum": 50,
      "description": "Maximum number of projects to return."
    }
  },
  "required": [
    "query"
  ],
  "additionalProperties": false
}
```

If the actual implementation does not accept `limit`, remove `limit` from this schema.

## Output

The output should follow the same common response envelope.

If the implementation returns a project collection, the output should use:

```json
{
  "success": true,
  "projects": [],
  "count": 0,
  "timestamp": "2026-09-03T10:00:00Z"
}
```

Do not document `query` or `total` as output fields unless the actual `search_project` implementation returns them.

## Example

```json
{
  "success": true,
  "projects": [
    {
      "project_id": "project_123",
      "visibility": "private",
      "project_slug": "robotics_research",
      "public_slug": null,
      "status": "active",
      "enabled": true,
      "name": "Robotics Research",
      "description": "Research on robotics and physical AI.",
      "notebook_count": 8,
      "file_count": 15,
      "storage_bytes": 2398123,
      "created_at": "2026-08-20T10:30:00Z",
      "updated_at": "2026-09-01T15:20:00Z"
    }
  ],
  "count": 1,
  "timestamp": "2026-09-03T10:00:00Z"
}
```

---

# 7. `create_project`

## Purpose

Creates a new project for the authenticated wott user.

## Annotation

```json
{
  "readOnlyHint": false,
  "openWorldHint": false,
  "destructiveHint": false
}
```

## Input Schema

The current implementation accepts:

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1,
      "description": "Name of the new project."
    },
    "description": {
      "type": "string",
      "description": "Optional description of the project."
    }
  },
  "required": [
    "name"
  ],
  "additionalProperties": false
}
```

## Output Schema

The current `createProject()` domain function intentionally returns only the information required to identify the newly created project.

It returns:

```json
{
  "project_id": "project_456",
  "project_slug": "humanoid_robotics",
  "project_name": "Humanoid Robotics"
}
```

Therefore, the MCP response should be:

```json
{
  "type": "object",
  "properties": {
    "success": {
      "type": "boolean",
      "const": true
    },
    "project": {
      "type": "object",
      "properties": {
        "project_id": {
          "type": "string"
        },
        "project_slug": {
          "type": "string"
        },
        "project_name": {
          "type": "string"
        }
      },
      "required": [
        "project_id",
        "project_slug",
        "project_name"
      ],
      "additionalProperties": false
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    }
  },
  "required": [
    "success",
    "project",
    "timestamp"
  ],
  "additionalProperties": false
}
```

## Example

```json
{
  "success": true,
  "project": {
    "project_id": "project_456",
    "project_slug": "humanoid_robotics",
    "project_name": "Humanoid Robotics"
  },
  "timestamp": "2026-09-03T10:00:00Z"
}
```

The create operation does not need to return the entire project document.

If the model needs complete project information after creation, it can use `get_project`.

---

# 8. `update_project`

## Purpose

Updates basic information for a project that the authenticated user owns.

## Annotation

```json
{
  "readOnlyHint": false,
  "openWorldHint": false,
  "destructiveHint": false
}
```

## Input Schema

The current tool accepts:

```json
{
  "type": "object",
  "properties": {
    "project_id": {
      "type": "string",
      "minLength": 1,
      "description": "Unique identifier of the project to update."
    },
    "project_name": {
      "type": "string",
      "minLength": 1,
      "description": "New project name."
    },
    "description": {
      "type": "string",
      "description": "New project description."
    },
    "location": {
      "type": [
        "string",
        "null"
      ],
      "description": "Project location."
    },
    "region": {
      "type": [
        "string",
        "null"
      ],
      "description": "Project region."
    },
    "country_code": {
      "type": [
        "string",
        "null"
      ],
      "description": "Project country code."
    },
    "region_code": {
      "type": [
        "string",
        "null"
      ],
      "description": "Project region code."
    }
  },
  "required": [
    "project_id"
  ],
  "additionalProperties": false
}
```

At least one mutable project field should be supplied in addition to `project_id`.

The implementation should reject an update request that contains only `project_id`.

The mutable fields are:

```text
project_name
description
location
region
country_code
region_code
```

The authenticated user's ID is not accepted as a tool argument.

The MCP server obtains the authenticated user from the OAuth authentication context.

## Output

The current `updateProject()` domain operation returns only the project ID:

```json
{
  "project_id": "project_456"
}
```

Therefore, the MCP response should be:

```json
{
  "type": "object",
  "properties": {
    "success": {
      "type": "boolean",
      "const": true,
      "description": "Whether the project was successfully updated."
    },
    "project": {
      "type": "object",
      "properties": {
        "project_id": {
          "type": "string",
          "description": "ID of the updated project."
        }
      },
      "required": [
        "project_id"
      ],
      "additionalProperties": false
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "Time when the MCP response was generated."
    }
  },
  "required": [
    "success",
    "project",
    "timestamp"
  ],
  "additionalProperties": false
}
```

## Example

```json
{
  "success": true,
  "project": {
    "project_id": "project_456"
  },
  "timestamp": "2026-09-03T11:15:00Z"
}
```

If the model needs to inspect the updated project, it can call `get_project`.

---

# 9. Output Design Principles

wott project tools should return small, predictable, operation-specific responses.

Every successful response should contain:

```json
{
  "success": true,
  "...operation-specific fields...": "...",
  "timestamp": "..."
}
```

Do not return raw Firestore documents directly from MCP tools.

Do not expose internal database or authentication fields unless they are explicitly required by the tool.

For collection operations:

```json
{
  "success": true,
  "projects": [],
  "count": 0,
  "timestamp": "..."
}
```

For creation:

```json
{
  "success": true,
  "project": {
    "project_id": "...",
    "project_slug": "...",
    "project_name": "..."
  },
  "timestamp": "..."
}
```

For updates:

```json
{
  "success": true,
  "project": {
    "project_id": "..."
  },
  "timestamp": "..."
}
```

For a single normalized project:

```json
{
  "success": true,
  "project": {
    "project_id": "...",
    "visibility": "private",
    "project_slug": "...",
    "public_slug": null,
    "status": "active",
    "enabled": true,
    "name": "...",
    "notebook_count": 0,
    "file_count": 0,
    "storage_bytes": 0,
    "created_at": "...",
    "updated_at": "..."
  },
  "timestamp": "..."
}
```

The response should contain only fields required by the operation.

Do not return a complete Firestore `ProjectDocument` merely because it is available internally.

---

# 10. MCP Structured Output

When an `outputSchema` is declared for an MCP tool, the server must return structured content that conforms to that schema.

Conceptually:

```ts
return {
  content: [
    {
      type: "text",
      text: JSON.stringify(result),
    },
  ],
  structuredContent: result,
};
```

The `structuredContent` object must conform to the declared output schema.

The serialized text content should represent the same result.

The MCP server should not declare an output schema that differs from the actual tool response.

The implementation and this reference document must remain synchronized.

---

# 11. Domain and MCP Separation

Project domain functions should not contain MCP-specific response fields.

For example, the domain create operation may return:

```ts
return {
  project_id,
  project_slug,
  project_name,
};
```

The MCP tool then wraps it:

```ts
return mcpResponse({
  success: true,
  project,
  timestamp: new Date().toISOString(),
});
```

Similarly, the domain update operation may return:

```ts
return {
  project_id,
};
```

The MCP tool wraps it with:

```ts
{
  success: true,
  project,
  timestamp
}
```

This keeps the domain layer independent from MCP while keeping the MCP interface predictable.

---

# 12. Security and Access

The authenticated user ID must never be accepted as an MCP tool argument.

The MCP server determines the authenticated user from the OAuth authentication context.

Project access must be checked server-side.

For project updates, the current implementation requires the authenticated user to have the `owner` role.

The model must not be trusted to provide or override:

```text
user_id
owner_id
membership_id
role
permissions
```

Access control must remain enforced by the server.
