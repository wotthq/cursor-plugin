---
name: search-specific-topic-example
description: Example workflow for finding information about a specific topic within an wott project's notebooks.
metadata:
  short-description: Search a topic in wott
---

# Search for a Specific Topic

## User request

> Find where I wrote about MCP OAuth and Cloud Run deployment in my AI project.

## Workflow

Identify the project:

```text
get_projects
```

Then search:

```text
search_project
```

with a query preserving the important technical terms:

```json
{
  "project_id": "PROJECT_ID",
  "query": "MCP OAuth Cloud Run deployment"
}
```

If relevant results are found, present them in ranked order.

If the user asks to read the most relevant result, use:

```text
get_notebook
```

with the returned:

```text
markdown_item_id
```

## Expected behavior

The model should not invent notebook content from the search result.

It should retrieve the notebook before quoting, summarizing, or analyzing its full content.
